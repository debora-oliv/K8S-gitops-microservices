<!-- <p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p> -->
![Continuous Integration](https://github.com/GoogleCloudPlatform/microservices-demo/workflows/Continuous%20Integration%20-%20Main/Release/badge.svg)

## Objetivo
Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Rancher Desktop, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub. Também entender como aplicações são executadas em ambientes distribuídos, como escalar cargas, lidar com falhas e automatizar o ciclo de vida dos serviços com ferramentas para deploy automatizado e seguro, trazendo mais controle e rastreabilidade para os times de desenvolvimento e operações. Isso permitiu vivenciar na prática como as empresas modernas operam em ambientes cloudnative.

## Online Boutique Demo
**Online Boutique** é um aplicativo de demonstração de microsserviços disponibilizado abertamente pelo google. A aplicação é um comércio eletrônico baseado na web onde os usuários podem navegar pelos itens, adicioná-los ao carrinho e comprá-los. Este aplicativo funciona em qualquer cluster Kubernetes.

Para conferir mais detalhes sobre a arquitetura, microserviços, termos e licença acesse o [**repositório oficial**](https://github.com/GoogleCloudPlatform/microservices-demo/tree/main).

## Pré-requisitos
- [ ] Git instalado
- [ ] Kubectl instalado
- [ ] Rancher Desktop instalado (com Kubernetes habilitado)
- [ ] Docker funcionando localmente
- [ ] Conta no GitHub com repositório público

## Deployment
1. Faça um fork desse repositório

2. Altere o arquivo k8s/**kubernetes-application.yaml** para apontar para o seu repositório pessoal

    `repoURL` : `https://github.com/seu_usuario/seu_repositorio.git`

3. Configure o Argo CD no cluster local

    3.1 Criar um namespace
    ```sh
    kubectl create namespace argocd
    ```

    3.2 Instalar Argo CD no namespace criado
    ```sh
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

4. Instale o Argo CD CLI para acesso via linha de comando

    4.1.1 Instalar em ambiente **Windows**
    ```sh
    choco install argocd-cli
    ```

    4.1.2 Instalar em ambiente **Linux**
    ```sh
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

    rm argocd-linux-amd64
    ```

    4.2 Confirmar a instalação
    ```sh
    argocd version
    ```

5. Login no Argo CD via linha de comando

    5.1.1 Pegar senha inicial do Argo CD em ambiente **Windows**
    ```sh
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}")))
    ```

    5.1.2 Pegar senha inicial do Argo CD em ambiente **Linux**
    ```sh
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
    ```

    5.2 Port-forward para acessar o ArgoCD
    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

    5.3 Logar no Argo CD
    ```sh
    argocd login localhost:8080
    ```
    > Usuário: `admin`

    > Senha: `sua_senha_inicial`

6. Crie a aplicação
    ```sh
    kubectl apply -f k8s/boutique-application.yaml
    ```

7. Confira o status da aplicação
    ```sh
    kubectl get applications -n argocd
    ```