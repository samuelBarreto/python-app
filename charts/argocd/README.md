# Python App Helm Chart

Este é um guia de instalação e configuração do Python App usando Helm no Kubernetes (Minikube).

## Pré-requisitos

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Docker](https://docs.docker.com/get-docker/)

## Configuração Inicial

1. Inicie o Minikube:
```bash
minikube start
```

2. Habilite o addon do Ingress:
```bash
minikube addons enable ingress
```

## Instalação com Helm

1. Navegue até o diretório do chart:
```bash
cd python-app/charts/python-app
```

2. Instale o chart:
```bash
helm install python-app . -n python --create-namespace
```

3. Verifique a instalação:
```bash
# Verificar pods
kubectl get pods -n python -w

# Verificar serviços
kubectl get svc -n python

# Verificar ingress
kubectl get ing -n python
```

## Configuração do Acesso

### Windows

1. Configure o arquivo hosts:
   - Abra PowerShell como Administrador
   - Edite o arquivo `C:\Windows\System32\drivers\etc\hosts`
   - Adicione a linha:
   ```
   127.0.0.1 python-app.test.com
   ```

2. Inicie o Minikube tunnel:
   ```powershell
   # Em uma nova janela do PowerShell como Administrador
   minikube tunnel
   ```

### Linux

1. Configure o arquivo hosts:
   ```bash
   sudo echo "$(minikube ip) python-app.test.com" | sudo tee -a /etc/hosts
   ```

2. Inicie o Minikube tunnel:
   ```bash
   sudo minikube tunnel
   ```

## Acessando a Aplicação

A aplicação estará disponível em:
```
http://python-app.test.com/api/v1/info
```

## Verificação de Status

1. Status dos Pods:
```bash
kubectl get pods -n python
```

2. Status do Serviço:
```bash
kubectl get svc -n python
```

3. Status do Ingress:
```bash
kubectl get ing -n python
```

4. Logs da Aplicação:
```bash
kubectl logs -n python -l app.kubernetes.io/name=python-app
```

## Endpoints Disponíveis

- `/api/v1/info` - Retorna informações sobre o host e timestamp
- `/api/v1/healthz` - Health check da aplicação

## Configurações do Chart

O chart inclui:
- Deployment com 1 réplica
- Service do tipo ClusterIP na porta 5000
- Ingress configurado para o host python-app.test.com
- Health checks configurados em /api/v1/healthz
- ServiceAccount dedicada

## Desinstalação

Para remover a aplicação:
```bash
helm uninstall python-app -n python
```

## Troubleshooting

1. **Pods não iniciam:**
```bash
kubectl describe pod -n python <nome-do-pod>
kubectl get events -n python --sort-by=.metadata.creationTimestamp
```

2. **Ingress não funciona:**
```bash
kubectl get events -n python --sort-by=.metadata.creationTimestamp
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

3. **Serviço inacessível:**
```bash
# Teste dentro do cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -n python -- wget -O- http://python-app:5000/api/v1/info
```

## Valores Padrão

Os principais valores configurados no `values.yaml`:
- Porta do serviço: 5000
- Ingress habilitado: true
- Classe do Ingress: nginx
- Health checks configurados
- ServiceAccount: python-app

Para customizar a instalação, você pode criar um arquivo `custom-values.yaml` e instalá-lo com:
```bash
helm install python-app . -n python -f custom-values.yaml
```
