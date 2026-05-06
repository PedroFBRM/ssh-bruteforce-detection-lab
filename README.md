# SSH Brute Force Detection Lab

> Blue Team lab focused on detecting and analyzing failed SSH login attempts using Linux authentication logs.

---

## Objective

Simulate repeated failed SSH authentication attempts in an isolated lab environment and analyze the generated logs from a Blue Team perspective, identifying indicators of compromise and applying defensive recommendations.

---

## Environment

| Component   | Details                  |
| ----------- | ------------------------ |
| Attacker VM | Kali Linux               |
| Target VM   | Debian 13                |
| Network     | NAT Network (VirtualBox) |
| Attacker IP | 10.0.2.5                 |
| Target IP   | 10.0.2.15                |
| Log source  | /var/log/auth.log        |
| SSH port    | 22                       |

---

## Steps Performed

### 1. SSH Service Validation

Confirmed that the SSH service was active and running on the target machine.

```bash
sudo systemctl status ssh --no-pager
```

Result: `active (running)` — OpenBSD Secure Shell server listening on port 22.

---

### 2. rsyslog Validation

Confirmed that the system logging service was active, ensuring authentication events would be recorded in `/var/log/auth.log`.

```bash
sudo systemctl status rsyslog --no-pager
```

Result: `active (running)` — rsyslog v8.2504.0 running and collecting system logs.

---

### 3. Target IP Identification

Identified the IP address of the Debian target machine.

```bash
ip a
```

**Target IP:** `10.0.2.15/24` — interface `enp0s3`

---

### 4. Connectivity Test

Verified network connectivity between the Kali attacker machine and the Debian target.

```bash
ping -c 4 10.0.2.15
```

**Result:** 4 packets transmitted, 4 received, 0% packet loss. Average RTT: 0.631ms.

---

### 5. SSH Port Scan

Confirmed that port 22 was open and accessible on the target machine.

```bash
nmap -p 22 10.0.2.15
```

**Result:** `22/tcp open ssh` — Nmap 7.95, host is up (0.00059s latency).

---

### 6. Failed Authentication Attempts

Generated failed SSH login attempts from the Kali machine targeting the Debian server.

#### Invalid user attempt

```bash
ssh fakeuser@10.0.2.15
```

Three password attempts were made with incorrect credentials. Result: `Permission denied (publickey,password)`.

#### Valid user with wrong password

```bash
ssh analyst@10.0.2.15
```

Three password attempts were made with incorrect credentials. Result: `Permission denied (publickey,password)`.

---

### 7. Log Analysis

Filtered authentication logs to identify failed login events.

```bash
sudo grep -E "Failed password|Invalid user" /var/log/auth.log
```

Events observed between `21:34` and `21:37` on 2026-05-05:

- `Invalid user fakeuser from 10.0.2.5 port 46916`
- `Failed password for invalid user fakeuser from 10.0.2.5 port 46916 ssh2` (3 occurrences)
- `Failed password for analyst from 10.0.2.5 port 54394 ssh2` (3 occurrences)

---

### 8. Event Count

Counted the total number of failed authentication events by type.

```bash
sudo grep "Failed password" /var/log/auth.log | wc -l
sudo grep "Invalid user" /var/log/auth.log | wc -l
```

| Event type      | Count |
| --------------- | ----- |
| Failed password | 9     |
| Invalid user    | 3     |

---

### 9. Events Filtered by Source IP

Filtered all authentication events originating from the attacker IP.

```bash
sudo grep "10.0.2.5" /var/log/auth.log
sudo grep "10.0.2.5" /var/log/auth.log | wc -l
```

Additional events observed beyond direct failures:

- `pam_unix(sshd:auth): authentication failure` — PAM-level auth failure
- `Connection closed by invalid user fakeuser` — connection terminated after max attempts
- `PAM 2 more authentication failures` — PAM reporting accumulated failures
- `Connection closed by authenticating user analyst` — connection terminated after max attempts

**Total events from 10.0.2.5:** `15`

---

## Analysis

The authentication logs revealed a clear pattern of repeated failed login attempts originating from a single IP address (`10.0.2.5`) targeting the SSH service on port 22.

Two distinct attack patterns were observed:

- Attempts against a **non-existent user** (`fakeuser`), generating `Invalid user` events
- Attempts against a **valid user** (`analyst`) with incorrect passwords, generating `Failed password` events

Beyond the direct failure events, the logs also recorded PAM-level authentication failures and connection terminations after reaching the maximum number of attempts — behavior consistent with **brute force** or **password guessing** activity.

The concentration of 15 authentication-related events from a single source IP within a short time window (approximately 7 minutes) reinforces this assessment.

No successful authentication was observed during the session.

---

## Key Indicators

| Indicator                   | Value                       |
| --------------------------- | --------------------------- |
| Source IP                   | 10.0.2.5                    |
| Target port                 | 22 (SSH)                    |
| Targeted users              | fakeuser, analyst           |
| Failed password events      | 9                           |
| Invalid user events         | 3                           |
| Total events from source IP | 15                          |
| Time window                 | 21:34 to 21:41 (2026-05-05) |
| Successful logins           | 0                           |
| Log source                  | /var/log/auth.log           |

---

## Defensive Recommendations

Based on the findings, the following controls are recommended:

- Disable password-based SSH authentication and enforce public key authentication
- Restrict SSH access by source IP using firewall rules
- Deploy `fail2ban` to automatically block IPs after repeated failures
- Monitor `/var/log/auth.log` continuously for authentication anomalies
- Avoid exposing port 22 directly to untrusted networks
- Consider changing the default SSH port to reduce automated scanning noise

---

## Conclusion

This lab demonstrated how repeated failed SSH login attempts generate identifiable patterns in Linux authentication logs. By analyzing `/var/log/auth.log`, it was possible to extract key indicators such as source IP, targeted usernames, event types, and event counts — all relevant data points for a SOC analyst performing log-based threat detection.

The exercise reinforces the importance of SSH hardening and continuous log monitoring as foundational Blue Team practices.

---

## Tools Used

- Kali Linux
- Debian 13
- OpenSSH Server
- rsyslog
- Nmap
- grep, wc

---

---

# SSH Brute Force Detection Lab

> Laboratório Blue Team focado em detectar e analisar tentativas de login SSH falhas utilizando logs de autenticação do Linux.

---

## Objetivo

Simular tentativas repetidas de autenticação SSH falha em um ambiente de laboratório isolado e analisar os logs gerados sob uma perspectiva Blue Team, identificando indicadores de comprometimento e aplicando recomendações defensivas.

---

## Ambiente

| Componente     | Detalhes                 |
| -------------- | ------------------------ |
| VM Atacante    | Kali Linux               |
| VM Alvo        | Debian 13                |
| Rede           | NAT Network (VirtualBox) |
| IP do Atacante | 10.0.2.5                 |
| IP do Alvo     | 10.0.2.15                |
| Fonte de logs  | /var/log/auth.log        |
| Porta SSH      | 22                       |

---

## Etapas Realizadas

### 1. Validação do Serviço SSH

Confirmado que o serviço SSH estava ativo e em execução na máquina alvo.

```bash
sudo systemctl status ssh --no-pager
```

Resultado: `active (running)` — OpenBSD Secure Shell server escutando na porta 22.

---

### 2. Validação do rsyslog

Confirmado que o serviço de logging do sistema estava ativo, garantindo que os eventos de autenticação seriam registrados em `/var/log/auth.log`.

```bash
sudo systemctl status rsyslog --no-pager
```

Resultado: `active (running)` — rsyslog v8.2504.0 em execução e coletando logs do sistema.

---

### 3. Identificação do IP do Alvo

Identificado o endereço IP da máquina Debian alvo.

```bash
ip a
```

**IP do alvo:** `10.0.2.15/24` — interface `enp0s3`

---

### 4. Teste de Conectividade

Verificada a conectividade de rede entre o Kali e o Debian.

```bash
ping -c 4 10.0.2.15
```

**Resultado:** 4 pacotes transmitidos, 4 recebidos, 0% de perda. RTT médio: 0.631ms.

---

### 5. Scan da Porta SSH

Confirmado que a porta 22 estava aberta e acessível na máquina alvo.

```bash
nmap -p 22 10.0.2.15
```

**Resultado:** `22/tcp open ssh` — Nmap 7.95, host ativo (latência 0.00059s).

---

### 6. Tentativas de Autenticação Falha

Geradas tentativas de login SSH falhas a partir do Kali em direção ao Debian.

#### Tentativa com usuário inválido

```bash
ssh fakeuser@10.0.2.15
```

Três tentativas com credenciais incorretas. Resultado: `Permission denied (publickey,password)`.

#### Usuário válido com senha errada

```bash
ssh analyst@10.0.2.15
```

Três tentativas com senha incorreta. Resultado: `Permission denied (publickey,password)`.

---

### 7. Análise de Logs

Filtrados os logs de autenticação para identificar eventos de falha de login.

```bash
sudo grep -E "Failed password|Invalid user" /var/log/auth.log
```

Eventos observados entre `21:34` e `21:37` em 2026-05-05:

- `Invalid user fakeuser from 10.0.2.5 port 46916`
- `Failed password for invalid user fakeuser from 10.0.2.5 port 46916 ssh2` (3 ocorrências)
- `Failed password for analyst from 10.0.2.5 port 54394 ssh2` (3 ocorrências)

---

### 8. Contagem de Eventos

Contabilizado o total de eventos de autenticação falha por tipo.

```bash
sudo grep "Failed password" /var/log/auth.log | wc -l
sudo grep "Invalid user" /var/log/auth.log | wc -l
```

| Tipo de evento  | Quantidade |
| --------------- | ---------- |
| Failed password | 9          |
| Invalid user    | 3          |

---

### 9. Eventos Filtrados por IP de Origem

Filtrados todos os eventos de autenticação originados do IP do atacante.

```bash
sudo grep "10.0.2.5" /var/log/auth.log
sudo grep "10.0.2.5" /var/log/auth.log | wc -l
```

Eventos adicionais observados além das falhas diretas:

- `pam_unix(sshd:auth): authentication failure` — falha de autenticação no nível PAM
- `Connection closed by invalid user fakeuser` — conexão encerrada após máximo de tentativas
- `PAM 2 more authentication failures` — PAM registrando falhas acumuladas
- `Connection closed by authenticating user analyst` — conexão encerrada após máximo de tentativas

**Total de eventos do IP 10.0.2.5:** `15`

---

## Análise

Os logs de autenticação revelaram um padrão claro de tentativas repetidas de login falho originadas de um único endereço IP (`10.0.2.5`) direcionadas ao serviço SSH na porta 22.

Dois padrões distintos foram observados:

- Tentativas contra um **usuário inexistente** (`fakeuser`), gerando eventos `Invalid user`
- Tentativas contra um **usuário válido** (`analyst`) com senhas incorretas, gerando eventos `Failed password`

Além das falhas diretas, os logs também registraram falhas de autenticação no nível PAM e encerramentos de conexão após atingir o número máximo de tentativas — comportamento compatível com atividade de **brute force** ou **password guessing**.

A concentração de 15 eventos relacionados à autenticação originados de um único IP em uma janela de tempo curta (aproximadamente 7 minutos) reforça essa avaliação.

Nenhuma autenticação bem-sucedida foi observada durante a sessão.

---

## Indicadores Principais

| Indicador               | Valor                      |
| ----------------------- | -------------------------- |
| IP de origem            | 10.0.2.5                   |
| Porta alvo              | 22 (SSH)                   |
| Usuários alvo           | fakeuser, analyst          |
| Eventos Failed password | 9                          |
| Eventos Invalid user    | 3                          |
| Total de eventos do IP  | 15                         |
| Janela de tempo         | 21:34 a 21:41 (2026-05-05) |
| Logins bem-sucedidos    | 0                          |
| Fonte de logs           | /var/log/auth.log          |

---

## Recomendações Defensivas

Com base nos achados, os seguintes controles são recomendados:

- Desabilitar autenticação SSH por senha e exigir autenticação por chave pública
- Restringir acesso SSH por IP de origem via regras de firewall
- Implantar `fail2ban` para bloquear automaticamente IPs após falhas repetidas
- Monitorar `/var/log/auth.log` continuamente em busca de anomalias de autenticação
- Evitar expor a porta 22 diretamente a redes não confiáveis
- Considerar alterar a porta padrão do SSH para reduzir ruído de varreduras automatizadas

---

## Conclusão

Este laboratório demonstrou como tentativas repetidas de login SSH falhas geram padrões identificáveis nos logs de autenticação do Linux. Ao analisar o `/var/log/auth.log`, foi possível extrair indicadores-chave como IP de origem, usuários alvo, tipos de evento e contagem de eventos — dados relevantes para um analista SOC realizando detecção de ameaças baseada em logs.

O exercício reforça a importância do hardening de SSH e do monitoramento contínuo de logs como práticas fundamentais de Blue Team.

---

## Ferramentas Utilizadas

- Kali Linux
- Debian 13
- OpenSSH Server
- rsyslog
- Nmap
- grep, wc
