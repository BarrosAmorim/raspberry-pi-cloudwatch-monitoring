# 05 — Troubleshooting

## Objetivo

Documentar os principais problemas encontrados durante a implementação do Amazon CloudWatch Agent no Raspberry Pi 5 e as ações realizadas para identificar e resolver cada situação.

O objetivo desta etapa foi praticar troubleshooting de infraestrutura envolvendo:

- AWS IAM;
- credenciais;
- CloudWatch Agent;
- AWS CLI;
- permissões;
- métricas;
- configuração de servidor Linux.

---

# 1. Falha ao utilizar o Parameter Store

## Problema

Durante a primeira configuração do CloudWatch Agent, o assistente tentou utilizar o AWS Systems Manager Parameter Store para armazenar a configuração.

A operação não foi concluída devido à ausência das permissões necessárias para essa utilização.

Fluxo observado:

```text
CloudWatch Agent
       │
       ▼
Parameter Store
       │
       ▼
    ❌ Falha
       │
       ▼
Permissões insuficientes
```

## Investigação

O usuário IAM utilizado pelo Raspberry Pi possuía a política:

```text
CloudWatchAgentServerPolicy
```

Essa política era suficiente para o objetivo principal do projeto, que era permitir o envio de métricas.

Em vez de adicionar novas permissões somente para utilizar o Parameter Store, foi analisada uma alternativa mais simples.

## Solução

Foi utilizada uma configuração local no Raspberry Pi.

O arquivo utilizado foi:

```text
/opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Dessa forma, não foi necessário ampliar as permissões do usuário IAM.

Fluxo final:

```text
CloudWatch Agent
       │
       ▼
Configuração local
       │
       ▼
Raspberry Pi 5
       │
       ▼
Amazon CloudWatch
```

## Aprendizado

Nem sempre a solução para um erro de permissão deve ser simplesmente adicionar mais permissões.

Antes de ampliar uma política IAM, é importante verificar se existe outra forma de realizar a mesma tarefa mantendo o princípio do menor privilégio.

---

# 2. Mensagem `NoCredentialProviders`

## Problema

Durante a inicialização do CloudWatch Agent foram observadas mensagens relacionadas à ausência de credenciais:

```text
NoCredentialProviders: no valid providers in chain
```

Também apareceu uma tentativa de consultar o metadata service de uma instância EC2:

```text
EC2RoleRequestError: no EC2 instance role found
```

Inicialmente essas mensagens poderiam indicar um problema de autenticação.

## Investigação

O Raspberry Pi não é uma instância EC2.

Portanto, não existe uma IAM Role de instância EC2 disponível através do metadata service.

O agente tentou diferentes formas de obter credenciais antes de utilizar o arquivo configurado para o Raspberry Pi.

A configuração utilizada foi:

```text
Perfil:
AmazonCloudWatchAgent
```

Arquivo:

```text
/home/rafael/.aws/credentials
```

E no `common-config.toml`:

```toml
[credentials]
    shared_credential_profile = "AmazonCloudWatchAgent"
    shared_credential_file = "/home/rafael/.aws/credentials"
```

## Validação

O funcionamento das credenciais foi confirmado anteriormente através do AWS CLI:

```bash
aws sts get-caller-identity \
  --profile AmazonCloudWatchAgent
```

O CloudWatch Agent posteriormente registrou:

```text
will use file based credentials provider
```

E depois:

```text
Everything is ready. Begin running and processing data.
```

## Resultado

A mensagem inicial não impediu o funcionamento do agente.

O agente conseguiu utilizar corretamente as credenciais armazenadas no arquivo local.

Fluxo final:

```text
CloudWatch Agent
       │
       ├── EC2 metadata ❌
       │
       └── arquivo de credenciais
                    │
                    ▼
          AmazonCloudWatchAgent
                    │
                    ▼
              AWS CloudWatch
                    │
                    ▼
                   ✅
```

## Aprendizado

Uma mensagem de erro durante a inicialização não significa necessariamente que o serviço inteiro falhou.

É necessário analisar as mensagens seguintes e verificar se o sistema conseguiu utilizar outro método válido de autenticação.

---

# 3. `AccessDenied` ao utilizar `ListMetrics`

## Problema

Durante a investigação das métricas existentes no namespace `CWAgent`, foi executado:

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

## Investigação

Foi identificado que o usuário IAM:

```text
raspberry-pi-cloudwatch
```

não possuía a permissão:

```text
cloudwatch:ListMetrics
```

A política utilizada pelo usuário era:

```text
CloudWatchAgentServerPolicy
```

## Decisão

A permissão `cloudwatch:ListMetrics` não foi adicionada.

O motivo foi que essa permissão não era necessária para o objetivo principal do agente.

O objetivo era:

```text
Raspberry Pi
     ↓
CloudWatch Agent
     ↓
Enviar métricas
     ↓
CloudWatch
```

O envio de métricas estava funcionando.

Portanto, adicionar uma permissão apenas para realizar consultas através do `ListMetrics` aumentaria o nível de acesso sem necessidade.

## Resultado

O erro foi compreendido e não foi necessário alterar a política IAM.

Isso manteve a configuração mais próxima do princípio de menor privilégio.

## Aprendizado

Um erro de `AccessDenied` deve ser analisado considerando:

1. Qual operação está sendo executada?
2. Qual permissão está faltando?
3. Essa permissão é realmente necessária?
4. Existe uma alternativa?
5. Adicionar a permissão aumenta o acesso sem necessidade?

---

# 4. Métricas antigas permanecendo no CloudWatch

## Problema

Depois de reduzir a configuração do CloudWatch Agent, algumas métricas antigas continuaram aparecendo no namespace:

```text
CWAgent
```

Entre elas estavam métricas relacionadas a:

- interfaces de rede;
- dispositivos;
- outros recursos do sistema.

Isso poderia gerar a dúvida:

> O CloudWatch Agent ainda está coletando essas métricas?

## Investigação

A configuração final do agente foi analisada.

Ela contém somente:

```text
CPU
MEM
/
 /mnt/wdblue
```

Não existe coleta de:

```text
Network
Docker
Interfaces
Temperatura
Outros dispositivos
```

na configuração final.

Também foram verificadas novas métricas e confirmados dados recentes somente para as métricas definidas na configuração final.

## Conclusão

As métricas antigas eram dados que já haviam sido publicados anteriormente.

Alterar a configuração do agente não significa que todo o histórico de métricas anteriormente publicado será imediatamente removido.

Portanto:

```text
Configuração antiga
       │
       ▼
Muitas métricas
       │
       ▼
Configuração alterada
       │
       ▼
Somente CPU/MEM/DISK
       │
       ▼
Métricas antigas ainda podem aparecer
```

Isso não significa que o agente atual continue coletando todas elas.

## Aprendizado

Durante um troubleshooting de monitoramento é importante diferenciar:

```text
Métrica existente
        ≠
Métrica sendo coletada atualmente
```

A existência de uma métrica no CloudWatch não é suficiente para concluir que ela ainda está sendo publicada pelo agente.

---

# 5. Validação do funcionamento após os problemas

Depois das correções e ajustes, foram realizadas novas verificações.

O agente apresentou:

```text
Everything is ready. Begin running and processing data.
```

As métricas esperadas apareceram no namespace:

```text
CWAgent
```

Foram confirmados dados para:

```text
CPU
└── cpu_usage_idle

MEM
└── mem_used_percent

DISCO INTERNO
└── / → disk_used_percent

DISCO EXTERNO
└── /mnt/wdblue → disk_used_percent
```

O serviço permaneceu em execução:

```text
Active: active (running)
```

---

# 6. Fluxo de troubleshooting utilizado

Durante o projeto, o processo de troubleshooting seguiu uma sequência semelhante a:

```text
Problema
   │
   ▼
Identificar o erro
   │
   ▼
Analisar logs/comandos
   │
   ▼
Identificar a causa provável
   │
   ▼
Testar a hipótese
   │
   ▼
Aplicar a menor alteração necessária
   │
   ▼
Validar novamente
   │
   ▼
Funcionamento confirmado
```

Esse processo foi utilizado principalmente nos problemas relacionados a:

- credenciais;
- permissões IAM;
- configuração do CloudWatch Agent;
- métricas;
- validação do envio para o CloudWatch.

---

# 7. Principais aprendizados

Este projeto permitiu praticar troubleshooting em um ambiente híbrido:

```text
Raspberry Pi 5
      │
      │ Linux
      │
      ▼
CloudWatch Agent
      │
      │ AWS CLI / IAM
      ▼
Amazon CloudWatch
```

Os principais aprendizados foram:

- interpretar mensagens de erro;
- verificar credenciais utilizando AWS CLI;
- entender o funcionamento de profiles;
- analisar permissões IAM;
- diferenciar erro de consulta de erro de funcionamento do agente;
- utilizar logs para validar o comportamento de um serviço;
- validar métricas após uma alteração;
- evitar adicionar permissões IAM desnecessárias;
- diferenciar dados históricos de dados atualmente coletados;
- realizar alterações de configuração e validar o resultado.

---

## Navegação

- [⬅️ Voltar ao README](../README.md)
- [⬅️ Métricas](04-metrics.md)
- [⬅️ CloudWatch Agent](03-cloudwatch-agent.md)
- [➡️ Próximo: Encerramento e aprendizados](06-encerramento.md)
