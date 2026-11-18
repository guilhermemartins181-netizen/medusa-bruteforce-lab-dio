# 🔐 Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux

> Projeto de laboratório do bootcamp **DIO** — documentação de um exercício prático de _password cracking_ conduzido em **ambiente isolado e autorizado**, usando o **Kali Linux** como máquina atacante e o **Metasploitable 2** como alvo propositalmente vulnerável.

![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)
![Ambiente](https://img.shields.io/badge/ambiente-laborat%C3%B3rio%20isolado-blue)
![Ferramenta](https://img.shields.io/badge/ferramenta-Medusa-red)
![SO](https://img.shields.io/badge/SO-Kali%20Linux-black)

---

## ⚠️ Aviso ético e legal (leia antes de tudo)

Este material é **exclusivamente educacional**. Todo o conteúdo foi executado em um **laboratório fechado**, montado na minha própria máquina, usando a VM **Metasploitable 2** — um sistema criado de propósito para treino de segurança ofensiva/defensiva.

- ✅ **Permitido:** testar sistemas que são **seus** ou para os quais você tem **autorização formal por escrito** (ex.: contrato de pentest, programa de bug bounty com escopo definido, ambientes de treino como Metasploitable, HackTheBox, TryHackMe).
- ⛔ **Proibido:** usar estas técnicas contra qualquer sistema, conta ou rede **sem autorização explícita**. No Brasil, isso configura crime — **Lei nº 12.737/2012 (Lei Carolina Dieckmann)** e **art. 154-A do Código Penal** (invasão de dispositivo informático), além de implicações da **LGPD**.

O objetivo aqui é **entender como o ataque funciona para saber defender**. Quem conhece o funcionamento do brute force projeta senhas, políticas e defesas melhores.

---

## 🎯 Objetivo do laboratório

Demonstrar, de ponta a ponta, como um atacante tentaria **adivinhar credenciais** de um serviço exposto por meio de **força bruta** e **ataques de dicionário**, e — mais importante — **quais controles** tornam esse ataque inviável.

Ao final, o laboratório responde a três perguntas:

1. Como o **Medusa** automatiza a tentativa de combinações usuário/senha?
2. Por que serviços mal configurados e senhas fracas são o elo mais frágil?
3. Quais **mitigações** quebram a eficácia do brute force na prática?

---

## 🧩 O que é força bruta (em uma frase)

> Força bruta é testar **muitas combinações de credenciais** contra um serviço de autenticação até encontrar uma que funcione — seja percorrendo todo o espaço de possibilidades (_pure brute force_) ou usando uma lista pré-montada de candidatos prováveis (_ataque de dicionário_).

| Variante | Ideia central | Quando é usada |
|---|---|---|
| **Pure brute force** | Testa todas as combinações possíveis de caracteres | Senhas curtas; inviável para senhas longas |
| **Ataque de dicionário** | Testa uma _wordlist_ de senhas prováveis | O mais comum — rápido contra senhas fracas |
| **Password spraying** | Poucas senhas (muito comuns) contra **muitos** usuários | Foge de bloqueio por tentativas |
| **Credential stuffing** | Reusa pares usuário/senha vazados em outros sites | Explora reuso de senha entre serviços |

---

## 🛠️ Ferramentas de brute force no Kali Linux

O Kali já traz um arsenal de ferramentas para testes de autenticação. Cada uma tem seu ponto forte:

| Ferramenta | Foco principal |
|---|---|
| **Medusa** | Brute force **paralelo** e modular contra serviços de rede (SSH, FTP, SMB, HTTP, MySQL…) — **foco deste lab** |
| **Hydra** | Alternativa clássica, também multiprotocolo e muito usada em formulários web |
| **Ncrack** | Otimizada para brute force em larga escala, integra com o Nmap |
| **Patator** | Multiuso e flexível, bom para lógica customizada |
| **John the Ripper** | _Offline_ — quebra **hashes** já capturados, não ataca o serviço diretamente |
| **WPScan** | Especializada em WordPress (usuários, plugins, brute force de login) |

### Por que o Medusa?

O Medusa foi escolhido por três características:

- **Velocidade por paralelismo:** testa várias credenciais simultaneamente (`-t` controla o número de threads).
- **Modularidade:** suporta dezenas de protocolos via módulos (`-M ssh`, `-M smbnt`, `-M ftp`, etc.).
- **Estabilidade:** foi projetado para ser leve e consistente em testes longos.

---

## 🧪 Ambiente do laboratório

```
┌─────────────────────────┐          Rede isolada (host-only)          ┌─────────────────────────┐
│      KALI LINUX         │  ───────────────────────────────────────▶  │     METASPLOITABLE 2    │
│   (máquina atacante)    │              força bruta                    │   (alvo vulnerável)     │
│   ferramentas: Medusa,  │                                             │   serviços: SSH, FTP,   │
│   Hydra, Nmap, smbclient│  ◀───────────────────────────────────────  │   SMB, MySQL, Telnet…   │
└─────────────────────────┘            respostas de auth                └─────────────────────────┘
```

| Componente | Papel | Observação |
|---|---|---|
| **Kali Linux** | Máquina atacante | Distribuição voltada a pentest, já traz o Medusa |
| **Metasploitable 2** | Alvo | VM intencionalmente vulnerável (Rapid7) — **nunca** exponha à internet |
| **Rede host-only / interna** | Isolamento | Garante que o tráfego **não sai** do laboratório |
| **Hypervisor** | VirtualBox / VMware | Hospeda as duas VMs |

> 🔒 **Regra de ouro do lab:** a rede é **host-only**. As VMs se enxergam entre si, mas **não têm rota para a internet nem para a rede doméstica**. Isso contém qualquer efeito colateral.

Passo a passo detalhado da montagem: [`docs/01-ambiente.md`](docs/01-ambiente.md).

---

## 🚶 Metodologia (fluxo do ataque)

O laboratório seguiu a lógica de qualquer teste de autenticação bem feito:

```
1. Reconhecimento   →  2. Enumeração    →  3. Preparação     →  4. Ataque        →  5. Validação
   (descobrir o       (quais serviços     (montar listas de     (rodar o Medusa)    (confirmar o
    host e portas)     e usuários)          usuário e senha)                          acesso)
```

### 1. Reconhecimento — localizar o alvo e mapear portas

```bash
# Descobrir o IP do Metasploitable na rede do lab
nmap -sn 192.168.56.0/24

# Mapear serviços e versões no alvo
nmap -sV -p- 192.168.56.101
```

O scan revela os serviços expostos (SSH/22, FTP/21, SMB/445, MySQL/3306, Telnet/23…) — os pontos onde a autenticação pode ser testada. Detalhes em [`docs/02-reconhecimento.md`](docs/02-reconhecimento.md).

### 2. Preparação — criar listas de usuários e senhas

O coração de um ataque de dicionário é a **qualidade da wordlist**. Neste lab, as listas são **pequenas e didáticas** (ver [`wordlists/`](wordlists/)):

```bash
# usuarios.txt  → nomes de conta comuns em servidores Linux/serviços
# senhas.txt    → senhas fracas/padrão típicas de ambientes mal configurados
```

Como gerar e refinar wordlists (incluindo `crunch` e as listas prontas do Kali, como a `rockyou.txt`): [`docs/03-wordlists.md`](docs/03-wordlists.md).

### 3. Ataque — Medusa contra o serviço

A sintaxe base do Medusa:

```bash
medusa -h <alvo> -U <lista_usuarios> -P <lista_senhas> -M <modulo> [opções]
```

| Flag | Significado |
|---|---|
| `-h` | Host alvo (ou `-H` para lista de hosts) |
| `-U` | Arquivo com lista de usuários (ou `-u` para um único) |
| `-P` | Arquivo com lista de senhas (ou `-p` para uma única) |
| `-M` | Módulo/protocolo (`ssh`, `ftp`, `smbnt`, `mysql`…) |
| `-t` | Número de threads (tentativas em paralelo) |
| `-f` | Para no primeiro par válido encontrado por host |
| `-O` | Salva o log da execução em arquivo |

**Exemplo — brute force de SSH no alvo do lab:**

```bash
medusa -h 192.168.56.101 \
       -U wordlists/usuarios.txt \
       -P wordlists/senhas.txt \
       -M ssh \
       -t 4 -f -O evidencias/medusa-ssh.log
```

O passo a passo comentado, com interpretação da saída (`ACCOUNT FOUND`), está em [`docs/04-medusa.md`](docs/04-medusa.md).

### 4. Ataque em cadeia — enumeração SMB + password spraying

O lab também encadeou técnicas em um **cenário corporativo mal configurado**: enumerar compartilhamentos e usuários via SMB, montar uma lista de usuários real e validar acesso com `smbclient`. Documentado em [`docs/05-ataque-em-cadeia-smb.md`](docs/05-ataque-em-cadeia-smb.md).

### 5. Validação

Encontrada uma credencial, o acesso é confirmado logando no serviço (ex.: `ssh usuario@192.168.56.101`). Isso fecha o ciclo e comprova o impacto de uma senha fraca.

---

## 📊 Resultados do laboratório

| Serviço testado | Resultado | Causa raiz |
|---|---|---|
| SSH | Credencial fraca encontrada via dicionário | Senha padrão/trivial + sem rate limiting |
| SMB | Compartilhamentos acessíveis após enumeração | Permissões abertas + usuários previsíveis |

A conclusão prática: **o ataque só teve sucesso porque o alvo estava mal configurado**. Contra um host endurecido, as mesmas tentativas teriam sido bloqueadas ou se tornado inviáveis em tempo. É exatamente aí que entra a parte defensiva.

---

## 🛡️ Mitigações — como quebrar a eficácia do brute force

Esta é a parte mais importante do projeto. Para cada fraqueza explorada, há uma defesa direta:

| Defesa | Por que funciona |
|---|---|
| **Senhas fortes e longas** (frases-senha) | Explode o espaço de busca e torna o dicionário inútil |
| **MFA / 2FA** | Mesmo com a senha certa, falta o segundo fator |
| **Rate limiting + bloqueio de conta** (ex.: `fail2ban`) | Limita tentativas por tempo/IP; o ataque "morre" de lento |
| **Troca de credenciais padrão** | Elimina o alvo mais fácil (admin/admin, msfadmin…) |
| **Chaves SSH em vez de senha** | Remove a senha como vetor |
| **Mínima exposição de serviços** (firewall, VPN) | Reduz a superfície de ataque |
| **Monitoramento e alertas** (SIEM/logs) | Detecta o padrão de tentativas massivas em tempo real |

Aprofundamento em [`docs/06-mitigacoes.md`](docs/06-mitigacoes.md).

---

## 💡 Insights e aprendizados

- **O brute force não "quebra" a criptografia — ele explora a preguiça.** Senha fraca, serviço padrão e ausência de bloqueio são o que viabilizam o ataque.
- **Paralelismo é uma faca de dois gumes:** acelera o atacante, mas gera um ruído enorme nos logs — fácil de detectar com monitoramento.
- **Defender é mais barato que remediar:** uma política de senha + MFA + `fail2ban` neutraliza quase todo o cenário deste lab.
- **Pensar como atacante melhora a defesa:** entender a ferramenta mostra exatamente onde colocar os controles.

Reflexão completa sobre limitações do Medusa e dicas finais: [`docs/07-insights.md`](docs/07-insights.md).

---

## 📁 Estrutura do repositório

```
medusa-bruteforce-lab/
├── README.md                        # este arquivo
├── docs/
│   ├── 01-ambiente.md               # montagem Kali + Metasploitable (rede isolada)
│   ├── 02-reconhecimento.md         # Nmap, descoberta de host e portas
│   ├── 03-wordlists.md              # criação e refino de listas (crunch, rockyou)
│   ├── 04-medusa.md                 # uso do Medusa passo a passo
│   ├── 05-ataque-em-cadeia-smb.md   # enumeração SMB + password spraying
│   ├── 06-mitigacoes.md             # defesas contra brute force
│   └── 07-insights.md               # limitações, dicas e conclusões
├── wordlists/
│   ├── usuarios.txt                 # exemplo didático de usuários
│   └── senhas.txt                   # exemplo didático de senhas fracas
└── evidencias/
    └── README.md                    # onde colocar prints/logs da sua execução
```

---

## 🔗 Referências

- Documentação do **Medusa** (JoMo-Kun / foofus.net)
- **Metasploitable 2** — Rapid7 (VM de treino)
- **Kali Linux** — documentação oficial de ferramentas
- **OWASP** — _Blocking Brute Force Attacks_ e _Authentication Cheat Sheet_
- Lei nº 12.737/2012 e art. 154-A do Código Penal Brasileiro

---

## 👤 Autor

Projeto desenvolvido como parte do desafio de laboratório da [**DIO**](https://www.dio.me/).

> _"A melhor forma de defender um sistema é entender como ele é atacado."_
