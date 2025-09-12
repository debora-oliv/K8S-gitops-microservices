<!-- <p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p> -->
![Continuous Integration](https://github.com/GoogleCloudPlatform/microservices-demo/workflows/Continuous%20Integration%20-%20Main/Release/badge.svg)

# Objetivo

Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub. Também entender como aplicações são executadas em ambientes distribuídos, como escalar cargas, lidar com falhas e automatizar o ciclo de vida dos serviços com ferramentas para deploy automatizado e seguro, trazendo mais controle e rastreabilidade para os times de desenvolvimento e operações. Isso permitiu vivenciar na prática como as empresas modernas operam em ambientes cloudnative.

# Online Boutique Demo

**Online Boutique** é um aplicativo de demonstração de microsserviços disponibilizado abertamente pelo google. A aplicação é um comércio eletrônico baseado na web onde os usuários podem navegar pelos itens, adicioná-los ao carrinho e comprá-los. Este aplicativo funciona em qualquer cluster Kubernetes.

Para conferir mais detalhes sobre a arquitetura, microserviços, termos e licença acesse o [**repositório oficial**](https://github.com/GoogleCloudPlatform/microservices-demo/tree/main).

# Pré-requisitos

- [ ] Git instalado
- [ ] Kubectl instalado
- [ ] Rancher Desktop instalado (com Kubernetes habilitado)
- [ ] Docker funcionando localmente
- [ ] Conta no GitHub com repositório público

# Deployment

**1. Faça um fork desse repositório**

**2. Altere o arquivo [boutique-application.yaml](k8s/boutique-application.yaml) para apontar para o seu repositório pessoal**

> `repoURL` : `https://github.com/seu_usuario/seu_repositorio.git`

**3. Configurar o Argo CD no cluster local**

- Criar um namespace
    ```sh
    kubectl create namespace argocd
    ```
    
- Instalar Argo CD no namespace criado
    ```sh
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

- Conferir o status do Argo CD
    ```sh
    kubectl get all -n argocd
    ```
    --------------------------------------------------------
    ```
    NAME                                                    READY   STATUS    RESTARTS   AGE
    pod/argocd-application-controller-0                     1/1     Running   0          82m
    pod/argocd-applicationset-controller-54f96997f8-6nzkz   1/1     Running   0          82m
    pod/argocd-dex-server-798cbff4c7-8c99s                  1/1     Running   0          82m
    pod/argocd-notifications-controller-644f66f7df-7lpsj    1/1     Running   0          82m
    pod/argocd-redis-6684c6947f-558hg                       1/1     Running   0          82m
    pod/argocd-repo-server-6fccc5759b-92q7p                 1/1     Running   0          82m
    pod/argocd-server-64d5fcbd58-vsqpq                      1/1     Running   0          82m

    NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
    service/argocd-applicationset-controller          ClusterIP   10.43.117.133   <none>        7000/TCP,8080/TCP            82m
    service/argocd-dex-server                         ClusterIP   10.43.170.127   <none>        5556/TCP,5557/TCP,5558/TCP   82m
    service/argocd-metrics                            ClusterIP   10.43.207.241   <none>        8082/TCP                     82m
    service/argocd-notifications-controller-metrics   ClusterIP   10.43.23.6      <none>        9001/TCP                     82m
    service/argocd-redis                              ClusterIP   10.43.109.0     <none>        6379/TCP                     82m
    service/argocd-repo-server                        ClusterIP   10.43.238.79    <none>        8081/TCP,8084/TCP            82m
    service/argocd-server                             ClusterIP   10.43.202.172   <none>        80/TCP,443/TCP               82m
    service/argocd-server-metrics                     ClusterIP   10.43.46.148    <none>        8083/TCP                     82m

    NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/argocd-applicationset-controller   1/1     1            1           82m
    deployment.apps/argocd-dex-server                  1/1     1            1           82m
    deployment.apps/argocd-notifications-controller    1/1     1            1           82m
    deployment.apps/argocd-redis                       1/1     1            1           82m
    deployment.apps/argocd-repo-server                 1/1     1            1           82m
    deployment.apps/argocd-server                      1/1     1            1           82m

    NAME                                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/argocd-applicationset-controller-54f96997f8   1         1         1       82m
    replicaset.apps/argocd-dex-server-798cbff4c7                  1         1         1       82m
    replicaset.apps/argocd-notifications-controller-644f66f7df    1         1         1       82m
    replicaset.apps/argocd-redis-6684c6947f                       1         1         1       82m
    replicaset.apps/argocd-repo-server-6fccc5759b                 1         1         1       82m
    replicaset.apps/argocd-server-64d5fcbd58                      1         1         1       82m

    NAME                                             READY   AGE
    statefulset.apps/argocd-application-controller   1/1     82m
    ```

**4. Instalar o Argo CD CLI para acesso via linha de comando**

- Instalar CLI
  
  - **Windows**
    
    ```sh
    choco install argocd-cli
    ```
    
  - **Linux**
    
    ```sh
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

    rm argocd-linux-amd64
    ```

- Conferir a instalação
  
    ```sh
    argocd version
    ```
    --------------------------------------------------------
    ```
    argocd: v3.1.3+c1467b8
        BuildDate: 2025-09-04T18:00:33Z
        GitTreeState: clean
        GoVersion: go1.24.6
        Compiler: gc
    ```

**5. Login no Argo CD via linha de comando**

- Pegar senha inicial do Argo CD
  
  - **Windows**
    
    ```sh
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}")))
    ```
  
  - **Linux**
    
    ```sh
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
    ```
    
- Port-forward para acessar o ArgoCD
  
    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

- Logar no Argo CD
  
    ```sh
    argocd login localhost:8080
    ```
    > Usuário: `admin`
    > 
    > Senha: `sua_senha_inicial`

**6. Criar a aplicação [](#2.altere-o-arquivo-boutique-application.yaml)**
    
- Criação automatizada via yaml
  
    ```sh
    kubectl apply -f k8s/boutique-application.yaml
    ```
    
- Conferir criação da aplicação
  
    ```sh
    kubectl get all -n app-boutique
    ```
    --------------------------------------------------------
    ```
    NAME                                         READY   STATUS    RESTARTS   AGE
    pod/adservice-54fdcb4646-jrt6g               1/1     Running   0          85m
    pod/cartservice-7d76bb9df-c676h              1/1     Running   0          85m
    pod/checkoutservice-5d9d84cd44-lsdf7         1/1     Running   0          85m
    pod/currencyservice-569f6c566d-7bsck         1/1     Running   0          85m
    pod/emailservice-7d4b8cd7d6-w9jf9            1/1     Running   0          85m
    pod/frontend-76dbbddfc5-hkmzd                1/1     Running   0          85m
    pod/loadgenerator-56674fd696-9l6ff           1/1     Running   0          85m
    pod/paymentservice-9ff6ffd6-6wk9z            1/1     Running   0          85m
    pod/productcatalogservice-74c67b9d8b-7sxq6   1/1     Running   0          85m
    pod/recommendationservice-5966b9f59d-s848m   1/1     Running   0          85m
    pod/redis-cart-c4fc658fb-kxlbh               1/1     Running   0          85m
    pod/shippingservice-5565748dc4-s92d5         1/1     Running   0          85m

    NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    service/adservice               ClusterIP   10.43.88.102    <none>        9555/TCP       85m
    service/cartservice             ClusterIP   10.43.119.48    <none>        7070/TCP       85m
    service/checkoutservice         ClusterIP   10.43.251.161   <none>        5050/TCP       85m
    service/currencyservice         ClusterIP   10.43.205.155   <none>        7000/TCP       85m
    service/emailservice            ClusterIP   10.43.167.154   <none>        5000/TCP       85m
    service/frontend                ClusterIP   10.43.223.224   <none>        80/TCP         85m
    service/frontend-external       NodePort    10.43.191.77    <none>        80:32496/TCP   85m
    service/paymentservice          ClusterIP   10.43.229.74    <none>        50051/TCP      85m
    service/productcatalogservice   ClusterIP   10.43.39.122    <none>        3550/TCP       85m
    service/recommendationservice   ClusterIP   10.43.85.193    <none>        8080/TCP       85m
    service/redis-cart              ClusterIP   10.43.125.150   <none>        6379/TCP       85m
    service/shippingservice         ClusterIP   10.43.179.124   <none>        50051/TCP      85m

    NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/adservice               1/1     1            1           85m
    deployment.apps/cartservice             1/1     1            1           85m
    deployment.apps/checkoutservice         1/1     1            1           85m
    deployment.apps/currencyservice         1/1     1            1           85m
    deployment.apps/emailservice            1/1     1            1           85m
    deployment.apps/frontend                1/1     1            1           85m
    deployment.apps/loadgenerator           1/1     1            1           85m
    deployment.apps/paymentservice          1/1     1            1           85m
    deployment.apps/productcatalogservice   1/1     1            1           85m
    deployment.apps/recommendationservice   1/1     1            1           85m
    deployment.apps/redis-cart              1/1     1            1           85m
    deployment.apps/shippingservice         1/1     1            1           85m

    NAME                                               DESIRED   CURRENT   READY   AGE
    replicaset.apps/adservice-54fdcb4646               1         1         1       85m
    replicaset.apps/cartservice-7d76bb9df              1         1         1       85m
    replicaset.apps/checkoutservice-5d9d84cd44         1         1         1       85m
    replicaset.apps/currencyservice-569f6c566d         1         1         1       85m
    replicaset.apps/emailservice-7d4b8cd7d6            1         1         1       85m
    replicaset.apps/frontend-76dbbddfc5                1         1         1       85m
    replicaset.apps/loadgenerator-56674fd696           1         1         1       85m
    replicaset.apps/paymentservice-9ff6ffd6            1         1         1       85m
    replicaset.apps/productcatalogservice-74c67b9d8b   1         1         1       85m
    replicaset.apps/recommendationservice-5966b9f59d   1         1         1       85m
    replicaset.apps/redis-cart-c4fc658fb               1         1         1       85m
    replicaset.apps/shippingservice-5565748dc4         1         1         1       85m
    ```
    
- Conferir o status da aplicação

    ```sh
    kubectl get applications -n argocd
    ```
    --------------------------------------------------------
    ```
    NAME           SYNC STATUS   HEALTH STATUS
    app-boutique   Synced        Healthy
    ```
    
**7. Acesse a aplicação**

No seu browser: `localhost:30080`

**8. Acesse a interface do Argo CD**

No seu browser: `localhost:8080`
    
