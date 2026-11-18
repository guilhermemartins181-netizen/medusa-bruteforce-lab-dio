# Evidências

Prints da execução real do laboratório:

- **`01-nmap-reconhecimento.png`** — varredura com `nmap` mostrando os serviços expostos do Metasploitable 2 (192.168.56.101).
- **`02-medusa-account-found.png`** — ataque com Medusa retornando `ACCOUNT FOUND: [ssh] User: msfadmin Password: msfadmin [SUCCESS]`.

---

Para novos prints, coloque aqui os **prints e logs da própria execução** no laboratório, para dar prova visual do processo (recrutadores gostam disso):

Sugestões de capturas:

1. `nmap` mostrando os serviços do alvo.
2. Comando do Medusa em execução.
3. A linha `ACCOUNT FOUND ... [SUCCESS]`.
4. Login válido confirmado (ex.: `ssh` conectado).
5. (Opcional) `fail2ban` bloqueando as tentativas — a parte defensiva.

Salve os logs do Medusa usando a flag `-O`, por exemplo:

```bash
medusa -h 192.168.56.101 -U ../wordlists/usuarios.txt -P ../wordlists/senhas.txt -M ssh -t 4 -f -O medusa-ssh.log
```

> Dica: tarje/oculte qualquer dado que não queira público. Como é tudo lab isolado, normalmente não há nada sensível — mas é um bom hábito.
