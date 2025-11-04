# Argo CD Installation Guide

Este guia detalha a instalação e configuração do Argo CD usando Helm no Kubernetes (Minikube).

## Pré-requisitos

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/)

## Configuração Inicial

1. Adicione o repositório Helm do Argo CD:
```bash

helm repo add argo https://argoproj.github.io/argo-helm; helm repo update


```

2. Certifique-se que o Ingress está habilitado no Minikube:
```bash
minikube addons enable ingress
```

## Instalação

1. Configure o arquivo `values.yaml`:
```yaml
redis-ha:
  enabled: false

controller:
  replicas: 1

server:
  autoscaling:
    enabled: true
    minReplicas: 1

repoServer:
  autoscaling:
    enabled: true
    minReplicas: 1

applicationSet:
  replicas: 1

global:
  domain: argocd.test.com

certificate:
  enabled: true

server:
  ingress:
    annotations:
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    enabled: true
    ingressClassName: nginx
    tls: true
```

2. Instale o Argo CD:
```bash
helm upgrade --install argocd argo/argo-cd -n argocd --create-namespace -f values.yaml
```

## Verificação da Instalação

1. Verifique os pods:
```bash
kubectl get pods -n argocd
```
Você deve ver os seguintes pods:
- argocd-application-controller
- argocd-applicationset-controller
- argocd-dex-server
- argocd-notifications-controller
- argocd-redis
- argocd-repo-server
- argocd-server

2. Verifique o Ingress:
```bash
kubectl get ing -n argocd
```

## Configuração de Acesso

### Windows

1. Configure o arquivo hosts:
   - Abra PowerShell como Administrador
   - Edite o arquivo `C:\Windows\System32\drivers\etc\hosts`
   - Adicione a linha:
   ```
   127.0.0.1 argocd.test.com
   ```

2. Inicie o Minikube tunnel:
```powershell
# Em uma nova janela do PowerShell como Administrador
minikube tunnel
```

### Linux

1. Configure o arquivo hosts:
```bash
sudo echo "$(minikube ip) argocd.test.com" | sudo tee -a /etc/hosts
```

2. Inicie o Minikube tunnel:
```bash
sudo minikube tunnel
```

## Primeiro Acesso

1. Obtenha a senha inicial do admin:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

2. Acesse a interface web:
- URL: https://argocd.test.com
- Usuário: admin
- Senha: (obtida no comando anterior)

## Configuração da CLI do Argo CD

1. Faça login via CLI:
```bash
argocd login argocd.test.com
```

2. Troque a senha do admin:
```bash
argocd account update-password
```

## Troubleshooting

1. **Verificar logs do servidor**:
```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
```

2. **Verificar eventos do namespace**:
```bash
kubectl get events -n argocd --sort-by=.metadata.creationTimestamp
```

3. **Verificar status do Ingress**:
```bash
kubectl describe ing argocd-server -n argocd
```

## Limpeza

Para desinstalar o Argo CD:
```bash
helm uninstall argocd -n argocd
kubectl delete namespace argocd
```

## Recursos Adicionais

- [Documentação oficial do Argo CD](https://argo-cd.readthedocs.io/)
- [Valores do Helm Chart](https://github.com/argoproj/argo-helm/tree/master/charts/argo-cd)
- [Guia de Segurança](https://argo-cd.readthedocs.io/en/stable/operator-manual/security/)