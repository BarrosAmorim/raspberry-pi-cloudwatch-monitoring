# 01 — Baseline do Raspberry Pi 5

## Objetivo

Registrar o estado inicial do Raspberry Pi 5 antes da implantação do monitoramento com AWS CloudWatch.

O baseline será utilizado posteriormente para comparação durante os testes de monitoramento e troubleshooting.

---

## Sistema operacional

```text
Linux rasp 6.18.39+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.18.39-1+rpt1 (2026-07-29) aarch64 GNU/Linux

PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
VERSION_ID="13"
DEBIAN_VERSION_FULL=13.6
```

**Arquitetura:** ARM64 (aarch64)

---

## CPU

```text
CPU: ARM Cortex-A76
Cores: 4
Frequência máxima: 2400 MHz
Frequência mínima: 1500 MHz
```

---

## Memória

```text
Memória total: 7.9 GiB
Memória utilizada: 1.5 GiB
Memória disponível: 6.4 GiB
Swap total: 2.0 GiB
Swap utilizada: 0 B
```

---

## Armazenamento

```text
/                    469G total | 228G usado | 223G disponível | 51%
/boot/firmware       510M total | 67M usado  | 444M disponível | 14%
/var/log             224M total | 15M usado  | 192M disponível | 7%
/mnt/wdblue          931G total | 849G usado | 83G disponível  | 92%
```

### Observação

O armazenamento montado em `/mnt/wdblue` apresenta utilização de aproximadamente **92%**.

Esse ponto será acompanhado durante o projeto e poderá ser utilizado posteriormente como cenário de troubleshooting.

**Nenhum arquivo foi removido ou alterado durante o levantamento do baseline.**

---

## Temperatura

```text
58.2°C
```

A temperatura registrada durante o levantamento inicial não apresentou, naquele momento, indicação de situação crítica.

---

## Uptime e carga

```text
Uptime: aproximadamente 2 dias
Load average: 0.00, 0.03, 0.01
```

O sistema apresentava carga muito baixa no momento da coleta.

---

## Docker

```text
Docker Engine: 29.7.2
API: 1.55
OS/Architecture: linux/arm64
containerd: 2.3.4
runc: 1.4.3
```

### Containers em execução

```text
jellyfin
qbittorrent
arcane
nginx-docker
cloudflared-navidrome
cloudflared-jellyfin
cloudflared-nccasa
glances
navidrome-navidrome-1
```

Os containers estavam em execução no momento da coleta.

O container `jellyfin` apresentava status `healthy`.

---

## Docker Stats

Durante a coleta, os principais consumos de CPU estavam baixos.

Exemplo:

```text
jellyfin                CPU: 3.07%
qbittorrent             CPU: 0.01%
arcane                  CPU: 0.05%
nginx-docker            CPU: 0.00%
cloudflared-navidrome   CPU: 0.11%
cloudflared-jellyfin    CPU: 0.10%
cloudflared-nccasa      CPU: 0.13%
glances                 CPU: 0.35%
navidrome               CPU: 0.01%
```

### Observação

O comando `docker stats --no-stream` apresentou `0B / 0B` na coluna de memória dos containers.

Esse comportamento será investigado posteriormente como parte do troubleshooting do projeto.

---

## Estado inicial

No momento da coleta, o servidor apresentava:

- CPU com baixa utilização
- Memória disponível suficiente
- Swap sem utilização
- Carga do sistema muito baixa
- Temperatura de 58.2°C
- Containers Docker em execução
- Jellyfin com status saudável
- `/mnt/wdblue` com 92% de utilização
- Métricas de memória do Docker necessitando investigação

Este documento representa o **estado inicial (baseline)** do ambiente antes da implantação do monitoramento com AWS CloudWatch.

---

## Próxima etapa

Configurar as permissões necessárias na AWS e preparar o Raspberry Pi 5 para enviar métricas ao **Amazon CloudWatch**.

## 🔗 Navegação

- [⬅️ README](../README.md)
- [➡️ Próximo: IAM](./02-iam.md)
- [📋 Encerramento do projeto](./06-encerramento.md)
