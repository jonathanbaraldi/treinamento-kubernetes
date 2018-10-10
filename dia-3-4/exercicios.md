

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 6 - Traefik - DNS
	- Deploy do Traefik e explicação

# kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml 
# kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-ds.yaml
# kubectl --namespace=kube-system get pods
# kubectl apply -f ui.yml

	Teste do Cheese
# kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/cheese-deployments.yaml
# kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/cheese-services.yaml
# kubectl apply -f cheese.yml


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 7 - Graylog - LOG
		
# kubectl apply -f graylog.yml
	Trocar o DNS em 2 pontos do template. Aguardar explicação.
	Fazer a configuração do Graylog
	Explicação sobre o USO do Graylog


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercicio 8 - Grafana - Monitoramento
	- Deploy do Grafana e prometheus e explicação
	- Registro

# Lançar prometheus do Catálogo e configurar os DNS.
	Explicação sobre o uso do Grafana.	


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 9 - App exemplo
	- Fazer deploy de aplicação exemplo no cluster, mostrando o uso do registro.

# docker pull cizixs/whoami
# docker tag cizixs/whoami registry.rancher.tesla-ads.com/jonathan/whoami
# docker push registry.rancher.tesla-ads.com/jonathan/whoami
# kubectl apply -f app.yml


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 10 - Cron Job
	Fazer deploy de cronjob
	Vamos rodar o cronjob.yml

# kubectl apply -f cronjob.yml
	cronjob "hello" created

Depois de criada a cron, pegamos o estado dela usando:

# kubectl -n <nome-aluno> get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         <none>


Ainda não existe um job ativo, e nenhum agendado também.
Vamos esperar por 1 minutos ate o job ser criado:

# kubectl -n <nome-aluno> get jobs --watch

Entrar no Kubernetes para ver os logs e a sequencia de execucao.



>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 11 - ConfigMap
	Fazer deploy de ConfigMap
	Define Pod environment variables with data from multiple ConfigMaps

# kubectl apply -f configmap.yml

Analisar o configmap criado no kubernetes e ver o log do POD no kuberentes, e os dados que estao no POD.
	
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 12 - Secrets
	Fazer deploy de Secrets
	Cada item precisa ser codificado com base64:

# echo -n "<nome-aluno>" | base64
YWRtaW4tam9u
# echo -n "<senha>" | base64
MTIzNDU2Nzg5MA==

Agora vamos escrever o secret com esses objetos:

# kubectl apply -f secrets.yml

Entrar dentro do volume, do container e verificar o secret criado com o seu valor.
















>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 11 - Liveness
	- Deployment com Healthcheck

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

# kubectl apply -f liveness.yml 
	Depois de 10 segundos, verificamos que o container reiniciou. 
# kubectl -n <nome-aluno> describe pod liveness-http 
# kubectl -n <nome-aluno> get pod liveness-http 


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
>> Exercício 13 - Rolling-update

# kubectl apply -f rolling-update.yml

	Nesse arquivo o nginx está na versão 1.7.9
	Para atualizar a imagem do container para 1.9.1 iremos usar o kubectl rolling-update e especificar a nova imagem.

# kubectl  -n <nome-aluno> rolling-update my-nginx --image=nginx:1.9.1

	Created my-nginx-ccba8fbd8cc8160970f63f9a2696fc46

Em outra janela, você pode ver que o kubectl adicionou o label do deployment para os pods, que o valor é um hash da configuração, para distinguir os pods novos dos velhos

# kubectl -n <nome-aluno> get pods -l app=nginx -L deployment






Traefik
https://docs.traefik.io/user-guide/kubernetes/


