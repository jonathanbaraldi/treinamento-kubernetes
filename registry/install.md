Instalação do Registro.

VMWare Harbor


URL = registry.rancher.signallink.us

# ssh -i chave.pem linux@201.166.145.58

# sudo su
# yum update -y
# yum install yum-utils -y
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum install -y --setopt=obsoletes=0  docker-ce-17.03.0.ce-1.el7.centos
# systemctl enable docker && systemctl start docker
# usermod -aG docker linux
# docker -v


	# sudo su
	# yum install docker
	# systemctl start docker
	# systemctl enable docker.service

# yum install epel-release
# yum install -y python-pip

# pip install docker-compose
# yum upgrade python* -y

# pip install docker-compose==1.9.0

# yum install wget -y

# wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.0.tgz
# tar xvf harbor-offline-installer-v1.6.0.tgz
# cd harbor
Editar o harbor.cfg, adicionar em HOSTNAME = registry.rancher.tesla-ads.com
	
	registry.rancher.signallink.us

# vi harbor.cfg
# ./install.sh
Aguardar até baixar as imagens. 

admin/Harbor12345

Logar no console e criar o projeto para o treinamento.
Criar os usuários. 

