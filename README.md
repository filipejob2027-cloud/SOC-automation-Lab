# 🛡️ SOC Automation & Incident Response Lab

## 📌 Project Overview
Este laboratório demonstra a implementação de um **SOC (Security Operations Center)** automatizado em ambiente local utilizando **Docker Compose**.

A infraestrutura integra a ingestão de telemetria de um endpoint Windows (via Sysmon + Wazuh Agent), análise defensiva no **Wazuh SIEM**, orquestração de resposta via **Shuffle SOAR** e gestão centralizada de incidentes no **TheHive 5**.

---

## 🏗️ Architecture & Deployment Strategy

Em vez de utilizar instâncias distribuídas pagas na nuvem, este projeto foi otimizado para ser implantado via **Docker Compose** localmente. Esta decisão de engenharia garante:
* **Custo Zero:** Eliminação de encargos com infraestruturas cloud mantendo as mesmas capacidades analíticas.
* **Portabilidade & Desempenho:** Orquestração leve de microserviços com consumo otimizado de RAM e CPU.
* **Reprodutibilidade:** Ambientes rápidos de implantar e replicar em qualquer máquina de testes.


[ Windows Endpoint (Sysmon) ] ───> [ Wazuh SIEM Stack ]
│
(Alert Trigger)
│
▼
[ TheHive 5 Case Mgmt ] <───── [ Shuffle SOAR ]


---

## 🛠️ Stack & Componentes

| Componente | Tecnologia | Função no Laboratório |
| :--- | :--- | :--- |
| **Endpoint Client** | Windows 11 Host + Sysmon | Telemetria detalhada de processos, rede e registo |
| **SIEM & XDR** | Wazuh Stack (Manager, Indexer, Dashboard) | Ingestão, correlação de logs e regras personalizadas |
| **Case Management** | TheHive 5 (Cassandra 4 + Elasticsearch 7) | Gestão e investigação de casos de segurança |
| **Orquestração (SOAR)**| Shuffle | Automação do fluxo de resposta e notificação de incidentes |

---

## 📂 Estrutura do Repositório

```text
.
├── wazuh-docker/       # Definições de orquestração do Wazuh SIEM
├── thehive-lab/        # Ficheiros Docker Compose e application.conf do TheHive
└── README.md           # Documentação principal do projeto
