# Extras 2

### Declarar políticas de rede

https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/

https://github.com/ahmetb/kubernetes-network-policy-recipes

https://ahmet.im/blog/kubernetes-network-policy/

Para ver como funciona as policies de rede do kubernetes, iremos criar um deployment do nginx e expor ele via serviço.

```sh
$ kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
$ kubectl run nginx --image=nginx --replicas=2
	deployment.apps/nginx created
$ kubectl expose deployment nginx --port=80
	service/nginx exposed
```

Iremos ter 2 pods no namespace default rodando o nginx e expondo atraves do serviço chamado nginx.


#### Testando o serviço

Iremos agora testar o acesso do nosso pod no nginx para os outros pods. Para acessar o serviço iremos rodar um prompt de dentro de um container busybox e iremos acessar o nginx

```sh
$ kubectl run busybox --rm -ti --image=busybox /bin/sh
	Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

	Hit enter for command prompt

$ wget --spider --timeout=1 nginx
	Connecting to nginx (10.100.0.16:80)
$ exit
```

Vendo o IP do POD do nginx respondendo, conseguimos ver que existe a conexão.

Agora iremos aplicar as regras no arquivo **nginx-policy.yml**.

```sh
$ kubectl create -f nginx-policy.yaml
```

Após as regras aplicadas, iremos testar novamente a conexão do pod busybox com o nginx.

```sh
$ kubectl run busybox --rm -ti --image=busybox /bin/sh
	Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

	Hit enter for command prompt

$ wget --spider --timeout=1 nginx
	Connecting to nginx (10.100.0.16:80)
	wget: download timed out
$ exit
```

Podemos perceber que está dando timeout na conexão, o container do busybox não consegue conectar no nginx

Agora iremos criar novamente o container do busybox , mas iremos colocar o label de access=true para que o POD possa se conectar.

```sh
$ kubectl run busybox --rm -ti --labels="access=true" --image=busybox /bin/sh
	Waiting for pod default/busybox-472357175-y0m47 to be running, status is Pending, pod ready: false

	Hit enter for command prompt

$ wget --spider --timeout=1 nginx
	Connecting to nginx (10.100.0.16:80)
$ exit
```

### Kubeless

https://kubeless.io

Para instalar o Kubeless em nosso cluster, iremos rodar os comandos abaixo.

```sh
$ export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
$ kubectl create ns kubeless
$ kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml

$ kubectl get pods -n kubeless
NAME                                           READY     STATUS    RESTARTS   AGE
kubeless-controller-manager-567dcb6c48-ssx8x   1/1       Running   0          1h

$ kubectl get deployment -n kubeless
NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubeless-controller-manager   1         1         1            1           1h

$ kubectl get customresourcedefinition
NAME                          AGE
cronjobtriggers.kubeless.io   1h
functions.kubeless.io         1h
httptriggers.kubeless.io      1h
```

Depois de instalado no cluster, iremos agora instalar a linha de comando no mesmo local onde usamos nosso kubectl.

```sh
$ export OS=$(uname -s| tr '[:upper:]' '[:lower:]')
$ curl -OL https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless_$OS-amd64.zip && unzip kubeless_$OS-amd64.zip && sudo mv bundles/kubeless_$OS-amd64/kubeless /usr/local/bin/
```

Para verificar se foi instalado corretamente, iremos rodar:

```sh
$ kubeless function ls
```

### Kubeless - Função exemplo

Para fazer o deploy da função, iremos usar o arquivo de modelo exemplo **test.py** 

```sh
$ kubeless function deploy hello --runtime python2.7 --from-file test.py --handler test.hello
$ kubectl get functions
$ kubeless function ls
```

Para chamar a função podemos fazer da seguinte forma:
```sh
$ kubeless function call hello --data 'Hello world!'
```


Agora iremos aplicar a função através do arquivo YML, contendo todos os dados da função

```sh
$ kubectl apply -f function-nodejs.yml
```

### Kubeless UI 

https://github.com/kubeless/kubeless-ui

O Kubeless possui uma UI para facilitar a operação. Para instalar ela, iremos rodar:

```sh
$ kubectl create -f https://raw.githubusercontent.com/kubeless/kubeless-ui/master/k8s.yaml
```

Na interface do Rancher, iremos acessar pelo IP:PORTA no qual a UI foi instalada. E iremos executar alguns exemplos para entender seu funcionamento.


### Kubeless Simple API 

https://github.com/kubeless/functions/tree/master/incubator/rest-api

Iremos usar os arquivos **crypto.py**, e **requirements.txt** para pegar o valor do Bitcoin através de uma função.

```sh 
$ kubeless function deploy crypto --from-file crypto.py --handler crypto.handler --runtime python2.7 --dependencies requirements.txt
$ kubeless function call crypto --data '{"crypto": "bitcoin"}'
```

As trigers HTTP podem ser consultadas na documentação oficial.

https://github.com/kubeless/http-trigger











