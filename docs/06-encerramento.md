# 06 — Encerramento e Aprendizados

## Objetivo

Registrar o encerramento do projeto de monitoramento do Raspberry Pi 5 utilizando o Amazon CloudWatch, apresentando os resultados obtidos, as limitações encontradas e os principais conhecimentos adquiridos durante a implementação.

---

## 1. Visão geral do projeto

O projeto teve como objetivo monitorar um Raspberry Pi 5 utilizando o Amazon CloudWatch Agent e enviar métricas de infraestrutura para o Amazon CloudWatch.

A arquitetura utilizada foi:

```text id="m1x8fz"
Raspberry Pi 5
      │
      ▼
CloudWatch Agent
      │
      ▼
Amazon CloudWatch
      │
      ▼
Métricas de infraestrutura
```

O servidor utilizado foi um Raspberry Pi 5 executando Debian GNU/Linux 13 (trixie), com arquitetura ARM64.

---

## 2. O que foi implementado

Durante o projeto foram realizadas as seguintes atividades:

### Infraestrutura

- levantamento das características do Raspberry Pi;
- identificação de CPU, memória e armazenamento;
- identificação dos pontos de montagem dos discos;
- definição das métricas relevantes para monitoramento.

### AWS IAM

- criação de um usuário IAM dedicado;
- criação de um grupo IAM;
- associação da política `CloudWatchAgentServerPolicy`;
- criação e utilização de Access Key;
- configuração de um profile específico para o CloudWatch Agent.

### CloudWatch Agent

- instalação do CloudWatch Agent em arquitetura ARM64;
- configuração para ambiente `onPremise`;
- configuração de credenciais utilizando profile;
- criação de configuração local;
- validação da configuração;
- inicialização e execução do serviço.

### Monitoramento

Foram configuradas quatro categorias principais:

```text id="1n6b2z"
CPU
└── cpu_usage_idle

Memória
└── mem_used_percent

Disco interno
└── / → disk_used_percent

Disco externo
└── /mnt/wdblue → disk_used_percent
```

---

## 3. Troubleshooting realizado

Durante a implementação ocorreram problemas reais que precisaram ser investigados.

Entre eles:

### Parameter Store

A primeira configuração tentou utilizar o Parameter Store, mas a operação não foi concluída devido às permissões disponíveis.

A solução foi utilizar uma configuração local no Raspberry Pi.

### Credenciais

Durante a inicialização apareceram mensagens relacionadas ao `NoCredentialProviders` e à tentativa de utilizar o metadata service de uma instância EC2.

A investigação mostrou que o agente posteriormente utilizou corretamente o arquivo de credenciais configurado.

### `AccessDenied`

Uma consulta utilizando `cloudwatch:ListMetrics` retornou `AccessDenied`.

A permissão não foi adicionada porque não era necessária para o funcionamento do agente.

### Métricas antigas

Métricas de configurações anteriores continuaram aparecendo no namespace `CWAgent`.

Foi necessário diferenciar métricas históricas de métricas que estavam sendo coletadas pela configuração atual.

---

## 4. O que não foi implementado

Nem todos os recursos disponíveis no Amazon CloudWatch foram utilizados.

Não foram implementados:

- CloudWatch Dashboard;
- CloudWatch Alarms;
- envio de logs do Raspberry Pi para o CloudWatch Logs;
- monitoramento de temperatura;
- monitoramento de rede;
- monitoramento de Docker através do CloudWatch Agent;
- armazenamento da configuração no Parameter Store.

Esses itens ficaram fora do escopo final do projeto.

---

## 5. Decisão relacionada a custos

Durante o projeto também foi analisada a questão de custos do Amazon CloudWatch.

Como o objetivo principal era aprender monitoramento e troubleshooting, foi decidido não ampliar o projeto com recursos adicionais que poderiam gerar custos desnecessários.

Por esse motivo, a configuração final foi mantida simples:

```text id="5f9g2c"
CPU
MEM
DISK
```

Essa decisão também reduziu a quantidade de métricas enviadas para a AWS.

O projeto demonstrou que uma solução de monitoramento deve considerar não apenas os aspectos técnicos, mas também o custo de operação.

---

## 6. O que foi aprendido

O projeto permitiu colocar em prática conceitos estudados em AWS, Linux e infraestrutura.

### AWS

- IAM;
- usuários e grupos;
- políticas;
- Access Keys;
- AWS CLI;
- Amazon CloudWatch;
- CloudWatch Agent;
- namespaces;
- métricas;
- permissões.

### Linux

- identificação de hardware e sistema operacional;
- armazenamento e pontos de montagem;
- serviços;
- logs;
- arquivos de configuração;
- permissões;
- execução de comandos administrativos.

### Troubleshooting

Foi praticado um processo baseado em:

```text id="jq7k2h"
Problema
   ↓
Investigação
   ↓
Identificação da causa
   ↓
Alteração
   ↓
Teste
   ↓
Validação
```

Esse processo foi mais importante do que simplesmente conseguir colocar o agente para funcionar.

---

## 7. Principal aprendizado

Um dos principais aprendizados do projeto foi entender que implementar uma solução não significa apenas seguir comandos.

Foi necessário:

- interpretar mensagens de erro;
- verificar permissões;
- testar credenciais;
- analisar logs;
- entender o comportamento do CloudWatch Agent;
- validar os dados enviados;
- reduzir a configuração;
- considerar custos;
- escolher uma solução adequada ao objetivo.

Isso ajudou a transformar conceitos teóricos de AWS e Linux em uma experiência prática.

---

## 8. Limitações do projeto

O projeto foi desenvolvido em um ambiente doméstico utilizando um Raspberry Pi 5.

Por isso, ele não representa uma infraestrutura empresarial completa.

Não foram utilizados, por exemplo:

- múltiplos servidores;
- alta disponibilidade;
- balanceadores;
- Auto Scaling;
- Kubernetes;
- infraestrutura como código;
- pipelines de CI/CD;
- ambientes separados de desenvolvimento e produção.

O objetivo não era reproduzir uma infraestrutura corporativa completa, mas praticar monitoramento, AWS, Linux e troubleshooting em um ambiente real.

---

## 9. Possíveis evoluções

Caso o projeto seja retomado futuramente, algumas possibilidades seriam:

- criação de alarmes;
- monitoramento de temperatura;
- monitoramento de rede;
- monitoramento dos containers Docker;
- criação de dashboards;
- integração com ferramentas de monitoramento locais;
- automação da configuração;
- utilização de infraestrutura como código;
- integração com um pipeline CI/CD.

Essas funcionalidades não fazem parte da implementação atual.

---

## 10. Resultado final

Ao final do projeto, foi possível implementar e validar o monitoramento básico do Raspberry Pi 5 utilizando o Amazon CloudWatch.

Resultado:

```text id="s9q6xk"
┌──────────────────────┐
│    Raspberry Pi 5    │
│                      │
│  CPU                 │
│  Memória             │
│  Disco /             │
│  Disco /mnt/wdblue   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  CloudWatch Agent    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Amazon CloudWatch   │
│                      │
│  CWAgent             │
└──────────────────────┘
```

### Estado final

**Projeto concluído e validado.**

A implementação atingiu o objetivo principal de utilizar um servidor Linux real para enviar métricas de infraestrutura para a AWS e praticar troubleshooting durante o processo.

---

## 11. Conclusão

O projeto serviu como uma experiência prática de integração entre Linux, AWS IAM, AWS CLI e Amazon CloudWatch.

Mais do que simplesmente configurar uma ferramenta, foi possível passar pelo ciclo completo:

```text id="r2o1n4"
Planejamento
     ↓
Configuração
     ↓
Erro
     ↓
Investigação
     ↓
Correção
     ↓
Validação
     ↓
Documentação
```

A documentação das etapas e dos problemas encontrados também faz parte do resultado do projeto.

O objetivo foi manter o registro fiel ao que foi realizado, incluindo limitações e decisões tomadas durante a implementação.

---

## Navegação

- [⬅️ Voltar ao README](../README.md)
- [⬅️ Troubleshooting](05-troubleshooting.md)
- [⬅️ Métricas](04-metrics.md)
- [⬅️ CloudWatch Agent](03-cloudwatch-agent.md)
- [🏠 Início do projeto](../README.md)
