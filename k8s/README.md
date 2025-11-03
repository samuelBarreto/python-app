# Python App Kubernetes Setup Guide

Este guia explica como configurar e acessar a aplicação Python usando Kubernetes (Minikube) com Ingress.

## Pré-requisitos

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Docker](https://docs.docker.com/get-docker/)

## Configuração Inicial

1. Inicie o Minikube:
```bash
minikube start --driver=docker
```

2. Habilite o addon do Ingress:
```bash
minikube addons enable ingress
```

## Deploy da Aplicação

1. Aplique os manifestos Kubernetes:
```bash
kubectl apply -f deploy.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

2. Verifique se tudo está rodando:
```bash
kubectl get pods
kubectl get svc
kubectl get ing
```

## Acesso à Aplicação

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

3. Acesse a aplicação:
   ```
   http://python-app.test.com/api/v1/info
   ```

   Alternativa (usando NodePort):
   ```powershell
   # Obter URL do serviço
   minikube service python-app --url
   # Acesse a URL retornada adicionando /api/v1/info
   # Exemplo: http://127.0.0.1:54433/api/v1/info
   ```

### Linux

1. Configure o arquivo hosts:
   ```bash
   # Adicione a entrada no arquivo /etc/hosts
   sudo echo "$(minikube ip) python-app.test.com" | sudo tee -a /etc/hosts
   ```

2. Inicie o Minikube tunnel:
   ```bash
   # Em um novo terminal
   sudo minikube tunnel
   ```

3. Acesse a aplicação:
   ```bash
   curl http://python-app.test.com/api/v1/info
   # Ou acesse no navegador: http://python-app.test.com/api/v1/info
   ```

## Troubleshooting

### Verificar Status e Saúde do Ingress

1. Verifique o status do Ingress:
```bash
# Status básico do Ingress
kubectl get ing

# Descrição detalhada do Ingress
kubectl describe ing python-app
```

2. Verifique a saúde do Ingress Controller:
```bash
# Status dos pods do Ingress Controller
kubectl get pods -n ingress-nginx

# Detalhes do pod do Ingress Controller
kubectl describe pod -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Verificar eventos relacionados ao Ingress
kubectl get events -n ingress-nginx --sort-by='.lastTimestamp'
```

3. Logs e Diagnóstico do Ingress:
```bash
# Logs do Ingress Controller
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Logs com mais detalhes (últimas 100 linhas)
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=100 -f

# Verificar configuração do nginx
kubectl exec -it -n ingress-nginx $(kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}') -- nginx -T
```

4. Teste a conectividade:
```bash
# Teste dentro do cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://python-app:8080/api/v1/info

# Verificar endpoints do serviço
kubectl get endpoints python-app
```

5. Verificar configuração do TLS (se estiver usando HTTPS):
```bash
# Listar certificados
kubectl get secrets -n ingress-nginx

# Verificar detalhes do certificado
kubectl describe secret -n ingress-nginx <nome-do-secret>
```

6. Status dos serviços relacionados:
```bash
# Verificar serviços no namespace do ingress-nginx
kubectl get svc -n ingress-nginx

# Verificar detalhes do serviço do Ingress Controller
kubectl describe svc -n ingress-nginx ingress-nginx-controller
```

### Problemas Comuns

1. **Não consegue acessar via hostname**
   - Verifique se o arquivo hosts está configurado corretamente
   - Certifique-se que o minikube tunnel está rodando
   - Teste usando o IP direto: `curl -H "Host: python-app.test.com" http://$(minikube ip)/api/v1/info`

2. **Erro 404**
   - Verifique se está acessando o endpoint correto (/api/v1/info)
   - Verifique os logs da aplicação: `kubectl logs deployment/python-app`

3. **Connection Refused**
   - Verifique se o minikube tunnel está rodando
   - Reinicie o tunnel se necessário
   - No Windows, certifique-se de rodar o tunnel como Administrador

## Endpoints Disponíveis

- `/api/v1/info` - Retorna informações sobre o host e timestamp
- `/api/v1/healthz` - Health check da aplicação

## Limpeza

Para remover tudo:
```bash
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deploy.yaml
```