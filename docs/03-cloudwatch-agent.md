# 03 — Configuração do Amazon CloudWatch Agent

## Objetivo

Instalar e configurar o Amazon CloudWatch Agent no Raspberry Pi 5 para enviar métricas de infraestrutura para o Amazon CloudWatch.

Durante a implementação, foram testadas algumas configurações e ocorreram erros relacionados principalmente a credenciais e permissões. O objetivo deste documento é registrar o processo real, incluindo as tentativas que apresentaram problemas e a configuração que funcionou.

---

## 1. Ambiente

Servidor utilizado:

- Raspberry Pi 5
- Debian GNU/Linux 13 (trixie)
- Arquitetura: ARM64 / aarch64
- 4 CPUs
- Memória: aproximadamente 7,9 GiB
- Região AWS: `us-east-1`

---

## 2. Instalação do CloudWatch Agent

Foi instalado o Amazon CloudWatch Agent utilizando o pacote oficial para arquitetura ARM64.

Versão instalada:

```text
CWAgent/1.300072.0b1766

arm64
```

O agente foi instalado no diretório:

```text
/opt/aws/amazon-cloudwatch-agent/
```

---

## 3. Configuração do IAM

Foi criado um usuário IAM dedicado para o Raspberry Pi:

```text
raspberry-pi-cloudwatch
```

Também foi criado o grupo:

```text
raspberry-pi-cloudwatch-group
```

O usuário recebeu a política gerenciada pela AWS:

```text
CloudWatchAgentServerPolicy
```

Foi utilizada uma Access Key para permitir que o CloudWatch Agent enviasse métricas para a AWS.

### Princípio de segurança

Foi utilizado um usuário IAM separado para o agente em vez de utilizar as credenciais da conta Root.

As credenciais não devem ser armazenadas no GitHub.

---

## 4. Configuração das credenciais

As credenciais foram configuradas no Raspberry Pi utilizando o perfil:

```text
AmazonCloudWatchAgent
```

O arquivo utilizado foi:

```text
/home/rafael/.aws/credentials
```

O CloudWatch Agent foi configurado para utilizar esse perfil através do arquivo:

```text
/opt/aws/amazon-cloudwatch-agent/etc/common-config.toml
```

Configuração utilizada:

```toml
[credentials]
    shared_credential_profile = "AmazonCloudWatchAgent"
    shared_credential_file = "/home/rafael/.aws/credentials"
```

O acesso foi validado utilizando:

```bash
aws sts get-caller-identity \
  --profile AmazonCloudWatchAgent
```

O comando confirmou que as credenciais estavam funcionando.

---

## 5. Primeira configuração do CloudWatch Agent

Foi utilizado o assistente de configuração do CloudWatch Agent.

A configuração inicial foi feita para um ambiente `onPremise`.

Durante o assistente foram selecionadas opções básicas de monitoramento.

Não foram configurados:

- StatsD
- collectd
- logs de arquivos
- journald
- X-Ray

### Problema encontrado

Na primeira tentativa, o assistente tentou utilizar o AWS Systems Manager Parameter Store para armazenar a configuração.

A operação não funcionou porque o usuário IAM utilizado pelo Raspberry Pi não possuía as permissões necessárias para essa operação.

### Solução

Em vez de adicionar permissões adicionais apenas para utilizar o Parameter Store, foi utilizada uma configuração local no Raspberry Pi.

Essa decisão manteve a política IAM mais restrita e suficiente para o objetivo do projeto.

---

## 6. Teste manual de envio de métrica

Antes de validar o CloudWatch Agent, foi realizado um teste manual de envio de métrica utilizando a AWS CLI.

Comando utilizado:

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

O comando foi executado com sucesso.

Esse teste confirmou que o usuário IAM possuía permissão para enviar métricas para o CloudWatch.

A métrica `TestMetric` foi criada durante esse teste.

---

## 7. Configuração final

Após os testes iniciais, a configuração foi simplificada para coletar somente as métricas necessárias para o projeto.

Arquivo utilizado:

```text
/opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Configuração final:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_idle"],
        "metrics_collection_interval": 60,
        "resources": ["*"],
        "totalcpu": true
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["/", "/mnt/wdblue"]
      }
    }
  }
}
```

### Métricas coletadas

A configuração final coleta somente:

```text
CPU
└── cpu_usage_idle

Memória
└── mem_used_percent

Disco interno
└── /

Disco externo
└── /mnt/wdblue
```

O intervalo de coleta é de:

```text
60 segundos
```

---

## 8. Backup da configuração

Antes da alteração da configuração final, foi criado um backup:

```text
/opt/aws/amazon-cloudwatch-agent/bin/config.json.backup
```

Isso permitiu manter uma cópia da configuração anterior.

---

## 9. Validação da configuração

A configuração foi validada utilizando o próprio CloudWatch Agent.

O agente confirmou:

```text
Valid Json input schema.

Configuration validation first phase succeeded

Configuration validation second phase succeeded

Configuration validation succeeded
```

Após a validação, o agente foi iniciado utilizando:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m onPremise \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

---

## 10. Verificação do serviço

O serviço foi verificado e permaneceu em execução.

Estado esperado:

```text
Active: active (running)
```

O agente também foi configurado para iniciar automaticamente junto com o sistema.

---

## 11. Verificação dos logs do Agent

O log utilizado para verificar o funcionamento do agente foi:

```text
/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Comando utilizado:

```bash
sudo tail -n 30 \
  /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Entre as mensagens importantes encontradas:

```text
will use file based credentials provider
```

e:

```text
Everything is ready. Begin running and processing data.
```

Também foi identificado:

```text
publish with ForceFlushInterval: 1m0s
```

Isso indica um intervalo de flush de aproximadamente 1 minuto.

---

## 12. Mensagem NoCredentialProviders

Durante a inicialização apareceu uma mensagem semelhante a:

```text
NoCredentialProviders: no valid providers in chain
```

Também houve uma tentativa de consulta ao metadata service de uma instância EC2:

```text
EC2RoleRequestError: no EC2 instance role found
```

Essas mensagens não impediram o funcionamento do agente.

Logo depois, o log confirmou:

```text
will use file based credentials provider
```

e:

```text
Everything is ready. Begin running and processing data.
```

Portanto, o agente conseguiu utilizar corretamente o arquivo de credenciais configurado para o perfil:

```text
AmazonCloudWatchAgent
```

---

## 13. Validação no CloudWatch

Após a configuração final, foram verificadas as métricas no namespace:

```text
CWAgent
```

Foram observados novos dados para:

```text
cpu-total cpu_usage_idle

mem_used_percent

sda3 exfat /mnt/wdblue disk_used_percent

sdb2 ext4 / disk_used_percent
```

Na visualização do CloudWatch utilizada durante a validação, os pontos apareciam agregados em períodos de aproximadamente 5 minutos, embora o agente estivesse configurado para coletar métricas a cada 60 segundos.

### Resultado

A coleta das quatro categorias principais foi confirmada:

```text
CPU       → funcionando

MEM       → funcionando

/         → funcionando

WD Blue   → funcionando
```

---

## 14. Métricas antigas

Antes da redução da configuração existiam diversas métricas antigas no namespace `CWAgent`.

Entre elas estavam métricas relacionadas a:

- interfaces de rede;
- dispositivos;
- outros recursos do sistema.

Essas métricas permaneceram visíveis no CloudWatch mesmo depois da alteração da configuração.

Isso ocorre porque métricas que já foram publicadas não são simplesmente apagadas quando a coleta é interrompida.

A configuração atual do Agent, entretanto, não possui coleta de rede nem outras métricas adicionais.

---

## 15. Verificação de permissão ListMetrics

Foi realizado um teste utilizando:

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

porque o usuário:

```text
raspberry-pi-cloudwatch
```

não possui a permissão:

```text
cloudwatch:ListMetrics
```

Essa permissão não foi adicionada.

A decisão foi manter o usuário com as permissões necessárias para o funcionamento do agente, evitando adicionar permissões adicionais somente para consulta.

---

## 16. CloudWatch Logs

O CloudWatch Agent não foi configurado para enviar logs para o CloudWatch Logs.

A configuração final contém somente métricas.

O log do próprio agente permanece localmente no Raspberry Pi:

```text
/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Não foi criado um Log Group específico para o CloudWatch Agent.

---

## 17. Resultado final

A implementação resultou na seguinte arquitetura:

```text
                    AWS
                     │
             Amazon CloudWatch
                     ▲
                     │
              Métricas CWAgent
                     │
            CloudWatch Agent
                     │
             Raspberry Pi 5
                     │
        ┌────────────┼────────────┐
        │            │            │
       CPU          MEM          DISK
                                  │
                         ┌────────┴────────┐
                         │                 │
                         /            /mnt/wdblue
```

### Métricas finais

| Recurso       | Métrica             |
| ------------- | ------------------- |
| CPU           | `cpu_usage_idle`    |
| Memória       | `mem_used_percent`  |
| Disco interno | `disk_used_percent` |
| WD Blue       | `disk_used_percent` |

### Estado da implementação

**Implementação concluída e validada.**

O projeto demonstrou:

- configuração de IAM;
- utilização de credenciais AWS via profile;
- instalação do CloudWatch Agent em ARM64;
- configuração de monitoramento de servidor on-premises;
- validação de métricas;
- troubleshooting de permissões;
- redução de métricas desnecessárias;
- preocupação com custo e princípio de menor privilégio.

---

## Navegação

- [⬅️ Voltar ao README](../README.md)
- [➡️ Próximo: Métricas](04-metrics.md)
- [📋 Baseline do servidor](01-baseline.md)
- [🔐 Configuração do IAM](02-iam.md)
