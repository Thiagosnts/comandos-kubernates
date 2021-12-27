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

### Comando para desfazer o deployment
`kubectl rollout undo deploy my-deployment-name -n my-namespace`

###  Criar Secret no cluster
```
kubectl create secret tls tls-secret \
    --namespace gdt-prd \
    --key tls.key \
    --cert tls.crt
```


### Executar Comandos dentros do pod
`kubectl exec <pod-name> -n <nome-namespace> -- curl --location --request GET 'http://localhost/'`

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







