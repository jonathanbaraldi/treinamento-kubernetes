

* Corrigir a documentação
* Adicionar novos exercícios
* Fazer Rancher HA




url-map.md

Registro
	registry.rancher.tesla-ads.com

Por usuario
	rancher.<user>.signallink.us
	traefik.rancher.<user>.rancher.jonathan.signallink.us
	graylog.rancher.<user>.rancher.jonathan.signallink.us
	grafana.rancher.<user>.rancher.jonathan.signallink.us
	app.rancher<user>.rancher.jonathan.signallink.us

	<user>.treinamento.tesla-ads.com
	



- Criar os usuários no Registro, e colocar os projetos. Testar os push's.

MAPA DAS MÁQUINAS

A - ssh -i chave.pem linux@186.200.35.174 
B - ssh -i chave.pem linux@186.200.35.173 
C - ssh -i chave.pem linux@186.200.35.87 
D - ssh -i chave.pem linux@186.200.35.150 




>>>>>>>>>>>>>>>>>>>>>>>>>
http://dontpad.com/k8s-dc

Todos usuários 
traefik.<user>.treinamento.tesla-ads.com
graylog.<user>.treinamento.tesla-ads.com
grafana.<user>.treinamento.tesla-ads.com
app.<user>.treinamento.tesla-ads.com

Harbor - Registry
	
	registry.rancher.signallink.us (189.84.130.1)
		ssh -i brcloud-dc.pem root@189.84.130.1

Jonathan
	A = instrutor1.rancher.firecall.us (189.84.130.18)
	B = instrutor2.rancher.firecall.us (189.84.130.19)
	C = instrutor3.rancher.firecall.us (189.84.130.20)
	D = instrutor4.rancher.firecall.us (189.84.130.21)

		ssh -i brcloud-dc.pem root@189.84.130.18
		ssh -i brcloud-dc.pem root@189.84.130.19
		ssh -i brcloud-dc.pem root@189.84.130.20
		ssh -i brcloud-dc.pem root@189.84.130.21
		

	A =       rancher.jonathan.signallink.us
	B e C = *.rancher.jonathan.signallink.us 


Anselmo 
	A = anselmo1.rancher.firecall.us (189.84.130.2)
	B = anselmo2.rancher.firecall.us (189.84.130.3)
	C = anselmo3.rancher.firecall.us (189.84.130.4)
	D = anselmo4.rancher.firecall.us (189.84.130.5)

	A =       rancher.anselmo.tesla-ads.com
	B e C = *.rancher.anselmo.tesla-ads.com 

Claiton
	A = claiton1.rancher.firecall.us (189.84.130.6)
	B = claiton2.rancher.firecall.us (189.84.130.7)
	C = claiton3.rancher.firecall.us (189.84.130.8)
	D = claiton4.rancher.firecall.us (189.84.130.9)

	A =       rancher.claiton.tesla-ads.com
	B e C = *.rancher.claiton.tesla-ads.com 

Renato
	A = renato1.rancher.firecall.us (189.84.130.10)
	B = renato2.rancher.firecall.us (189.84.130.11)
	C = renato3.rancher.firecall.us(189.84.130.12)
	D = renato4.rancher.firecall.us (189.84.130.13)

	A =       rancher.renato.tesla-ads.com
	B e C = *.rancher.renato.tesla-ads.com 

Silvanei
	A = silvanei1.rancher.firecall.us (189.84.130.14)
	B = silvanei2.rancher.firecall.us (189.84.130.15)
	C = silvanei3.rancher.firecall.us (189.84.130.16)
	D = silvanei4.rancher.firecall.us (189.84.130.17)

	A =       rancher.silvanei.tesla-ads.com
	B e C = *.rancher.silvanei.tesla-ads.com 





