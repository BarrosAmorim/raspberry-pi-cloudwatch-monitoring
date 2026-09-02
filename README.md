# 🚀 Raspberry Pi 5 — Monitoramento e Troubleshooting com AWS CloudWatch

Projeto prático de **Cloud / Infrastructure / DevOps**, utilizando um Raspberry Pi 5 como servidor real e o **Amazon CloudWatch** para monitoramento da infraestrutura.

O projeto tem como objetivo acompanhar a saúde do servidor, identificar problemas e praticar processos de **monitoramento, troubleshooting e observabilidade** utilizando serviços da AWS.

> 🚧 **Projeto em desenvolvimento**

---

## 🎯 Objetivo

Construir uma solução de monitoramento para um Raspberry Pi 5, permitindo acompanhar remotamente:

- 🖥️ CPU
- 🧠 Memória RAM
- 💾 Armazenamento
- 🌡️ Temperatura
- 🌐 Rede
- 🐳 Containers Docker
- ⚠️ Disponibilidade dos serviços

As métricas serão enviadas para o **Amazon CloudWatch**, onde posteriormente serão utilizados dashboards e alarmes.

---

## 🏗️ Arquitetura

```text
                    INTERNET
                        │
                        │
                ┌───────▼────────┐
                │      AWS       │
                │                │
                │  CloudWatch    │
                │                │
                │  📊 Métricas   │
                │  📈 Dashboard  │
                │  🚨 Alarmes    │
                └───────▲────────┘
                        │
                        │ Métricas
                        │
                ┌───────┴────────┐
                │  Raspberry Pi 5│
                │                │
                │   Debian 13    │
                │     ARM64      │
                │                │
                │    Docker      │
                │                │
                │ ┌────────────┐ │
                │ │ Jellyfin   │ │
                │ │ Nginx      │ │
                │ │ qBittorrent│ │
                │ │ Navidrome  │ │
                │ │ Cloudflare │ │
                │ └────────────┘ │
                └────────────────┘
```

---

## 🖥️ Ambiente

| Componente          | Informação          |
| ------------------- | ------------------- |
| Hardware            | Raspberry Pi 5      |
| Arquitetura         | ARM64 / aarch64     |
| Sistema operacional | Debian GNU/Linux 13 |
| Docker              | 29.7.2              |
| AWS Region          | `us-east-1`         |
| Monitoramento       | Amazon CloudWatch   |

---

# 📚 Documentação

A implementação está sendo realizada por etapas.

### 01 — Baseline

Levantamento inicial do servidor antes da implantação do monitoramento.

**Status:** ✅ Concluído

👉 [📊 Ver documentação do Baseline](docs/01-baseline.md)

---

### 02 — IAM

Configuração das permissões necessárias para que o Raspberry Pi possa enviar métricas para o CloudWatch.

**Status:** ⬜ A fazer

👉 [🔐 Ver documentação do IAM](docs/02-iam.md)

---

### 03 — CloudWatch Agent

Instalação e configuração do CloudWatch Agent no Raspberry Pi.

**Status:** ⬜ A fazer

👉 [☁️ Ver documentação do CloudWatch Agent](docs/03-cloudwatch-agent.md)

---

### 04 — Métricas

Configuração das métricas que serão coletadas e enviadas para a AWS.

**Status:** ⬜ A fazer

👉 [📈 Ver documentação das Métricas](docs/04-metrics.md)

---

### 05 — Dashboard

Criação de um dashboard no CloudWatch para visualizar o estado do servidor.

**Status:** ⬜ A fazer

👉 [📊 Ver documentação do Dashboard](docs/05-dashboard.md)

---

### 06 — Alarmes

Configuração de alarmes para identificar situações anormais.

**Status:** ⬜ A fazer

👉 [🚨 Ver documentação dos Alarmes](docs/06-alarms.md)

---

### 07 — Troubleshooting

Documentação de problemas encontrados durante o projeto e os processos utilizados para investigá-los e solucioná-los.

**Status:** ⬜ A fazer

👉 [🔧 Ver documentação de Troubleshooting](docs/07-troubleshooting.md)

---

# 📊 Métricas planejadas

| Métrica     | Objetivo                                      |
| ----------- | --------------------------------------------- |
| CPU         | Identificar utilização elevada do processador |
| RAM         | Identificar consumo excessivo de memória      |
| Disco       | Acompanhar espaço disponível                  |
| Temperatura | Monitorar temperatura do Raspberry Pi         |
| Rede        | Acompanhar tráfego de rede                    |
| Docker      | Verificar estado dos containers               |

---

# 🔧 Troubleshooting

O projeto também será utilizado para praticar troubleshooting em situações reais.

Alguns cenários planejados:

- 🐳 Container Docker parado
- 🔥 CPU com utilização elevada
- 💾 Disco próximo da capacidade máxima
- 🌐 Problema de conectividade
- ❌ Serviço indisponível
- 🐳 Investigação de consumo dos containers

Cada cenário será documentado seguindo o fluxo:

```text
Problema
   ↓
Coleta de informações
   ↓
Investigação
   ↓
Identificação da causa
   ↓
Correção
   ↓
Validação
   ↓
Documentação
```

---

# 📁 Estrutura do projeto

```text
raspberry-pi-cloudwatch-monitoring/
│
├── README.md
│
├── docs/
│   ├── 01-baseline.md
│   ├── 02-iam.md
│   ├── 03-cloudwatch-agent.md
│   ├── 04-metrics.md
│   ├── 05-dashboard.md
│   ├── 06-alarms.md
│   └── 07-troubleshooting.md
│
├── scripts/
│   ├── system-info.sh
│   ├── docker-health.sh
│   └── monitoring.sh
│
├── cloudwatch/
│   └── agent-config.json
│
├── troubleshooting/
│   ├── README.md
│   ├── 01-docker-container-down.md
│   ├── 02-high-cpu.md
│   ├── 03-disk-usage.md
│   ├── 04-network-problem.md
│   └── 05-service-unavailable.md
│
└── screenshots/
    ├── 01-baseline/
    ├── 02-cloudwatch/
    ├── 03-dashboard/
    └── 04-alarms/
```

---

# 🛣️ Roadmap

- [x] Levantamento do ambiente
- [x] Coleta do baseline
- [ ] Configuração do IAM
- [ ] Instalação do CloudWatch Agent
- [ ] Configuração das métricas
- [ ] Envio das métricas para o CloudWatch
- [ ] Criação do Dashboard
- [ ] Criação dos Alarmes
- [ ] Testes de troubleshooting
- [ ] Documentação dos incidentes
- [ ] Revisão dos custos AWS

---

# 📚 Principais conhecimentos praticados

- Linux
- Docker
- AWS
- IAM
- CloudWatch
- Monitoramento
- Observabilidade
- Troubleshooting
- Infraestrutura
- Redes
- Análise de problemas

---

## 👨‍💻 Sobre o projeto

Este projeto está sendo desenvolvido como um laboratório prático para consolidar conhecimentos em **Cloud, Infrastructure e DevOps**.

O diferencial do projeto é utilizar um **servidor físico real (Raspberry Pi 5)** como ambiente de infraestrutura, realizando monitoramento, análise de métricas e troubleshooting.

Toda a implementação será documentada conforme o projeto evolui, incluindo decisões técnicas, problemas encontrados, comandos utilizados, resultados e evidências.

---

### 📚 Navegação

🏠 **README**

👉 [📊 Baseline](docs/01-baseline.md)

👉 [🔐 IAM](docs/02-iam.md)

👉 [☁️ CloudWatch Agent](docs/03-cloudwatch-agent.md)

👉 [📈 Métricas](docs/04-metrics.md)

👉 [📊 Dashboard](docs/05-dashboard.md)

👉 [🚨 Alarmes](docs/06-alarms.md)

👉 [🔧 Troubleshooting](docs/07-troubleshooting.md)
