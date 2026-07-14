# Golang API CRUD - Kubernetes Manifests & GitOps

Repositório de **manifests Kubernetes** para deploy de uma API CRUD desenvolvida em **Go (Golang)**, organizados com **Kustomize** e preparados para deploy contínuo via **GitOps (ArgoCD)**.

Este repositório contém apenas a camada de infraestrutura/deploy (manifests). O código-fonte da aplicação está em um repositório separado: *[link do repo da API, se público]*.

## Sobre o projeto

O objetivo deste projeto foi praticar e demonstrar conhecimentos de:

- Estruturação de manifests Kubernetes seguindo boas práticas (separação `base` + `overlays`)
- Gerenciamento de múltiplos ambientes (dev/staging/prod) sem duplicação de código YAML
- Deploy declarativo via GitOps, usando ArgoCD como ferramenta de entrega contínua
- Deploy de uma aplicação containerizada (API Go) em um cluster Kubernetes gerenciado

## Estrutura do repositório

```
.
├── base/
│   ├── App/
│   │   ├── configmap.yml            # Variáveis de ambiente da aplicação
│   │   ├── deployment.yml           # Deployment da API Go
│   │   └── horizontal-autoscaler.yml # HPA (escalonamento automático de pods)
│   ├── Cluster/
│   │   ├── pod-disruption-budget.yml # Garante disponibilidade mínima durante updates/manutenção
│   │   ├── service-account.yml       # Identidade da aplicação dentro do cluster
│   │   └── service.yml               # Service interno (ClusterIP)
│   ├── Policy/
│   │   └── network-policy.yml        # Regras de tráfego permitido entre pods
│   ├── Secret/
│   │   └── secret.example.yml        # Exemplo de estrutura de Secret (valores reais não versionados)
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── namespace.yml
│   │   └── service-external.yml      # Exposição externa específica de dev
│   └── prod/
│       ├── kustomization.yaml
│       ├── namespace.yml
│       └── service-external.yml      # Exposição externa específica de prod
├── applications.yaml                 # Application manifest do ArgoCD (GitOps)
└── README.md
```

> **Sobre Secrets:** cada ambiente (dev/prod) possui sua própria Secret com credenciais reais, mas elas **não são versionadas neste repositório** por segurança. Apenas um `secret.example.yml` é mantido como referência de estrutura — as Secrets reais devem ser criadas manualmente ou via um gerenciador externo (ex: Sealed Secrets, External Secrets Operator, Azure Key Vault) antes do deploy.

### Por que essa organização?

- **`App/`** concentra tudo relacionado diretamente à aplicação (Deployment, ConfigMap, autoscaling)
- **`Cluster/`** concentra recursos de infraestrutura do cluster que dão suporte à aplicação (Service Account, PDB, Service interno)
- **`Policy/`** isola regras de segurança de rede (Network Policy), facilitando auditoria separada
- **`Secret/`** mantém apenas um exemplo versionado, sem expor credenciais reais no Git

### Por que Kustomize?

Ao invés de duplicar arquivos YAML para cada ambiente, o Kustomize permite manter uma configuração **base** comum e aplicar **patches** (overlays) específicos por ambiente — neste projeto, cada overlay (`dev`/`prod`) define seu próprio **namespace** e sua própria estratégia de **exposição externa** (`service-external.yml`), sem duplicar o restante dos manifests (Deployment, ConfigMap, HPA, etc.), que permanecem centralizados em `base/`.

### Por que ArgoCD (GitOps)?

O arquivo `applications.yaml` define uma **Application** do ArgoCD, que monitora este repositório e sincroniza automaticamente o estado do cluster com o que está declarado no Git. Isso segue o princípio de GitOps: o **Git como fonte única da verdade** — qualquer mudança de infraestrutura passa por commit/PR, com histórico e possibilidade de rollback via `git revert`.

## Tecnologias e conceitos aplicados

- **Go (Golang)** — linguagem da API CRUD
- **Kubernetes** — orquestração dos containers
- **Kustomize** — gerenciamento de manifests por ambiente (base + overlays)
- **ArgoCD** — entrega contínua via GitOps
- **Docker** — containerização da aplicação
- **Horizontal Pod Autoscaler (HPA)** — escalonamento automático de réplicas conforme uso de CPU/memória
- **Pod Disruption Budget (PDB)** — garante um número mínimo de pods disponíveis durante manutenções e rolling updates
- **Service Account** — identidade dedicada da aplicação dentro do cluster, seguindo o princípio de menor privilégio
- **Network Policy** — controle de tráfego de rede permitido entre pods (segurança de rede a nível de cluster)
- **Secrets & ConfigMaps** — separação entre configuração não sensível (ConfigMap) e credenciais (Secret), com exemplo versionado sem expor valores reais

## Próximos passos (em andamento)

Este projeto está sendo expandido para praticar deploy **multi-cloud**:

- [ ] Deploy do cluster no **Azure Kubernetes Service (AKS)**, provisionando a infraestrutura e conectando este repositório via ArgoCD
- [ ] Documentação do passo a passo de provisionamento (rede, cluster, integração com Azure Container Registry)
- [ ] Réplica do mesmo deploy no **Amazon EKS**, para comparar as duas abordagens (AKS vs EKS) e documentar as equivalências de arquitetura entre as clouds

A ideia é usar este mesmo conjunto de manifests como base para deploys equivalentes em diferentes provedores de nuvem, documentando as diferenças práticas entre eles (rede, registry de imagens, autenticação, storage).

## Como usar

```bash
# 1. Criar as Secrets reais do ambiente escolhido (não versionadas neste repo)
kubectl apply -f secrets-reais.yml -n <namespace-do-ambiente>

# 2. Aplicar os manifests do ambiente com Kustomize (ex: dev)
kubectl apply -k overlays/dev

# Ou registrar a Application no ArgoCD, para que o cluster seja sincronizado
# automaticamente a partir deste repositório (GitOps)
kubectl apply -f applications.yaml
```
