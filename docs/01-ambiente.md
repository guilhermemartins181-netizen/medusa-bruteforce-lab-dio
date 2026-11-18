# 01 — Montagem do Ambiente

Todo o laboratório roda em **VMs isoladas**, sem rota para a internet. A ideia é reproduzir um alvo realista sem nenhum risco para sistemas reais.

## Componentes

| VM | Função | Fonte |
|---|---|---|
| **Kali Linux** | Máquina atacante | kali.org (imagem para VirtualBox/VMware) |
| **Metasploitable 2** | Alvo vulnerável | Rapid7 (download oficial) |

## Rede: host-only (isolada)

O ponto mais importante da montagem. Configure **as duas VMs** no mesmo adaptador **host-only** (ou rede interna):

- VirtualBox → *Configurações da VM* → *Rede* → *Conectado a: Rede só de hospedeiro (host-only)*.
- As VMs enxergam uma à outra, mas **não** alcançam a internet nem a sua rede doméstica.

> ⚠️ **Nunca** coloque o Metasploitable 2 em modo *bridge* ou com acesso à internet. Ele é cheio de serviços vulneráveis de propósito — exposto, viraria porta de entrada real.

## Verificando a conectividade do lab

```bash
# No Kali, descubra os hosts ativos na rede do laboratório
nmap -sn 192.168.56.0/24

# Teste alcance até o alvo (ajuste o IP conforme sua rede host-only)
ping -c 3 192.168.56.101
```

Credenciais padrão do Metasploitable 2 (apenas para referência de login inicial na própria VM): `msfadmin` / `msfadmin`. Elas são, inclusive, parte do que o ataque de dicionário tende a encontrar — um ótimo exemplo de **credencial padrão não trocada**.

## Checklist antes de começar

- [ ] Kali e Metasploitable no mesmo adaptador host-only
- [ ] Sem acesso à internet nas VMs do alvo
- [ ] IP do alvo confirmado via `nmap -sn`
- [ ] Snapshot das VMs criado (para restaurar o estado limpo depois)
