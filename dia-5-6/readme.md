  

# Exercício 15

### DR Rancher - Backup

Iremos executar um plano de DR do Rancher, onde iremos fazer o backup do banco do rancher, e depois iremos fazer o restore do Rancher na mesma máquina onde ele está.

Para isso, iremos parar o container do Rancher server, criar o volume com a cópia do banco, e gerar o arquivo com o backup. Feito isso, iremos reiniciar o Rancher para continuar de onde havíamos parado.  


```sh
$ docker stop rancher
$ docker create --volumes-from rancher --name rancher-data-14-10-2018 rancher/rancher:stable
$ docker run  --volumes-from rancher-data-14-10-2018 -v $PWD:/backup alpine tar zcvf /backup/rancher-data-backup-rancher2-14-10-2018.tar.gz /var/lib/rancher  
$ docker start rancher
```

Foi gerado um arquivo ''rancher-data-backup-rancher2-14-10-2018.tar.gz'', que contém o backup do Rancher Server.

### DR Rancher - Restore

Para que possamos fazer o restore do Rancher, iremos parar nosso Rancher rodando, e iremos remover o container dele, bem como os volumes todos, limpando qualquer registro que ele possa ter deixado.

Para limpar, iremos usar:
```sh
$ docker rm -f $(docker ps -a -q)
$ docker volume rm $(docker volume ls)
```
Uma vez limpo o host, iremos agora colocar para rodar um Rancher inteiramente novo:
```sh
$ docker ps -a
$ docker run -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:stable
```

Com esse novo Rancher Server rodando, iremos acessar sua url para certificar que é uma nova instalação. 

O que irá nos dizer logo na tela inicial, é se o Rancher pedir para criar a senha de administrador, se aparecer essa tela, é uma instalação nova.

Após se certifcar que realmente é uma nova instalação, iremos parar o container que está executando, e fazer o restore do banco que está no arquivo da etapa anterior.

Verificar através do comando de docker logs, que o Rancher foi completamente instalado.

```sh
$ docker stop rancher
$ docker run  --volumes-from rancher -v $PWD:/backup  alpine sh -c "rm /var/lib/rancher/* -rf  &&  tar zxvf /backup/rancher-data-backup-rancher2-14-10-2018.tar.gz"
$ docker start rancher
```

Iremos agora acessar o Rancher novamente e iremos verificar que o usuário admin já está configurado, e que nosos ambiente voltou ao seu estado anterior.




# Exercício 16

### DR Kubernetes - Backup

Para termos o backup do kubernetes sendo executado de tempos em tempos, iremos configurar isto no arquivo de configuração do Kubernetes, no painel do Rancher.

Para colocar o kubernetes para fazer backup. Editar o arquivo YML com a configuração do cluster, no painel do Rancher.

Localizar o item do ETCD, setar o snapshot para true, definir o tempo de criação em 15 minutos, e o período de retencão, que será de 24 horas.

```sh
etcd:
    snapshot: true # enables recurring etcd snapshots
    creation: "15m" # time increment between snapshots
    retention: "24h" # time increment before snapshot purge
```

Após fazer as alterações, é preciso salvar e verificar se as mesmas foram salvas, abrindo novamente o arquivo de configuração.

Com as alterações feitas, iremos localizar o diretório onde está sendo feito o backup. Ele estará na host onde foi alocado o etcd-1 do kubernetes.

Localizar o diretório do backup.

Se ele não estiver criado, é preciso criar.
```sh
$ mkdir -p /opt/rke/
$ mkdir -p /opt/rke/etcd-snapshots
$ cd /opt/rke/etcd-snapshots
```

### DR Kubernetes - Restore

Para nos certificarmos que realmente é um restore, iremos remover o cluster que temos no Rancher, e ficar sem cluster algum. 

Agora, iremos transferir para a máquina de destino o backup que iremos restaurar. 

Logar na máquina de destino, criar o diretório e dar permissão:
```sh
$ mkdir -p /opt/rke/
$ mkdir -p /opt/rke/etcd-snapshots/
$ chmod -R 777 /opt/rke/etcd-snapshots/
```

Copiar o backup para a outra máquina, host D

```sh
$ scp -i rancher.pem /opt/rke/etcd-snapshots/2018-10-02T19:12:51Z_etcd root@186.200.35.150:/opt/rke/etcd-snapshots/2018-10-02T19:12:51Z_etcd

$ scp -i rancher.pem /opt/rke/etcd-snapshots/pki.bundle.tar.gz root@186.200.35.150:/opt/rke/etcd-snapshots/pki.bundle.tar.gz
```

Criar o arquivo cluster.yml

Nesse arquivo, iremos colocar o IP da máquina destino do restore e o usuário do ssh que o RKE irá usar. Será o nome do aluno, e iremos criar esse usuário mais adiante. Não é possível usar root para RedHat e CentOS, por isso iremos criar um usuário com permissão de rodar o docker.

```sh
nodes:
    - address: 201.166.145.161
      user: jonathan
      role:
        - controlplane
        - etcd
        - worker
```

Criação do usuário

Iremos logar na máquina de destino do restore, e criar o usuário que será usado pelo RKE para fazer o restore do Kubernetes.


```sh
$ useradd -g docker rke
$ cp -av /root/.ssh /home/rke && chown rke. -R /home/rke/.ssh
```




```sh
$ adduser jonathan
$ su jonathan
$ cd ~/
$ ssh-keygen
```

Foram criadas as chaves públicas e privadas no diretório  **/home/jonathan/.ssh/**
```sh
$ cp /home/jonathan/.ssh/id_rsa.pub /home/jonathan/.ssh/authorized_keys
$ exit
```
Virar sudo e adicionar o usuario ao grupo do docker.
```sh
$ usermod -aG docker jonathan
```

Editar o arquivo  **/etc/ssh/sshd_config**, e alterar os parâmetros:

- PasswordAuthentication no
```sh
$ vi /etc/ssh/sshd_config
$ service sshd restart
````

Iremos pegar a chave privada que foi criada para o usuário, e adicionar ela nas chaves da máquina de onde iremos rodar os comandos do restore pelo RKE.

Iremos copiar ela, pela tela mesmo.
```sh
$ cat /home/jonathan/.ssh/id_rsa
```

E adicionar a chave do usuário linux para **.ssh/id_rsa**, para que o usuário possa se logar e rodar o docker.
Logado na máquina onde iremos rodar os comandos, colar a chave.

```sh
$ vi /root/.ssh/id_rsa
$ chmod 600 /root/.ssh/id_rsa 
$ ssh jonathan@189.84.130.21
```
Para ter finalizado com sucesso, acessamos o terminal da máquina de destino.

Iremos então Instalar o RKE na máquina de onde iremos disparar os comandos para restauração do kubernetes.

```sh
$ yum install wget -y
$ wget https://github.com/rancher/rke/releases/download/v0.1.9/rke_linux-amd64
$ mv rke_linux-amd64 rke
$ chmod +x rke
```
O comando abaixo executa a funcionalidade de restore do RKE, onde passamos o nome do arquivo de backup que iremos usar.
```sh
$ ./rke etcd snapshot-restore --name 2018-10-02T19:12:51Z_etcd --config cluster.yml
```

Aogra iremos acompanhar todo o processo até ter terminado de restaurar o Snapshot.

Após ter terminado, ir até o painel do Rancher, e lançar um novo cluster, com as mesmas configurações do anterior. Colocar o nome de '''Restore'''.

Iremos rodar o comando de docker run para adicionar os host, e iremos rodar na máquina onde o kubernetes foi restaurado.
```sh
$ docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.0.8 --server https://rancher.instrutor.signallink.us --token vlssfsgk6qwmd4fgbgdtg2h65bpdg6mrp2tps98v9km82zch6kzvdr --ca-checksum 4d37519c0bedda5c379af1844614cc52744ca8c9eea912024430608dfe547188 --etcd --controlplane --worker
```
Esperar até o cluster estar 100% operacional e verificar se os deployments continuam e foram restaurados.














# Exercício 17

### RANCHER HA


Agora iremos fazer uma instalação do Rancher em HA. Para isso iremos usar a instância que está instalado o RKE do exercício anterior, e iremos colocar o Rancher em 2 máquinas as C e D.

Limpar todos os nodes de todos container rodando, para sanitizar o ambiente todo. 

```sh
$ docker rm -f $(docker ps -qa)
$ docker volume rm $(docker volume ls)
$ systemctl stop docker
$ for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
$ rm -rf /etc/ceph /etc/cni /etc/kubernetes /opt/cni /opt/rke /run/secrets/kubernetes.io /run/calico /run/flannel /var/lib/calico /var/lib/etcd /var/lib/cni /var/lib/kubelet /var/lib/rancher/rke/log /var/log/containers /var/log/pods /var/run/calico
$ systemctl start docker
```

Iremos criar o usuario na máquina de destino, o nosso host A. 

```sh
$ adduser rancher
$ su rancher
$ cd ~/
$ ssh-keygen
```

Foram criadas as chaves públicas e privadas no diretório  **/home/rancher/.ssh/**
```sh
$ cp /home/rancher/.ssh/id_rsa.pub /home/rancher/.ssh/authorized_keys
$ exit
```
Virar sudo e adicionar o usuario ao grupo do docker.
```sh
$ usermod -aG docker rancher
```

Editar o arquivo  **/etc/ssh/sshd_config**, e alterar os parâmetros:

- PasswordAuthentication no
```sh
$ vi /etc/ssh/sshd_config
$ service sshd restart
````

Iremos pegar a chave privada que foi criada para o usuário, e adicionar ela nas chaves da máquina de onde iremos rodar os comandos do restore pelo RKE.

Iremos copiar ela, pela tela mesmo.
```sh
$ cat /home/rancher/.ssh/id_rsa
```

E adicionar a chave do usuário linux para **.ssh/id_rsa**, para que o usuário possa se logar e rodar o docker.
Logado na máquina onde iremos rodar os comandos, colar a chave.

```sh
$ adduser rancher
$ su rancher
$ cd ~/

$ vi /home/rancher/.ssh/id_rsa
$ chmod 600 /home/rancher/.ssh/id_rsa 
$ ssh rancher@189.84.130.18
$ exit
```
Para ter finalizado com sucesso, acessamos o terminal da máquina de destino.


Criar o arquivo rancher-cluster.yml


```sh
$ cd /home/rke

$ vi rancher-cluster.yml

nodes:
  - address: 189.84.130.18
    # internal_address: 172.16.22.12
    user: rancher
    role: [controlplane,worker,etcd]
    ssh_key_path: /home/rancher/.ssh/id_rsa 
  #- address: 165.227.116.167
    #internal_address: 172.16.32.37
    #user: ubuntu
    #role: [controlplane,worker,etcd]
  #- address: 165.227.127.226
    #internal_address: 172.16.42.73
    #user: ubuntu
    #role: [controlplane,worker,etcd]
services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h
```

```sh
$ ./rke up --config ./rancher-cluster.yml
```

Iremos acompanhar o deployment de todo Rancher e do cluster Kubernetes do Rancher.

Ao término, o RKE deverá ter gerado o arquivo **kube_config_rancher-cluster.yml** , que contém as configurações do 

Se pegarmos este arquivo, e usarmos na instância onde instalamos o kubectl, podemos rodar os comandos no cluster.

Com o cluster rodando, iremos para o próximo passo que é instalar o Helm.

```sh
$ cd /home
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
$ tar -xvf helm-v2.11.0-linux-amd64.tar.gz
$ cd linux-amd64
$ kubectl -n kube-system create serviceaccount tiller
$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
$ ./helm init --service-account tiller
```

Instalado Rancher

```sh
$ ./helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

Iremos ver a versão do Rancher disponível para o Helm
```sh
$ ./helm search rancher
```


Instalar o gerenciador de certificados
```sh
$ ./helm install stable/cert-manager  --name cert-manager --namespace kube-system
```

```sh
$ ./helm install rancher-stable/rancher --name rancher --namespace cattle-system --set hostname=rancher.instrutor.signallink.us
```

Irá demorar um certo tempo até o ambiente estar 100% disponível, por isso é preciso aguardar um pouco.






Backup
https://rancher.com/docs/rancher/v2.x/en/backups/backups/single-node-backups/


