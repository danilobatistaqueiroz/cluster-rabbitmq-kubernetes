# -- informações essenciais --

maneiras de subir um cluster de RabbitMQ:  
usando etcd, consul, aws, kubernetes  
no kubernetes pode-se usar: aplicar yamls, helmchart, operator  


é necessário usar statefulset para que não ocorra peer discovery race condition,  
no qual os nodes não sabem quem será o master,  
e mais de um vai assumir que é master e terá mais de um cluster.  

Important reasons for using a Stateful Set: sticky identity,  
simple network identifiers,  
stable persistent storage and the ability to perform ordered rolling upgrades.  

nodes que terminam de inicializar emitem um evento para o kubernetes.  

um node tenta se sincronizar com outros durante um tempo (default 5 minutos),  
se esse tempo estourar, ele desiste e não se considera inicializado.  

Health Checks podem acabar achando que um sistema saudável e operacional, não esteja bem, e reiniciar ou  
destruir e recriar sem uma razão, reduzindo a disponibilidade do sistema.  
Não há uma solução definitiva, há diversas opções para health checks, mas todas sujeitas a falhas.  

As ferramentas RabbitMQ CLI fornecem uma série de verificações de saúde predefinidas  

A reinicialização de um nó RabbitMQ não necessariamente resolverá um problema.  
Por exemplo, reiniciar um nó que está em estado de alarme porque está com pouco espaço em disco disponível não ajudará.  

um Liveness Probe dos mais simples:  
rabbitmq-diagnostics -q ping  

outros Liveness Probe:  
rabbitmq-diagnostics -q check_port_connectivity  
rabbitmq-diagnostics -q check_local_alarms  

Note, however, that they will fail for the nodes paused by the “pause minority” partition handliner strategy.  
Esses Liveness Probes vão falhar numa partição do cluster, quando parte do cluster ficar em pausa.  


# -- passos para subir um cluster rabbitmq num kind cluster --

./kind-with-registry.sh  

kubectl apply -f namespace.yaml  
kubectl apply -f rbac.yaml  

kubectl config set-context --current --namespace=test-rabbitmq  

kubectl config view --minify | grep namespace:  

kubectl apply -f rabbitmq-headless.yaml  

echo -n "this secret value is JUST AN EXAMPLE. Replace it" > cookie  
kubectl create secret generic erlang-cookie --from-file=./cookie  

criar uma secret usando um encoder base64 e o rabbitmq-admin.yaml  

--ou--  

echo -n "administrator" > user  
echo -n "g3N3rAtED-Pa$$w0rd" > pass  
kubectl create secret generic rabbitmq-admin --from-file=./user --from-file=./pass  

-- --  

kubectl apply -f configmap.yaml  

kubectl apply -f statefulset.yaml  

watch kubectl get all  


kubectl apply -f client-service.yaml  

kubectl get svc  

-- --  

kind delete cluster
