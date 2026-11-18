# 06 — Mitigações contra Força Bruta

A documentação de um ataque só tem valor real quando vira **defesa**. Abaixo, cada fraqueza explorada no lab e o controle que a neutraliza.

## Mapa fraqueza → defesa

| Fraqueza explorada no lab | Mitigação | Efeito |
|---|---|---|
| Senhas fracas/triviais | **Política de senha forte** + proibição de senhas comuns | Dicionário vira inútil |
| Credencial padrão (`msfadmin`) | **Troca obrigatória** de credenciais default | Elimina o alvo mais fácil |
| Sem limite de tentativas | **Rate limiting / lockout** (`fail2ban`) | O ataque "morre" de lento |
| Só senha (fator único) | **MFA / 2FA** | Senha certa ainda não basta |
| SSH por senha | **Autenticação por chave** | Remove a senha como vetor |
| Serviços muito expostos | **Firewall / VPN / mínima exposição** | Reduz a superfície |
| Ataque passava invisível | **Monitoramento + alertas (SIEM)** | Detecção em tempo real |

## Configurações práticas

### fail2ban (bloqueio automático por tentativas)

```bash
sudo apt install fail2ban
# Habilita jail de SSH: após N falhas, o IP é banido por um tempo
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

### Endurecimento do SSH (`/etc/ssh/sshd_config`)

```conf
PermitRootLogin no
PasswordAuthentication no      # força uso de chave
MaxAuthTries 3
AllowUsers usuario_especifico
```

### Política de senha / MFA

- Exigir comprimento mínimo alto (frases-senha) e bloquear senhas de listas vazadas.
- Habilitar **MFA** em todos os acessos remotos e administrativos.

### Reduzir superfície

- Desligar serviços que não precisam estar no ar (Telnet, FTP sem TLS, etc.).
- Expor administração apenas via **VPN** ou rede de gestão segmentada.

## Pirâmide de prioridade

```
        ┌───────────────────────┐
        │         MFA           │  ← mesmo com senha vazada, barra o acesso
        ├───────────────────────┤
        │  Política de senha    │  ← derruba o ataque de dicionário
        ├───────────────────────┤
        │ Rate limit / lockout  │  ← torna o brute force inviável no tempo
        ├───────────────────────┤
        │  Mínima exposição     │  ← menos serviços = menos alvos
        └───────────────────────┘
          Monitoramento cobre tudo isso em paralelo
```

> A combinação **política de senha + MFA + fail2ban** já neutralizaria praticamente todo o cenário deste laboratório.
