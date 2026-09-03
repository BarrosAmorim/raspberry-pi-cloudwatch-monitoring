# 04 — Métricas do Amazon CloudWatch

## Objetivo

Documentar as métricas coletadas pelo Amazon CloudWatch Agent no Raspberry Pi 5 e registrar como esses dados foram validados no Amazon CloudWatch.

A configuração final foi reduzida para coletar somente informações consideradas relevantes para o monitoramento básico do servidor:

- CPU
- Memória
- Disco interno
- Disco externo WD Blue

---

## 1. Namespace utilizado

As métricas enviadas pelo CloudWatch Agent foram armazenadas no namespace:

```text
CWAgent
```

Esse namespace foi utilizado para identificar as métricas provenientes do Amazon CloudWatch Agent.

---

## 2. Métricas monitoradas

A configuração final do agente ficou limitada a quatro categorias principais.

| Recurso                     | Métrica             |   Intervalo |
| --------------------------- | ------------------- | ----------: |
| CPU                         | `cpu_usage_idle`    | 60 segundos |
| Memória                     | `mem_used_percent`  | 60 segundos |
| Disco interno `/`           | `disk_used_percent` | 60 segundos |
| Disco externo `/mnt/wdblue` | `disk_used_percent` | 60 segundos |

---

## 3. CPU

A métrica utilizada para CPU foi:

```text
cpu_usage_idle
```

Essa métrica representa a porcentagem de tempo em que a CPU permaneceu ociosa.

Portanto, quanto maior o valor de `cpu_usage_idle`, menor tende a ser a utilização da CPU.

### Exemplo observado

Durante a validação foram observados valores próximos de:

```text
92% — 99% idle
```

Isso corresponde aproximadamente a:

```text
1% — 8% de utilização da CPU
```

O servidor estava com baixa utilização de CPU durante os períodos em que as métricas foram consultadas.

### Interpretação

```text
cpu_usage_idle = 95%

        ↓

aproximadamente 5% da CPU utilizada
```

Essa métrica pode ser útil para identificar situações de alta utilização do processador.

---

## 4. Memória

A métrica utilizada para memória foi:

```text
mem_used_percent
```

Ela representa a porcentagem de memória utilizada pelo sistema.

Durante a validação foram observados valores próximos de:

```text
16%
```

Isso indicava que aproximadamente 16% da memória RAM estava sendo utilizada naquele momento.

O Raspberry Pi possuía aproximadamente:

```text
7,9 GiB de RAM
```

### Interpretação

```text
mem_used_percent = 16%

        ↓

baixo nível de utilização de memória
```

Essa métrica pode ser utilizada para identificar crescimento anormal do consumo de RAM.

---

## 5. Disco interno

O disco interno do Raspberry Pi foi monitorado utilizando:

```text
disk_used_percent
```

O ponto de montagem monitorado foi:

```text
/
```

Durante a validação foi observado aproximadamente:

```text
50,6%
```

de utilização.

### Interpretação

```text
/ = 50,6%

        ↓

aproximadamente metade do armazenamento utilizado
```

O monitoramento do disco interno é importante porque o preenchimento excessivo pode causar problemas no sistema operacional e nos serviços executados no servidor.

---

## 6. Disco externo WD Blue

O disco externo utilizado pelo servidor também foi monitorado.

Ponto de montagem:

```text
/mnt/wdblue
```

Métrica:

```text
disk_used_percent
```

Durante a validação foi observado aproximadamente:

```text
91,2%
```

de utilização.

### Interpretação

```text
/mnt/wdblue = 91,2%

        ↓

disco próximo da capacidade máxima
```

Essa foi uma das métricas que mereceu maior atenção durante a análise do servidor.

Um armazenamento próximo da capacidade máxima pode causar problemas para serviços que dependem de espaço disponível para gravação.

---

## 7. Identificação dos discos no CloudWatch

Durante a consulta das métricas de disco foram observadas as seguintes informações:

```text
sda3  exfat  /mnt/wdblue
sdb2  ext4   /
```

Isso permitiu identificar qual série de métricas correspondia a cada armazenamento.

Representação:

```text
Raspberry Pi 5
      │
      ├── sdb2
      │    └── /
      │
      └── sda3
           └── /mnt/wdblue
```

---

## 8. Validação das métricas

Após a aplicação da configuração final, foram verificadas novas métricas no namespace:

```text
CWAgent
```

Foram confirmados dados para:

```text
cpu-total cpu_usage_idle

mem_used_percent

sda3 exfat /mnt/wdblue disk_used_percent

sdb2 ext4 / disk_used_percent
```

Isso confirmou que o CloudWatch Agent estava enviando dados para a AWS.

---

## 9. Intervalo de coleta

A configuração do agente foi definida com:

```text
metrics_collection_interval = 60
```

Portanto, a coleta das métricas foi configurada para ocorrer a cada:

```text
60 segundos
```

Na visualização utilizada durante a validação, os pontos apareciam agregados em períodos maiores, dependendo da visualização selecionada no CloudWatch.

Isso não significa que o agente estivesse configurado para coletar somente nesses períodos maiores.

---

## 10. Métrica de teste

Durante a implementação também foi criada manualmente uma métrica chamada:

```text
TestMetric
```

Ela foi enviada utilizando a AWS CLI através do comando:

```bash
aws cloudwatch put-metric-data \
  --namespace CWAgent \
  --metric-data '[
    {
      "MetricName": "TestMetric",
      "Value": 1,
      "Unit": "Count"
    }
  ]' \
  --profile AmazonCloudWatchAgent \
  --region us-east-1
```

O envio foi realizado com sucesso.

Esse teste teve como objetivo verificar se as credenciais utilizadas pelo Raspberry Pi possuíam permissão para publicar métricas no CloudWatch.

---

## 11. Métricas antigas

Durante a análise do namespace `CWAgent`, foram encontradas métricas relacionadas a configurações anteriores do agente.

Entre elas estavam informações relacionadas a:

- interfaces de rede;
- dispositivos;
- outros recursos do sistema.

Essas métricas não fazem parte da configuração final.

A configuração atual foi reduzida para:

```text
CPU
MEM
/
 /mnt/wdblue
```

Portanto, a existência de métricas antigas no namespace não significa que o agente atual continue coletando esses dados.

---

## 12. Permissão ListMetrics

Durante a investigação das métricas antigas foi realizado um teste com:

```bash
aws cloudwatch list-metrics \
  --namespace CWAgent \
  --dimensions Name=interface \
  --profile AmazonCloudWatchAgent \
  --region us-east-1
```

O comando retornou:

```text
AccessDenied
```

A causa foi a ausência da permissão:

```text
cloudwatch:ListMetrics
```

no usuário:

```text
raspberry-pi-cloudwatch
```

Essa permissão não foi adicionada.

A decisão foi manter a configuração IAM sem uma permissão adicional que não era necessária para o envio das métricas pelo CloudWatch Agent.

---

## 13. Resultado da análise

As quatro categorias definidas no projeto foram validadas:

```text
CPU
└── cpu_usage_idle
    └── funcionando

MEM
└── mem_used_percent
    └── funcionando

DISCO INTERNO
└── /
    └── funcionando

DISCO EXTERNO
└── /mnt/wdblue
    └── funcionando
```

Os dados demonstraram que o Raspberry Pi estava conseguindo enviar métricas de infraestrutura para o Amazon CloudWatch.

---

## 14. O que foi aprendido

Durante essa etapa foi possível praticar:

- identificação de métricas de infraestrutura;
- interpretação de utilização de CPU;
- interpretação de utilização de memória;
- monitoramento de armazenamento;
- identificação de pontos de montagem;
- diferenciação entre métricas atuais e métricas antigas;
- validação de dados enviados pelo CloudWatch Agent;
- análise de permissões IAM;
- troubleshooting de `AccessDenied`;
- utilização da AWS CLI para testes.

Mais importante que simplesmente visualizar os gráficos, foi entender a relação entre:

```text
Servidor
   ↓
CloudWatch Agent
   ↓
Métrica
   ↓
CloudWatch
   ↓
Análise
   ↓
Identificação de possível problema
```

---

## Navegação

- [⬅️ Voltar ao README](../README.md)
- [⬅️ CloudWatch Agent](03-cloudwatch-agent.md)
- [➡️ Próximo: Troubleshooting](05-troubleshooting.md)
- [📋 Baseline do servidor](01-baseline.md)
- [🔐 Configuração do IAM](02-iam.md)
