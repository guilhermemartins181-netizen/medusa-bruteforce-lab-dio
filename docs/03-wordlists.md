# 03 — Criação e Refino de Wordlists

A eficácia de um ataque de dicionário depende diretamente da **qualidade da lista**. Uma lista enorme e genérica é lenta; uma lista pequena e bem direcionada é cirúrgica.

## Listas deste laboratório

As listas em [`../wordlists/`](../wordlists/) são **propositalmente pequenas e didáticas** — servem para o ataque terminar rápido no lab e para ilustrar o conceito. Elas contêm usuários comuns de servidores Linux e senhas fracas/padrão típicas de ambientes mal configurados.

## Listas prontas do Kali

O Kali já traz wordlists famosas:

```bash
# A clássica rockyou (precisa descompactar na primeira vez)
ls -lh /usr/share/wordlists/
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Listas de usuários/senhas do SecLists (se instalado)
ls /usr/share/seclists/Passwords/
```

> A `rockyou.txt` tem ~14 milhões de senhas vindas de um vazamento real antigo. É ótima para entender o poder (e o risco) do reuso de senhas — e justamente por isso é usada só em ambiente de treino.

## Gerando listas sob medida com `crunch`

Quando se conhece o **padrão** de senha do alvo (ex.: política "mínimo 6 caracteres"), dá para gerar combinações específicas:

```bash
# Gera todas as combinações de 6 caracteres minúsculos → arquivo
crunch 6 6 abcdefghijklmnopqrstuvwxyz -o lista6.txt

# Padrão: palavra "senha" + 2 dígitos (senha00..senha99)
crunch 7 7 -t senha%% -o senha_dig.txt
```

| Argumento | Significado |
|---|---|
| `6 6` | comprimento mínimo e máximo |
| `-t` | padrão/template (`@` letra, `%` dígito, `^` símbolo) |
| `-o` | arquivo de saída |

## Boa prática de criação de usuários

No lab, a lista de usuários foi montada a partir da **enumeração real** do alvo (ver [`05-ataque-em-cadeia-smb.md`](05-ataque-em-cadeia-smb.md)) — e não chutada. Usuário correto reduz drasticamente o número de tentativas.

> 💡 **Lição defensiva:** políticas de senha que proíbem senhas comuns e exigem comprimento/complexidade tornam qualquer wordlist obsoleta. É a mitigação mais barata e eficaz.
