# 🚀 Raspberry Pi 5 — Monitoramento e Troubleshooting com AWS CloudWatch

Projeto prático de **Cloud / Infrastructure / DevOps**, utilizando um Raspberry Pi 5 como servidor Linux real e o **Amazon CloudWatch** para monitoramento de métricas de infraestrutura.

O projeto teve como objetivo praticar **AWS, Linux, IAM, AWS CLI, monitoramento e troubleshooting**, utilizando um ambiente físico real.

> ✅ **Projeto concluído e encerrado**

---

## 🎯 Objetivo

O objetivo foi configurar o Raspberry Pi 5 para enviar métricas básicas de infraestrutura para o **Amazon CloudWatch** utilizando o **CloudWatch Agent**.

Foram monitorados:

- 🖥️ CPU
- 🧠 Memória RAM
- 💾 Disco interno
- 💾 Disco externo

O projeto também teve como foco a prática de troubleshooting durante a instalação e configuração.

---

## 🏗️ Arquitetura

```text
                    INTERNET
                        │
                        │ Métricas
                        ▼
                ┌─────────────────┐
                │      AWS        │
                │                 │
                │   CloudWatch    │
                │                 │
                │    Namespace    │
                │     CWAgent     │
                └────────▲────────┘
                         │
                         │
                ┌────────┴────────┐
                │  Raspberry Pi 5 │
                │                 │
                │   Debian 13     │
                │     ARM64       │
                │                 │
                │ CloudWatch Agent│
                └─────────────────┘
```

---

## 🖥️ Ambiente

| Componente          | Informação                       |
| ------------------- | -------------------------------- |
| Hardware            | Raspberry Pi 5                   |
| Arquitetura         | ARM64 / aarch64                  |
| Sistema operacional | Debian GNU/Linux 13              |
| CPU                 | 4 cores                          |
| Memória             | ~7,9 GiB                         |
| AWS Region          | `us-east-1`                      |
| Monitoramento       | Amazon CloudWatch                |
| Agent               | CloudWatch Agent 1.300072.0b1766 |

---

# 📚 Documentação

A implementação foi documentada em etapas.

### 01 — Baseline

Levantamento inicial do ambiente antes da implantação do monitoramento.

**Status:** ✅ Concluído

👉 [📊 Ver documentação do Baseline](docs/01-baseline.md)

---

### 02 — IAM

Criação e configuração das permissões necessárias para permitir que o Raspberry Pi enviasse métricas para o CloudWatch.

**Status:** ✅ Concluído

👉 [🔐 Ver documentação do IAM](docs/02-iam.md)

---

### 03 — CloudWatch Agent

Instalação, configuração e validação do CloudWatch Agent no Raspberry Pi.

**Status:** ✅ Concluído

👉 [☁️ Ver documentação do CloudWatch Agent](docs/03-cloudwatch-agent.md)

---

### 04 — Métricas

Configuração e validação das métricas coletadas pelo CloudWatch Agent.

**Status:** ✅ Concluído

👉 [📈 Ver documentação das Métricas](docs/04-metrics.md)

---

### 05 — Troubleshooting

Registro dos principais problemas encontrados durante a implementação e do processo utilizado para investigá-los e solucioná-los.

**Status:** ✅ Concluído

👉 [🔧 Ver documentação de Troubleshooting](docs/05-troubleshooting.md)

---

### 06 — Encerramento e Aprendizados

Registro do resultado final, limitações, decisões relacionadas a custos, aprendizados e encerramento dos recursos utilizados no projeto.

**Status:** ✅ Concluído

👉 [📚 Ver documentação do Encerramento](docs/06-encerramento.md)

---

# 📊 Métricas monitoradas

A configuração final do CloudWatch Agent coletava:

| Métrica                           | Objetivo                           |
| --------------------------------- | ---------------------------------- |
| `cpu_usage_idle`                  | Acompanhar o uso da CPU            |
| `mem_used_percent`                | Acompanhar o uso da memória        |
| `disk_used_percent` `/`           | Acompanhar o armazenamento interno |
| `disk_used_percent` `/mnt/wdblue` | Acompanhar o armazenamento externo |

As métricas foram enviadas para o namespace:

```text
CWAgent
```

---

# 🔧 Troubleshooting

Durante a implementação foram encontrados problemas reais que exigiram investigação.

Entre eles:

- configuração do Parameter Store;
- problemas relacionados à obtenção de credenciais;
- tentativa de acesso ao metadata service de EC2;
- erro `AccessDenied` relacionado ao `cloudwatch:ListMetrics`;
- identificação de métricas históricas;
- validação da configuração do CloudWatch Agent;
- análise dos logs do agente.

O processo utilizado foi:

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
Teste
   ↓
Validação
   ↓
Documentação
```

---

# 💰 Decisão sobre custos

O projeto foi mantido propositalmente simples para evitar custos desnecessários.

Não foram implementados:

- CloudWatch Dashboard;
- CloudWatch Alarms;
- CloudWatch Logs para os logs do Raspberry Pi;
- monitoramento de temperatura;
- monitoramento de rede;
- monitoramento de Docker pelo CloudWatch Agent;
- Parameter Store;
- infraestrutura como código.

Após a validação do projeto, os recursos específicos utilizados para o monitoramento foram encerrados.

---

# 🧹 Encerramento

Após a conclusão dos testes, o monitoramento foi encerrado.

### AWS

Foram removidos:

- usuário IAM dedicado;
- grupo IAM dedicado;
- Access Key utilizada pelo Raspberry Pi.

### Raspberry Pi

Foram removidos:

- CloudWatch Agent;
- serviço do CloudWatch Agent;
- configurações do agente;
- logs do agente;
- profile `AmazonCloudWatchAgent`;
- credenciais relacionadas ao projeto.

O profile `default` da AWS CLI foi preservado.

As métricas históricas do namespace `CWAgent` permanecem no CloudWatch, mas o Raspberry Pi não envia mais novos dados.

---

# 📁 Estrutura do projeto

```text
raspberry-pi-cloudwatch-monitoring/
│
├── README.md
│
└── docs/
    ├── 01-baseline.md
    ├── 02-iam.md
    ├── 03-cloudwatch-agent.md
    ├── 04-metrics.md
    ├── 05-troubleshooting.md
    └── 06-encerramento.md
```

A estrutura foi mantida simples porque o projeto não utilizou scripts ou infraestrutura como código.

---

# 🛣️ O que foi praticado

Durante o projeto foram colocados em prática conhecimentos relacionados a:

- Linux
- AWS
- IAM
- AWS CLI
- CloudWatch
- CloudWatch Agent
- Monitoramento
- Observabilidade
- Troubleshooting
- Permissões
- Logs
- Infraestrutura

---

# 🎓 Principais aprendizados

O projeto permitiu praticar não apenas a configuração do CloudWatch, mas também o processo de troubleshooting.

Foi necessário:

- interpretar mensagens de erro;
- analisar logs;
- verificar permissões;
- testar credenciais;
- validar configurações;
- analisar métricas;
- diferenciar dados históricos de dados atuais;
- considerar custos;
- documentar problemas e soluções.

O principal aprendizado foi entender que **troubleshooting envolve investigação e validação**, e não apenas execução de comandos.

---

# 🏁 Resultado

O projeto atingiu seu objetivo principal:

> Utilizar um servidor Linux físico real, o Raspberry Pi 5, para implementar e validar o envio de métricas de infraestrutura para o Amazon CloudWatch.

Durante o período de funcionamento, foram coletadas métricas de:

```text
CPU
Memória
Disco /
Disco /mnt/wdblue
```

O projeto também proporcionou experiência prática com **AWS IAM, AWS CLI, CloudWatch Agent, Linux, permissões, logs e troubleshooting**.

> **Projeto concluído, documentado e encerrado.**

---

## 📚 Navegação

🏠 **README**

👉 [📊 Baseline](docs/01-baseline.md)

👉 [🔐 IAM](docs/02-iam.md)

👉 [☁️ CloudWatch Agent](docs/03-cloudwatch-agent.md)

👉 [📈 Métricas](docs/04-metrics.md)

👉 [🔧 Troubleshooting](docs/05-troubleshooting.md)

👉 [📚 Encerramento e Aprendizados](docs/06-encerramento.md)
