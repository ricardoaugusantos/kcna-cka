
# Laboratorio Kubernetes — KCNA e CKA

Repositorio de estudos praticos de Kubernetes, com foco inicial nos fundamentos da **KCNA (Kubernetes and Cloud Native Associate)** e evolucao para a **CKA (Certified Kubernetes Administrator)**.

O projeto utiliza manifestos YAML versionados em Git e um cluster local criado com **Kind (Kubernetes in Docker)**, no ambiente **WSL2/Ubuntu**.

## Objetivos

- Aprender a definir e gerenciar recursos Kubernetes de forma declarativa.
- Entender o funcionamento do control plane, dos controllers e do scheduler.
- Praticar diagnostico de falhas com `kubectl`.
- Evoluir para Services, DNS, networking, armazenamento, seguranca e administracao de clusters.

## Ambiente do laboratorio

- WSL2 com Ubuntu
- Docker
- Kind
- kubectl
- VS Code e Git

Cluster Kind: `kcna-cka-lab`, com **1 control plane e 2 workers**.

> A configuracao do cluster esta em `infra/kind/cluster.yaml`. Criar o cluster exige Docker e Kind instalados. O arquivo de configuracao do Kind nao altera automaticamente um cluster ja existente.

## Estrutura do repositorio

```text
kcna-cka/
├── infra/
│   └── kind/
│       └── cluster.yaml
├── modules/
│   ├── 01-control-plane/
│   │   └── manifests/
│   │       ├── deployment.yaml
│   │       ├── pod-pending.yaml
│   │       ├── pod-sem-toleration.yaml
│   │       ├── pod-com-toleration.yaml
│   │       └── worker02-taint.yaml
│   └── 02-networking/       # Proximo modulo
├── scripts/
├── .gitignore
└── README.md
```

## Modulo 01 — Fundamentos e Scheduling

Exercicios praticados:

1. Criacao de cluster Kind com tres nos.
2. Criacao e escalabilidade de Deployment com Nginx.
3. Relacao entre Deployment, ReplicaSet e Pods.
4. Restricao de agendamento com `nodeSelector`.
5. Investigacao de Pods em `Pending` e eventos `FailedScheduling`.
6. Uso de taints `NoSchedule` e tolerations.

### Comandos uteis

```bash
# Criar o cluster (somente se ele ainda nao existir)
kind create cluster --config infra/kind/cluster.yaml

# Escolher o contexto do laboratorio
kubectl config use-context kind-kcna-cka-lab

# Inspecionar os nos
kubectl get nodes -o wide

# Inspecionar o Deployment e seus Pods
kubectl get deployment nginx-lab
kubectl get pods -l apps=nginx-lab -o wide

# Verificar eventos e problemas de agendamento
kubectl describe pod pod-pending

# Validar um manifesto sem persistir alteracoes
kubectl apply --dry-run=server -f modules/01-control-plane/manifests/deployment.yaml
```

> **Cuidado:** os exemplos `pod-pending.yaml` e `pod-sem-toleration.yaml` foram criados para exercicios de agendamento e podem ficar em `Pending` intencionalmente. O arquivo `worker02-taint.yaml` documenta um taint experimental; nao o aplique indiscriminadamente no Worker 02, que tambem hospeda as replicas do `nginx-lab`.

## Proximo modulo — Services, DNS e Networking

- Criar um Service do tipo `ClusterIP` para `nginx-lab`.
- Entender seletores, endpoints e DNS interno.
- Testar conectividade entre Pods e Services.
- Explorar NetworkPolicies em um CNI que ofereca suporte a enforcement.

## Boas praticas

- Validar manifestos antes de aplica-los.
- Conferir o contexto ativo de `kubectl` antes de modificar recursos.
- Inspecionar eventos e o estado real do cluster durante troubleshooting.
- Nunca versionar tokens, credenciais ou kubeconfigs locais.
- Registrar os exercicios em commits pequenos e descritivos.

## Referencias

- [Documentacao oficial do Kubernetes](https://kubernetes.io/docs/)
- [Kind](https://kind.sigs.k8s.io/)
- [CNCF](https://www.cncf.io/)


## Módulo 02 — Services, DNS e Networking

**Status:** Exercícios 13 a 19 concluídos.

### Conceitos praticados

- Criação de Services por manifestos YAML.
- Services `ClusterIP`, `NodePort` e `LoadBalancer`.
- Seleção de Pods por labels e selectors.
- Descoberta de endpoints com EndpointSlices.
- Resolução DNS pelo CoreDNS.
- Comunicação HTTP entre Pods.
- Comunicação entre namespaces usando FQDN.
- Acesso via NodePort em diferentes Workers.
- Diagnóstico de `EXTERNAL-IP: <pending>` no Kind.

### Recursos criados

| Recurso | Tipo | Finalidade |
|---|---|---|
| nginx-svc | ClusterIP | Comunicação interna |
| nginx-nodeport | NodePort | Acesso pela porta 30080 dos nós |
| nginx-loadbalancer | LoadBalancer | Simulação de balanceador externo |
| network-client | Pod | Testes de DNS e HTTP |
| financeiro | Namespace | Testes entre namespaces |
| financeiro-client | Pod | Cliente em outro namespace |

### Resultados observados

- CoreDNS resolveu `nginx-svc.default.svc.cluster.local`.
- O Service encaminhou conexões aos cinco Pods Nginx.
- O cliente no namespace `financeiro` acessou o Service no namespace `default`.
- O NodePort `30080` funcionou pelos IPs de ambos os Workers.
- O LoadBalancer permaneceu com IP externo pendente, pois não há controlador de LoadBalancer configurado no cluster Kind.

### Próxima etapa

**Exercício 20 — NetworkPolicy**

Estudar políticas de entrada e saída, isolamento de Pods e controle de tráfego entre namespaces.

Para testar o bloqueio real, será necessário utilizar um CNI que implemente NetworkPolicy, como Calico ou Cilium.