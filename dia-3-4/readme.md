# Exercício 6

### Traefik - DNS

O Traefik é a aplicação que iremos usar como ingress. Ele irá ficar escutando pelas entradas de DNS que o cluster deve responder. Ele possui um dashboard de  monitoramento e com um resumo de todas as entradas que estão no cluster.
```sh
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml 
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-ds.yaml
$ kubectl --namespace=kube-system get pods
```
Agora iremos configurar o DNS pelo qual o Traefik irá responder. No arquivo ui.yml, localizar a url, e fazer a alteração. Após a alteração feita, iremos rodar o comando abaixo para aplicar o deployment no cluster.
```sh
$ cd treinamento-kubernetes/dia-3-4/
$ kubectl apply -f ui.yml
```
#### Teste do Chesse
Iremos fazer o deployment das aplicações teste do Trafik. São aplicações com nomes de queijo. 
```sh
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/cheese-deployments.yaml
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/cheese-services.yaml
```
Precisamos configurar os ingress, para que as nossas aplicações respondam pelo DNS delas.
```sh
$ kubectl apply -f cheese.yml
```
Após aplicar o deployment, acessar o traefik, e acessar as url's das aplicações de queijo.



# Exercício 7

### Graylog - LOG

O Graylog é a aplicação que iremos usar como agregador de logs do cluster. Os logs dos containers podem ser vistos pelo Rancher, é um dos níveis de visualização. Pelo Graylog temos outros funcionalidades, e também é possível salvar para posterior pesquisa, e muitas outras funcionalidades.

Para instalar o Graylog, iremos aplicar o template dele, que está em graylog.yml. Para isso, é preciso que sejam editados 2 pontos no arquivo.

Linha 177 - value: http://graylog.{user}.rancher.tesla-ads.com/api
Linha 234 - host: graylog.{user}.rancher.tesla-ads.com

Substituir o {user}, pelo nome do aluno. Após substituir, aplicar e entrar no Graylog para configurar.
```sh
$ kubectl apply -f graylog.yml
```
Seguir os passos do instrutor para configuração do Graylog.




# Exercício 8

### Grafana - MONITORAMENTO

O Grafana/Prometheus é a stack que iremos usar para monitoramento. O Deployment dela será feito pelo Catálogo de Apps.
Iremos configurar seguindo as informações do instrutor, e fazer o deployment.

Será preciso altear os DNS das aplicações para que elas fiquem acessíveis.

Após o deploymnet, entrar no Grafana e Prometheus e mostrar seu funcionamento.




# Exercício 9

### Whoami - APP EXEMPLO

Neste exemplo de aplicação, iremos criar a nossa imagem, e enviar para o registro. Após isto, iremos configurar o arquivo app.yml, que contem todo deployment da nossa aplicação. 

Após o push para o registro, rodar os comandos abaixo para que a aplicação exemplo rode no cluster.

```sh
$ docker pull cizixs/whoami
$ docker tag cizixs/whoami registry.rancher.signallink.us/{user}/whoami
$ docker push registry.rancher.signallink.us/{user}/whoami
$ kubectl apply -f app.yml
```




# Exercício 10

### CronJob

O tipo de serviço como CronJob é um serviço igual a uma cron, porém é executado no cluster kubernetes. Você agenda um pod que irá rodar em uma frequência determinada de tempo. Pode ser usado para diversas funções, como executar backup's dos bancos de dados.

Nesse exemplo, iremos executar um pod, com um comando para retornar uma mensagem de tempos em tempos, a mensagem é "Hello from the Kubernetes cluster"

```sh
$ kubectl apply -f cronjob.yml
	cronjob "hello" created
```
Depois de criada a cron, pegamos o estado dela usando:
```sh
$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         <none>
```
Ainda não existe um job ativo, e nenhum agendado também.
Vamos esperar por 1 minutos ate o job ser criado:
```sh
$ kubectl  get jobs --watch
```
Entrar no Rancher para ver os logs e a sequencia de execucao.




# Exercício 11

### ConfigMap

O ConfigMap é um tipo de componente muito usado, principalmente quando precisamos colocar configurações dos nossos serviços externas aos contâiners que estão rodando a aplicação. 

Nesse exemplo, iremos criar um ConfigMap, e iremos acessar as informações dentro do container que está a aplicação.
```sh
$ kubectl apply -f configmap.yml
```
Agora iremos entrar dentro do container e verificar as configurações definidas no ConfigMap.






# Exercício 12

### Secrets

Os secrets são usados para salvar dados sensitivos dentro do cluster, como por exemplo senhas de bancos de dados. Os dados que ficam dentro do secrets não são visíveis a outros usuários, e também podem ser criptografados por padrão no banco.

Iremos criar os segredos.

```sh
$ echo -n "<nome-aluno>" | base64
YWRtaW4tam9u
$ echo -n "<senha>" | base64
MTIzNDU2Nzg5MA==
```

Agora vamos escrever o secret com esses objetos. Após colocar os valores no arquivo secrets.yml, aplicar ele no cluster.
```sh
$ kubectl apply -f secrets.yml
```
Agora com os secrets aplicados, iremos entrar dentro do container e ver como podemos recuperar seus valores.



# Exercício 13

### Liveness

Nesse exercício do liveness, iremos testar como fazer para dizer ao kubernetes, quando recuperar a nossa aplicação, caso alguma coisa aconteça a ela.
```js
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) { 
	duration := time.Now().Sub(started) 
	if duration.Seconds() > 10 { 
		w.WriteHeader(500) 
		w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds()))) 
	} else { 
		w.WriteHeader(200) 
		w.Write([]byte("ok")) 
	} 
}) 
```
O código acima, está dentro do container que iremos rodar. Nesse código, vocês podem perceber que tem um IF, que irá fazer que de vez em quando a aplicação responda dando erro. 

Como a aplicação irá retornar um erro, o serviço de liveness que iremos usar no Kubernetes, ficará verificando se a nossa aplicação está bem, e como ela irá falhar de tempos em tempos, o kubernetes irá reiniciar o nosso serviço.

```sh
$ kubectl apply -f liveness.yml 
	Depois de 10 segundos, verificamos que o container reiniciou. 
$ kubectl describe pod liveness-http 
$ kubectl get pod liveness-http 
```







# Exercício 14

### Rolling-update


Nesse exercício de rolling-update, iremos fazer o deployment do nginx na versão 1.7.9. Sendo 5 pods rodando a aplicação.

Iremos rodar o comando de rolling update, para atualizar para a versão 1.9.1. Dessa forma o Kubernetes irá rodar 1 container com a nova versão, e para um container com a antiga versão. Ele irá fazer isso para cada um dos containers, substituindo todos eles, e não havendo parada de serviço.

```sh
$ kubectl apply -f rolling-update.yml
```
Nesse arquivo o nginx está na versão 1.7.9
Para atualizar a imagem do container para 1.9.1 iremos usar o kubectl rolling-update e especificar a nova imagem.
```sh
$ kubectl  rolling-update my-nginx --image=nginx:1.9.1
	Created my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
```
Em outra janela, você pode ver que o kubectl adicionou o label do deployment para os pods, que o valor é um hash da configuração, para distinguir os pods novos dos velhos
```sh
$ kubectl get pods -l app=nginx -L deployment
```
