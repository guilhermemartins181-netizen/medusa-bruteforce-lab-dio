# 02 — Reconhecimento e Enumeração

Antes de qualquer tentativa de autenticação, é preciso saber **o que** está exposto. Força bruta sem alvo definido é só desperdício.

## Descoberta de host

```bash
# Varredura de hosts ativos na rede do lab
nmap -sn 192.168.56.0/24
```

Isso lista os IPs ativos. O Metasploitable aparece como um host da rede host-only.

## Mapeamento de serviços e versões

```bash
# -sV identifica versão; -p- cobre todas as 65535 portas
nmap -sV -p- 192.168.56.101
```

Saída típica (resumida) do Metasploitable 2:

| Porta | Serviço | Relevância para brute force |
|---|---|---|
| 21 | FTP (vsftpd) | Login usuário/senha |
| 22 | SSH (OpenSSH) | **Alvo clássico do Medusa** |
| 23 | Telnet | Autenticação em texto claro |
| 139/445 | SMB (Samba) | Enumeração + spraying |
| 3306 | MySQL | Login de banco |
| 5432 | PostgreSQL | Login de banco |

## Por que isso importa

Cada serviço de autenticação é um **módulo** possível no Medusa (`-M ssh`, `-M ftp`, `-M smbnt`, `-M mysql`…). O reconhecimento define **onde** faz sentido atacar e **qual módulo** usar.

> 💡 Do ponto de vista defensivo: quanto menos serviços expostos, menor a superfície de ataque. Metade da "vitória" do atacante neste lab vem simplesmente de haver muitos serviços abertos.
