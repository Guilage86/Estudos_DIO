## Checklist de estudos

- [ ] Entender **Pod, Deployment, Service** e **Ingress**.

- [ ] Subir um cluster local com **Minikube**.

- [ ] Implantar um **Deployment** e expor via **Service**.

- [ ] Habilitar **Dashboard** do Minikube e explorar métricas.

- [ ] Simular **rollout/rollback** e testar **probes**.

- [ ] Ler sobre **RBAC**, **NetworkPolicy** e **HPA**.

- [ ] Estudar **backup/restore** do **etcd**.



# Curso — Introdução a Kubernetes e Orquestração de Contêineres

> Objetivo do curso

- Entender **o que é Kubernetes (K8s)** e quando usá-lo.
- Conhecer **conceitos fundamentais** (cluster, nó, pod, etc.).
- Montar **um ambiente local** com Minikube e `kubectl`.
- Ter uma visão de **arquitetura de produção** e boas práticas.

------

## Por que Kubernetes?

- **Modernização**: migração de monólitos para **microserviços**.
- **Alta disponibilidade**: menos **downtime** e **self-healing**.
- **Escalabilidade**: escala horizontal/vertical de forma automatizada.
- **Observabilidade e gestão**: implantação, atualização e rollback padronizados.
- **Recuperação de desastre**: estratégias de **backup/restore** e redundância.

------

## Módulo 1 — Introdução ao Kubernetes

### O que é o Kubernetes?

Kubernetes é uma **plataforma open source** para **orquestrar contêineres**, automatizando **implantação**, **escalonamento** e **gerenciamento** de aplicações.

### Conceitos essenciais

- **Cluster**: conjunto de máquinas (físicas/virtuais) executando Kubernetes.
- **Plano de Controle (Control Plane)**: camada que **decide** e **orquestra** (API Server, Scheduler, etcd, Controller Manager).
- **Nós (Nodes)**:
  - **Master/Control Plane**: hospeda os componentes de controle.
  - **Workers**: executam as cargas de trabalho (pods).
- **Pod**: **menor unidade** do Kubernetes; encapsula **um ou mais contêineres** que compartilham rede/armazenamento.
- **Deployment**: controla **réplicas** e **atualizações** de pods.
- **Service**: expõe pods de forma estável (ClusterIP/NodePort/LoadBalancer).
- **ConfigMap/Secret**: configuração e segredos desacoplados do container.
- **Namespace**: isolamento lógico de recursos no cluster.

### O que é um Pod?

Um **Pod** é um agrupamento de **um ou mais contêineres** que compartilham **rede** (IP/porta) e **volumes**. Ele é a **unidade mínima agendável** no K8s — normalmente **1 app por pod**.

> Ex.: um sidecar para **logs** ou **proxy** pode rodar no mesmo pod do app principal.

#### Exemplo (Deployment + Service)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: web
          image: nginx:stable
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
spec:
  selector:
    app: webapp
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
```

------

## Módulo 2 — Ambiente de Desenvolvimento Kubernetes (Minikube)

### O que é o Minikube?

**Minikube** executa um **cluster Kubernetes local** (1 nó por padrão). Ótimo para **estudo e testes**.

### Pré-requisitos rápidos

- Virtualização habilitada (BIOS/UEFI).
- Um **driver**: Docker, **VirtualBox**, Hyper-V (Windows) ou QEMU/KVM (Linux).
- Recomendado: **Docker Desktop** (Windows/macOS) ou Docker Engine (Linux).

### Instalação

- **Minikube (oficial)**: https://minikube.sigs.k8s.io/docs/start/
- **kubectl (oficial)**: https://kubernetes.io/pt-br/docs/tasks/tools/

> Dica (Windows): é comum usar o **driver Docker** (WSL2) ou **VirtualBox**.

### Subindo o cluster local

```
# Escolha 1: usando Docker como driver
minikube start --driver=docker

# Escolha 2: usando VirtualBox
minikube start --driver=virtualbox
```

### Verificação básica

```
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

### Comandos úteis do Minikube

```
minikube status
minikube dashboard    # UI web de observabilidade local
minikube stop
minikube delete
```

### Problemas comuns (e correções)

- **“minikube não é reconhecido”**: verifique se o **binário está no PATH** (reinicie o terminal após instalar).

- **Driver não encontrado**: instale/ative **Docker**, **VirtualBox** ou outro driver compatível.

- **WSL2/Hyper-V conflito** (Windows): use **um** deles por vez; prefira **WSL2 + Docker** para simplicidade.

- **kubectl aponta para outro cluster**: ajuste o **contexto**:

  ```
  kubectl config get-contexts
  kubectl config use-context minikube
  ```

------

## Módulo 3 — Cluster de Produção em Cloud

### Organização geral

- O **Plano de Controle** roda nos **masters** (replique para **alta disponibilidade**).
- Os **Workers** hospedam **pods** da aplicação.
- Use **Múltiplas Zonas/Regiões** quando possível (cloud) e **balanceadores gerenciados**.

### Componentes do Control Plane

| Componente                  | Função resumida                                              |
| --------------------------- | ------------------------------------------------------------ |
| **kube-apiserver**          | Porta de entrada do cluster (autenticação, validação e orquestração). |
| **etcd**                    | **Banco de dados chave-valor** com o **estado do cluster**. Fundamental para backups. |
| **kube-scheduler**          | Decide **em qual nó** cada pod deve rodar (recursos, afinidade/anti-afinidade, taints/tolerations). |
| **kube-controller-manager** | Controladores que convergem o **estado atual** ao **estado desejado** (deployments, endpoints, etc.). |

### Add-ons e plano de dados

- **kube-proxy** (ou CNI avançada) para rede de serviços.
- **CNI** (Calico, Cilium, Flannel…) para **rede dos pods** e políticas de segurança.
- **CoreDNS** para resolução interna de nomes.
- **Ingress Controller** (Nginx, Traefik, etc.) para roteamento HTTP/HTTPS.

### Boas práticas de produção

- **Alta disponibilidade**: múltiplos masters e backups periódicos do **etcd**.

- **Backups/Restore** (etcd):

  ```
  # Snapshot do etcd (exemplo)
  ETCDCTL_API=3 etcdctl snapshot save backup.db \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/etcd/ca.crt \
    --cert=/etc/etcd/etcd.crt \
    --key=/etc/etcd/etcd.key
  ```

- **Observabilidade**: métricas (Prometheus), logs centralizados (ELK/Loki) e tracing (OpenTelemetry).

- **Segurança**: RBAC, Namespaces, NetworkPolicies, Secrets, PodSecurity (ou PSP/alternativas), assinaturas de imagens.

- **Ciclo de vida**: Deployments com **readiness/liveness/startup probes**, **HPA** (Horizontal Pod Autoscaler), **rolling update** com **rollbacks**.

------

## Comandos essenciais do `kubectl`

```
# Listagens
kubectl get nodes
kubectl get pods -A
kubectl get deploy,svc,ingress -n meu-namespace

# Detalhes e logs
kubectl describe pod <nome>
kubectl logs <pod> -n <ns>
kubectl logs -f <pod> -c <container> -n <ns>

# Aplicar/atualizar recursos
kubectl apply -f arquivo.yaml
kubectl rollout status deploy/<nome> -n <ns>
kubectl rollout undo deploy/<nome> -n <ns>

# Acesso e debug
kubectl exec -it <pod> -- /bin/sh
kubectl port-forward svc/webapp-svc 8080:80 -n <ns>

# Contextos e namespaces
kubectl config get-contexts
kubectl config use-context <contexto>
kubectl config set-context --current --namespace=<ns>
```

------

## Glossário rápido

- **Contêiner**: processo isolado com dependências empacotadas (ex.: imagem Docker).
- **Pod**: 1+ contêineres que compartilham rede/volumes; unidade agendável.
- **Service**: IP/porta estáveis para acessar pods.
- **Ingress**: regra de entrada HTTP/HTTPS para serviços internos.
- **ConfigMap/Secret**: dados de configuração/segredos injetados no pod.
- **ReplicaSet/Deployment**: garantem quantidade de réplicas e atualizações.
- **StatefulSet**: aplicações com **identidade** e **armazenamento estável**.
- **DaemonSet**: 1 pod **por nó**, para agentes/sidecars de cluster.
- **Namespace**: compartimentos lógicos de recursos dentro do cluster.

------

## Links oficiais

- Minikube: https://minikube.sigs.k8s.io/docs/start/
- kubectl: https://kubernetes.io/pt-br/docs/tasks/tools/
- Documentação Kubernetes: https://kubernetes.io/pt-br/docs/home/

------

- [ ] 