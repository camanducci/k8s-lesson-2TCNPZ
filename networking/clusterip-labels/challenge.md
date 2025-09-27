# Kubernetes Service e Label Selectors
## Este guia demonstra como um Service no Kubernetes usa labels (rótulos) para rotear o tráfego para os Pods corretos. O teste é dividido em duas partes: um cenário de sucesso onde o acesso funciona, e um cenário de falha onde o acesso é quebrado intencionalmente.

### O Cenário de Teste

1. Pod Alvo (pod-nginx): Um Pod com um servidor Nginx rodando. Ele tem a label curso: 2TCNPZ.

2. Service (svc-acesso-nginx): Um serviço do tipo ClusterIP que roteia o tráfego para qualquer Pod que tenha a label curso: 2TCNPZ.

3. Pod de Teste (pod-de-teste): Um Pod separado com a ferramenta curl para simular o acesso ao serviço.
---
### Passo 1: Preparação (Criação dos Recursos)

Crie todos os recursos necessários com um único comando. Copie o bloco abaixo e cole diretamente no seu terminal.


``` Bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    curso: 2TCNPZ
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: svc-acesso-nginx
spec:
  selector:
    curso: 2TCNPZ
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

---
apiVersion: v1
kind: Pod
metadata:
  name: pod-de-teste
spec:
  containers:
    - name: curl-container
      image: curlimages/curl
      command: ["sleep", "3600"]
EOF
```

### Passo 2: Teste de Acesso (Cenário de Sucesso)

Nesta etapa, confirmaremos que o Service está corretamente roteando o tráfego para o pod-nginx.

a) Verificar a Conexão

Use o comando kubectl get endpoints para ver a lista de Pods que o Service está "enxergando".


```Bash 
kubectl get endpoints svc-acesso-nginx
```

Saída Esperada: Você verá o IP do pod-nginx na lista de endpoints, confirmando que a conexão foi estabelecida com sucesso.

|NAME            |ENDPOINTS         |AGE. |
| ------          | ------        | ------ |
|svc-acesso-nginx |   10.244.0.12:80 |   1m|

b) Realizar o Acesso

Agora, execute o curl a partir do pod-de-teste para acessar o Service e, por consequência, o pod-nginx.

```Bash 
kubectl exec -it pod-de-teste -- curl svc-acesso-nginx
```
Saída Esperada: O comando será bem-sucedido e mostrará o código HTML da página padrão do Nginx.

### Passo 3: Teste de Falha (Removendo a Label)

Aqui, vamos quebrar a conexão entre o Service e o Pod removendo a label do pod-nginx.

a) Remover a Label

Use o comando kubectl label para remover a label curso.

```Bash 
kubectl label pod pod-nginx curso-
```

b) Verificar a Conexão Novamente
Verifique os endpoints do Service outra vez.

```Bash 
kubectl get endpoints svc-acesso-nginx
```
Saída Esperada: A lista de endpoints estará vazia, pois o Service não encontra mais nenhum Pod com a label curso: 2TCNPZ.

|NAME              | ENDPOINTS         |AGE|
| ------          | ------        | ------ |
|svc-acesso-nginx  | <none>      |      2m|

c) Tentar o Acesso Novamente

Repita o comando curl do pod-de-teste.

```Bash
kubectl exec -it pod-de-teste -- curl svc-acesso-nginx
```

Saída Esperada: O acesso irá falhar. A conexão será recusada porque o Service não tem mais nenhum destino para o qual rotear o tráfego.

Este experimento demonstra de forma clara e prática como a label é a chave para o roteamento de tráfego nos Services do Kubernetes.