	

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

etcd:
    snapshot: true # enables recurring etcd snapshots
    creation: "15m" # time increment between snapshots
    retention: "24h" # time increment before snapshot purge

Após fazer as alterações, é preciso salvar e verificar se as mesmas foram salvas, abrindo novamente o arquivo de configuração.

Com as alterações feitas, iremos localizar o diretório onde está sendo feito o backup. Ele estará na host onde foi alocado o etcd-1 do kubernetes.

Localizar o diretório do backup.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	/opt/rke/etcd-snapshots.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Agora iremos transferir para a máquina de destino o backup que iremos restaurar. 

Logar na máquina de destino, criar o diretório e dar permissão:
# mkdir -p /opt/rke/
# mkdir -p /opt/rke/etcd-snapshots/
# chmod -R 777 /opt/rke/etcd-snapshots/


Copiar o backup para a outra máquina, host D

scp -i rancher.pem /opt/rke/etcd-snapshots/2018-10-02T19:12:51Z_etcd linux@186.200.35.150:/opt/rke/etcd-snapshots/2018-10-02T19:12:51Z_etcd

scp -i rancher.pem /opt/rke/etcd-snapshots/pki.bundle.tar.gz linux@186.200.35.150:/opt/rke/etcd-snapshots/pki.bundle.tar.gz


Criar o arquivo cluster.yml
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
nodes:
    - address: 201.166.145.161
      user: linux
      role:
        - controlplane
        - etcd
        - worker
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Adicionar a chave do usuário linux para .ssh/id_rsa, para que o usuário possa se logar e rodar o docker.

#vi .ssh/id_rsa
# chmod 600 .ssh/id_rsa 
# ssh linux@186.200.35.150

Atualizar o OPENSSH

# yum update -y
# yum install update
# yum update openssh

Instalar o RKE

# wget https://github.com/rancher/rke/releases/download/v0.1.9/rke_linux-amd64
# mv rke_linux-amd64 rke
# chmod +x rke

Rodar o comando de restaurar o ETCD

./rke etcd snapshot-restore --name 2018-10-02T19:12:51Z_etcd --config cluster.yml


Ir no Rancher no console e lançar novo cluster com ETCD restaurado








Backup
https://rancher.com/docs/rancher/v2.x/en/backups/backups/single-node-backups/