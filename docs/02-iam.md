# 🔐 02 — IAM

## 🎯 Objetivo

Criar uma identidade AWS exclusiva para o Raspberry Pi 5, permitindo que o servidor se autentique na AWS e posteriormente envie métricas para o Amazon CloudWatch.

O usuário pessoal utilizado para administrar a conta AWS não será utilizado diretamente pelo Raspberry Pi.

---

## 🧠 Por que utilizar um usuário separado?

O Raspberry Pi é um servidor externo à AWS e precisa de uma identidade própria para realizar a comunicação com os serviços AWS.

Em vez de utilizar as credenciais do usuário administrativo, foi criado um usuário exclusivo para o projeto.

Isso permite separar:

```text
Usuário pessoal
      │
      └── Administração da AWS


Raspberry Pi
      │
      └── Usuário exclusivo
              │
              └── CloudWatch
```

Essa separação reduz o impacto caso as credenciais utilizadas pelo servidor sejam comprometidas.

---

## 👤 Usuário IAM

Foi criado o usuário:

```text
raspberry-pi-cloudwatch
```

O usuário foi criado sem acesso ao AWS Management Console, pois sua finalidade é permitir a autenticação do Raspberry Pi para comunicação programática com a AWS.

---

## 👥 Grupo IAM

Para organizar as permissões, foi criado o grupo:

```text
raspberry-pi-cloudwatch-group
```

O usuário `raspberry-pi-cloudwatch` foi associado a esse grupo.

A estrutura ficou:

```text
raspberry-pi-cloudwatch-group
            │
            └── raspberry-pi-cloudwatch
```

---

## 🔑 Política IAM

Foi associada ao grupo a política gerenciada pela AWS:

```text
CloudWatchAgentServerPolicy
```

Essa política fornece as permissões necessárias para o funcionamento do CloudWatch Agent, incluindo a possibilidade de enviar métricas para o CloudWatch.

### Estrutura final

```text
IAM
│
└── raspberry-pi-cloudwatch-group
        │
        └── CloudWatchAgentServerPolicy
        │
        └── raspberry-pi-cloudwatch
```

---

## 🏷️ Identificação

Foi adicionada uma etiqueta ao usuário/grupo para facilitar a identificação do projeto.

```text
Chave: Project
Valor: raspberry-pi-cloudwatch
```

A Access Key também recebeu uma descrição:

```text
CloudWatch Agent - Raspberry Pi 5 - Monitoramento
```

---

## 🔑 Access Key

Foi criada uma Access Key exclusiva para o usuário:

```text
raspberry-pi-cloudwatch
```

A chave será utilizada pelo Raspberry Pi para autenticação programática na AWS.

### ⚠️ Segurança

As credenciais não devem ser armazenadas no código do projeto.

Não devem ser adicionadas ao:

- GitHub
- README
- arquivos `.md`
- scripts
- arquivos de configuração versionados

A Access Key e a Secret Access Key foram configuradas diretamente no Raspberry Pi.

---

## 💻 AWS CLI

A AWS CLI não estava instalada inicialmente no Raspberry Pi.

Foi realizada a instalação através do gerenciador de pacotes do Debian.

Após a instalação, foi realizada a validação:

```bash
aws --version
```

Resultado:

```text
aws-cli/2.23.6 Python/3.13.5
Linux 6.18.39+rpt-rpi-2712
source/aarch64.debian.13
```

Isso confirmou que a AWS CLI estava instalada e funcionando na arquitetura ARM64 do Raspberry Pi.

---

## ⚙️ Configuração

A AWS CLI foi configurada através do comando:

```bash
aws configure
```

Foram configurados:

```text
Access Key ID
Secret Access Key
Default region: us-east-1
Output format: json
```

As credenciais não são registradas neste documento.

---

## 🧪 Teste de autenticação

Para validar a autenticação, foi utilizado:

```bash
aws sts get-caller-identity
```

O comando retornou uma identidade associada ao usuário:

```text
arn:aws:iam::XXXXXXXXXXXX:user/raspberry-pi-cloudwatch
```

O teste confirmou que o Raspberry Pi está utilizando corretamente o usuário IAM criado especificamente para o projeto.

---

## ✅ Resultado

A etapa de configuração do IAM foi concluída.

```text
☑ Usuário IAM criado
☑ Grupo IAM criado
☑ Política CloudWatchAgentServerPolicy associada
☑ Access Key criada
☑ AWS CLI instalada
☑ AWS CLI configurada
☑ Autenticação validada
```

O Raspberry Pi agora está preparado para a próxima etapa do projeto.

---

## ➡️ Próxima etapa

A próxima etapa será instalar e configurar o **Amazon CloudWatch Agent** no Raspberry Pi.

O objetivo será começar a coletar métricas do servidor e enviá-las para o CloudWatch.

---

### 📚 Navegação

[⬅️ Anterior: Baseline](01-baseline.md)

[🏠 Voltar ao README](../README.md)

[➡️ Próxima: CloudWatch Agent](03-cloudwatch-agent.md)
