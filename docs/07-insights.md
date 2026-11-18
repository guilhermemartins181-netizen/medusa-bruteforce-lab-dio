# 07 — Limitações do Medusa, Dicas Finais e Conclusões

## Limitações do Medusa

| Limitação | Detalhe |
|---|---|
| **Depende do serviço responder** | Se o alvo tem rate limiting/lockout, o Medusa fica lento ou é bloqueado |
| **Barulhento** | Muitas tentativas geram logs e alertas — fácil de detectar |
| **Não quebra hashes** | Ataca o serviço vivo; para hashes capturados, usa-se John/Hashcat (offline) |
| **Qualidade da wordlist manda** | Lista ruim = ataque ineficaz, por mais rápido que seja |
| **Threads demais derrubam o serviço** | `-t` alto pode causar falhas de conexão e falsos negativos |

## Dicas finais

- **Comece pelo reconhecimento.** Atacar sem saber o serviço/usuário certo é desperdício.
- **Enumere antes de forçar.** Um usuário real vale mais que mil chutes.
- **Ajuste as threads com cuidado.** Estabilidade > velocidade bruta em testes sérios.
- **Documente tudo.** Logs (`-O`) e evidências são o que transformam um teste em um relatório útil.
- **Sempre com autorização.** Sem escopo assinado, não existe "só testando".

## Conclusão

O laboratório mostrou, na prática, que **força bruta não derrota a criptografia — ela explora falhas de configuração e senhas fracas**. O sucesso contra o Metasploitable 2 veio inteiramente de:

1. serviços expostos demais,
2. credenciais padrão não trocadas,
3. ausência de limite de tentativas,
4. senhas triviais.

Cada um desses pontos tem uma defesa barata e conhecida. O maior aprendizado é que **entender a ferramenta ofensiva é o caminho mais curto para projetar a defesa certa**: depois de ver o Medusa achar `msfadmin:msfadmin` em segundos, fica óbvio por que política de senha, MFA e `fail2ban` não são opcionais.

> 🎯 **Resumo em uma linha:** o brute force é um teste de disciplina de configuração — e quase sempre o defensor perde por descuido, não por falta de tecnologia.
