## Comandos Kubernates

### Pegar todos os namespaces
`kubectl get namespaces`

### Pegar todos o deploys
`kubectl get deployments --all-namespaces=true`

### Pegar o deploy de uma namespace especifico
`kubectl get deployments -n <nome-namespace>`

###  pods de um namespce especifico
`kubectl get pods -n <nome-namespace>`

###  Descrever todas as informações de um pod
`kubectl describe pods -n <nome-namespace> <pod-name>`

###  Descrever todos services de um namespace
`kubectl get service -n <nome-namespace>`

###  Descrever todos services e mais informações
`kubectl get service -n <nome-namespace> <service-name> -o yaml`

###  Limpar todos recursos de um namespace
`kubectl delete namespace <nome-namespace>`

###  Carregar certificado ssl
`kubectl create secret tls tls-secret --key tls.key --cert tls.crt`

###  listar varios recursos namespace
`kubectl get deploy,pod,rs,svc,ing -n <nome-namespace>`

###  listar ingress de um namespace
`kubectl get ingress -n gdt-dev`

### Mostra historico de deployments
`kubectl rollout history deployment/<nome-deployment> -n gdt-dev`

### Fazer rollback para a versão anterior
`kubectl rollout undo deploy my-deployment-name -n my-namespace`

### Fazer rollback para um versão especifica
`kubectl rollout undo deployment/<nome-deployment> -n <nome-namespace> --to-revision=12`

### Escalar numero de réplicas
`kubectl scale deploy <nome-deployment>.yaml -n <nome-namespace> --replicas=1`

### Mostrar detalhes de um ReplicationController
`kubectl describe replicationcontrollers/<nome-ReplicationController>`

### Logar no Container Registry
`az acr login -n <nome-registry>`

### Apagar Namespace travado
```
kubectl get namespace <nome-namespace> -o json > dados.json
nano dados.json
kubectl replace --raw "/api/v1/namespaces/<nome-namespace>/finalize" -f ./dados.json
```
### Apagar pod travado
```
kubectl delete --wait=false pod <pod-name> -n <nome-namespace>
```

###  Criar Secret no cluster
```
kubectl create secret tls tls-secret \
    --namespace gdt-prd \
    --key tls.key \
    --cert tls.crt
```


### Executar Comandos dentros do pod
`kubectl exec <pod-name> -n <nome-namespace> -- curl --location --request GET 'http://localhost/'`


### Fazer requisição para serviços de outro namespace
`curl http://<nome-service>.<nome-namespace>:8080/healthcheck`


### Copiar arquivo do Pod
`kubectl cp <nome-namespace>/<nome-pod>:<caminho-arquivo> <destino-local>`


### Copiar arquivo para o Pod
`kubectl cp <caminho-arquivo-local> <nome-namespace>/<nome-pod>:<caminho-arquivo>`


### Entrar no pod
**Linux com bash**

`kubectl exec -it <pod-name> -n <nome-namespace> -- /bin/bash`

**Alpine (`apk add curl`) inserir curl no alpine**

`kubectl exec -it <pod-name> -n <nome-namespace> -- /bin/ash`


### Pegar Logs de pod ( Método 1)
**Log em tempo real**

`kubectl logs -f <pod-name> -n <nome-namespace>`

**Logs capturado**

`kubectl logs <pod-name> -n <nome-namespace>`







