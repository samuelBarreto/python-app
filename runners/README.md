# GitHub Actions Runner Controller (ARC)

Este guia detalha a instalação e configuração do GitHub Actions Runner Controller no Kubernetes (Minikube).

## Pré-requisitos

- [Kubernetes cluster](https://kubernetes.io/docs/setup/) (Minikube)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)
- [cert-manager](https://cert-manager.io/docs/installation/) v1.8.2 ou superior
- Token de acesso do GitHub com permissões apropriadas

## Instalação do Cert Manager

O cert-manager é um pré-requisito para o Actions Runner Controller. Instale-o usando o seguinte comando:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
```

Verifique se os pods do cert-manager estão rodando:
```bash
kubectl get pods -n cert-manager
```

Você deve ver três pods em execução:
- cert-manager-cainjector
- cert-manager
- cert-manager-webhook

## Configuração do GitHub Token

1. Crie um Personal Access Token (PAT) no GitHub:
   - Acesse GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic)
   - Gere um novo token com as seguintes permissões:
     - `repo` (todo o escopo)
     - `admin:org` (se estiver usando organizações)
     - `admin:public_key`
     - `admin:repo_hook`
     - `admin:org_hook`
     - `notifications`

2. Guarde o token com segurança (exemplo do token atual: ghp_****)

## Instalação do Actions Runner Controller

1. Adicione o repositório Helm do ARC:
```bash
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
```

2. Instale o controller usando Helm:
```bash
helm upgrade --install --namespace actions-runner-system --create-namespace \
  --set=authSecret.create=true \
  --set=authSecret.github_token="seu_token_aqui" \
  --wait actions-runner-controller actions-runner-controller/actions-runner-controller
```

## Configuração dos Runners

1. Crie um arquivo `runners.yaml` para definir seus runners:

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: example-runner
  namespace: actions-runner-system
spec:
  replicas: 2
  template:
    spec:
      repository: samuelBarreto/python-app
      labels:
        - k8s-runner
---
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: example-runner-autoscaler
  namespace: actions-runner-system
spec:
  scaleTargetRef:
    name: example-runner
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
      repositoryNames:
        - samuelBarreto/python-app
```

2. Aplique a configuração:
```bash
kubectl apply -f runner-basic.yaml 
```

## Verificação da Instalação

1. Verifique os pods do controller:
```bash
kubectl get pods -n actions-runner-system
```

2. Verifique os runners em execução:
```bash
kubectl get runners -n actions-runner-system
```

3. Verifique os logs do controller:
```bash
kubectl logs -n actions-runner-system -l app.kubernetes.io/name=actions-runner-controller
```

## Uso nos Workflows

Para usar os runners em seus workflows GitHub Actions, adicione a label correspondente:

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: [k8s-runner]  # Use a label definida no RunnerDeployment
    steps:
      - uses: actions/checkout@v2
      # ... resto do seu workflow
```

## Troubleshooting

1. **Verificar status dos runners**:
```bash
kubectl describe runnerdeployment -n actions-runner-system
kubectl describe runners -n actions-runner-system
```

2. **Verificar logs do controller**:
```bash
kubectl logs -n actions-runner-system \
  -l app.kubernetes.io/name=actions-runner-controller
```

3. **Verificar eventos do namespace**:
```bash
kubectl get events -n actions-runner-system
```

kubectl apply  -n actions-runner-system -f runnerdeployment.yaml
runnerdeployment.actions.summerwind.dev/self-hosted-runners created


 kubectl get pods -n actions-runner-system
NAME                                        READY   STATUS              RESTARTS   AGE
actions-runner-controller-6f4c8db58-mx5zv   2/2     Running             0          5m56s
self-hosted-runners-g9k5x-sxcxp             0/2     ContainerCreating   0          29s



## Limpeza

Para remover o Actions Runner Controller:
```bash
helm uninstall actions-runner-controller -n actions-runner-system
kubectl delete namespace actions-runner-system
```

Para remover o cert-manager:
```bash
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
```

## Recursos Adicionais

- [Documentação oficial do ARC](https://github.com/actions/actions-runner-controller)
- [Guia de Autoscaling](https://github.com/actions/actions-runner-controller/blob/master/docs/automatically-scaling-runners.md)
- [Configurações avançadas](https://github.com/actions/actions-runner-controller/tree/master/docs)
