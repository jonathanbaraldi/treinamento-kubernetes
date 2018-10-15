# Exercício 1

### Instalar Docker em todos os hosts

Iremos instalar o Docker ce 17.03 em todos os 4 hosts.

```sh
$ sudo su
$ yum update -y
$ yum install yum-utils -y
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install -y --setopt=obsoletes=0  docker-ce-17.03.0.ce-1.el7.centos
$ systemctl enable docker && systemctl start docker
# $ usermod -aG docker linux - Adicionar somente se precisar
$ docker -v
```

### Logar no registro

Depois de instalar o Docker em todos os hosts, será preciso fazer o login do docker engine do host no registro.
Para isso, temos que editar o arquivo /etc/docker/daemon.json, e inserir o parâmetro de registro inseguro. 
```sh
$ vi /etc/docker/daemon.json
{
	"insecure-registries" : ["registry.rancher.signallink.us"]
}
```
Após inserir o registro inseguro, devido ao certificado SSL, é preciso reiniciar o docker e fazer o login, usando seu usuário do treinamento.
```sh
$ systemctl restart docker
$ docker login registry.rancher.signallink.us
	Login sucessful
```
Para testar se está tudo certo, somente no host A, iremos criar uma imagem para enviar para nosso repositorio no registro. Iremos baixar a imagem do node:alpine, colocar a tag correta para então fazer o push para o nosso repositório privado. 

Onde estiver escrito {user}, é preciso substituir pelo seu nome de usuario no registro.

```sh
$ docker pull node:alpine
$ docker tag node:alpine registry.rancher.signallink.us/{user}/node:alpine
$ docker push registry.rancher.signallink.us/{user}/node:alpine
```
### Limpar o host
Algumas vezes é preciso sanitizar o host, e deixar ele apto para receber carga novamente. Para isso, muitas vezes é preciso remover o docker, suas dependências, arquivos do kubernetes, entre outros.

Não itemos limpar o host nesse passo, mas os comandos estão aqui como referência.
```sh
$ docker rm -f $(docker ps -qa)
$ docker rmi -f $(docker images -q)
$ docker volume rm $(docker volume ls)
$ systemctl stop docker
$ for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
$ rm -rf /etc/ceph /etc/cni /etc/kubernetes /opt/cni /opt/rke /run/secrets/kubernetes.io /run/calico /run/flannel /var/lib/calico /var/lib/etcd /var/lib/cni /var/lib/kubelet /var/lib/rancher/rke/log /var/log/containers /var/log/pods /var/run/calico
$ systemctl start docker
```

### FirewallD
Talvez o firewall esteja ativado nos host's, será preciso desativar.
```sh
$ iptables -L -n -v
$ systemctl stop firewalld
$ systemctl disable firewalld
$ iptables -L -n -v
$ systemctl stop firewalld
$ systemctl disable firewalld
```




# Exercício 2

### Fazer build das imagens, rodar docker-compose.

Nesse exercício iremos construir as imagens dos containers que iremos usar, colocar elas para rodar em conjunto com o docker-compose. 

Entrar no host A, e instalar os pacotes abaixo, que incluem Git, Python, Pip e o Docker-compose.
```sh
$ sudo su
$ yum install git -y
$ yum install epel-release -y
$ yum install -y python-pip
$ pip install docker-compose
$ yum upgrade python* -y
$ pip uninstall docker-compose
$ pip install docker-compose==1.9.0
```
Com os pacotes instalados, agora iremos baixar o código fonte e começaremos a fazer os build's e rodar os containers.
```sh
$ cd /root
$ git clone https://github.com/jonathanbaraldi/treinamento-kubernetes
$ cd treinamento-kubernetes/dia-1-2/app
```
#### Container=REDIS
Iremos fazer o build da imagem do Redis para a nossa aplicação.
```sh
$ cd redis
$ docker build -t registry.rancher.signallink.us/{user}/redis .
$ docker run -d --name redis -p 6379:6379 registry.rancher.signallink.us/{user}/redis
$ docker ps
$ docker logs redis
```
Com isso temos o container do Redis rodando na porta 6379.

#### Container=NODE
Iremos fazer o build do container do NodeJs, que contém a nossa aplicação.
```sh
$ cd ../node
$ docker build -t registry.rancher.signallink.us/{user}/node .
```
Agora iremos rodar a imagem do node, fazendo a ligação dela com o container do Redis.
```sh
$ docker run -d --name node -p 8080:8080 --link redis registry.rancher.signallink.us/{user}/node
$ docker ps 
$ docker logs node
```
Com isso temos nossa aplicação rodando, e conectada no Redis. A api para verificação pode ser acessada em /redis.

#### Container=NGINX
Iremos fazer o build do container do nginx, que será nosso balanceador de carga.
```sh
$ cd ../nginx
$ docker build -t registry.rancher.signallink.us/{user}/nginx .
```
Criando o container do nginx a partir da imagem e fazendo a ligação com o container do Node
```sh
$ docker run -d --name nginx -p 80:80 --link node registry.rancher.signallink.us/{user}/nginx
$ docker ps
```
Podemos acessar então nossa aplicação nas portas 80 e 8080 no ip da nossa instância.

Iremos acessar a api em /redis para nos certificar que está tudo ok, e depois iremos limpar todos os containers e volumes.
```sh
$ docker rm -f $(docker ps -a -q)
$ docker volume rm $(docker volume ls)
```


#### DOCKER-COMPOSE
Nesse exercício que fizemos agora, colocamos os containers para rodar, e interligando eles, foi possível observar  como funciona nossa aplicação que tem um contador de acessos.
Para rodar nosso docker-compose, precisamos remover todos os containers que estão rodando e ir na raiz do diretório para rodar.

É preciso editar o arquivo docker-compose.yml, onde estão os nomes das imagens e colocar o seu nome de usuário.

Linha 8 = registry.rancher.signallink.us/{user}/nginx
Linha 18 = image: registry.rancher.signallink.us/{user}/redis
Linha 47 = image: registry.rancher.signallink.us/{user}/node

Após alterar e colocar o nome correto das imagens, rodar o comando de up -d para subir a stack toda.

```sh
$ cd ..
$ vi docker-compose.yml
$ docker-compose -f docker-compose.yml up -d
$ curl <ip>:80 
	----------------------------------
	This page has been viewed 29 times
	----------------------------------
```
Se acessarmos o IP:80, iremos acessar a nossa aplicação. Olhar os logs pelo docker logs, e fazer o carregamento do banco em /load

Para terminar nossa aplicação temos que rodar o comando do docker-compose abaixo:
```sh
$ docker-compose down
```





# Exercício 3

### Instalar Rancher - Single Node

Nesse exercício iremos instalar o Rancher 2.0 versão single node. Isso significa que o Rancher e todos seus componentes estão em um container. 

Entrar no host A, que será usado para hospedar o Rancher Server. Iremos verficar se não tem nenhum container rodando ou parado, e depois iremos instalar o Rancher.
```sh
$ docker ps -a
$ docker run -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:stable
```
Com o Rancher já rodando, irei adicionar a entrada de cada DNS para o IP de cada máquina.

```sh
$ rancher.<user>.signallink.us = IP do host A
```


# Exercício 4

### Criar cluster Kubernetes

Nesse exercício iremos criar um cluster Kubernetes. Após criar o cluster, iremos instalar o kubectl no host A, e iremos usar para interagir com o cluster.

Seguir as instruções na aula para fazer o deployment do cluster.
Após fazer a configuração, o Rancher irá exibir um comando de docker run, para adicionar os host's.

Adicionar o host B e host C. 

Não adicionar o host D, pois ele será usado para restore do kubernetes.

Pegar o seu comando no seu rancher.
```sh
$ docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.8 --server https://rancher.tesla-ads.com --token f5vgmgvg55h2fghsxqcwt966bb4skmmz4nl4sgx2f559vjmtrw6m25 --ca-checksum eb4748a50fa4a5b1b2bc92620406a1a6d584246a1dd84c710e2437e62caa6e6e --etcd --controlplane --worker
```
Será um cluster com 2 nós.
Navegar pelo Rancher e ver os painéis e funcionalidades.



# Exercício 5

### Instalar kubectl no host A

Agora iremos instalar o kubectl, que é a CLI do kubernetes. Através do kubectl é que iremos interagir com o cluster.
```sh
$ sudo su
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
$ yum install -y kubectl
$ kubectl
```
Com o kubectl instalado, pegar as credenciais de acesso no Rancher e configurar o kubectl.
```sh
$ vi ~/.kube/config
$ kubectl get nodes
```








docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.8 --server https://189.84.130.21 --token sz6cmqr9n94qsl26vz8ztcs6nstdmschpwxp6pplg6vhfzkqg99jk7 --ca-checksum 63f3971e273fb1ffe92c79213584e73c9d54cd035ff881e40332e4c38861ca35 --etcd --controlplane --worker
