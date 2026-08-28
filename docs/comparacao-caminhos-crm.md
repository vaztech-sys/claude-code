# Comparação de caminhos — CRM de vendas vs CRM de WhatsApp

**Status:** apoio à decisão (decisão em aberto)
**Data:** 2026-08-28
**Complementa:** [`decisao-arquitetura-crm.md`](./decisao-arquitetura-crm.md)

---

## Resumo executivo

Surgiu um segundo caminho: em vez de um CRM de vendas tipo Attio, um **CRM de
atendimento por WhatsApp**. São produtos diferentes, e o segundo resolve o
problema de licença que travava o primeiro.

| | Caminho A — CRM de vendas (Attio-like) | Caminho B — CRM de WhatsApp |
|---|---|---|
| Base de referência | Twenty (AGPL + carve-out) | Deskcomm / wacrm (**MIT**) |
| Licença para SaaS proprietário | **Bloqueada** ou paga | **Livre** |
| Tempo até produção | Meses (ou negociação) | Semanas |
| Custo variável por cliente | Nenhum | **Meta, por mensagem** |
| Concorrência | Alta e madura | Alta e pulverizada |
| Risco dominante | Licença / custo de construção | Dependência de plataforma (Meta) |

**A licença deixa de ser o fator decisivo no caminho B; o custo variável passa a ser.**

---

## 1. Licenças (verificado)

| Projeto | Licença | Stars | Último commit | Integração |
|---|---|---|---|---|
| [DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) | **MIT** | 706 | 27/08/2026 | WAHA (QR) + Meta Cloud API |
| [wacrm](https://github.com/ArnasDon/wacrm) | **MIT** | 2.100 | 12/08/2026 | Meta Cloud API |
| Twenty | AGPL-3.0 + carve-out | ~45.000 | ativo | — |

MIT permite fechar o código, revender como SaaS e não devolver nada à
comunidade. É a licença que o modelo de negócio definido (SaaS multi-cliente)
exige. Ambos os projetos estão ativamente mantidos.

---

## 2. O custo que muda a conta

O material de origem afirma "custo de licenciamento zero". Verdadeiro para o
software — e **incompleto**: a Meta cobra por mensagem entregue.

### Tabela Meta (Brasil, 2026)

| Categoria | Custo por mensagem | Quando ocorre |
|---|---|---|
| **Service** | **R$ 0** | Cliente inicia; janela de 24h |
| Utility | R$ 0,04 – 0,05 | Notificação transacional |
| Authentication | R$ 0,15 – 0,19 | Códigos de verificação |
| **Marketing** | **R$ 0,31 – 0,38** | Campanha, disparo ativo |

A distinção é decisiva e corta nos dois sentidos:

- **Atendimento reativo é praticamente gratuito.** Conversa iniciada pelo
  cliente não custa nada. O material subestimou o custo; aqui ele o superestima.
- **Disparo ativo é caro.** É onde a conta quebra.

### Simulação de margem (por cliente atendido)

| Cenário | Volume mensal | Custo Meta | Infra | Total |
|---|---|---|---|---|
| **A** — Suporte reativo | 3.000 conversas do cliente | R$ 0 | R$ 235 | **R$ 235** |
| **B** — Suporte + transacional | 3.000 service + 2.000 utility | R$ 90 | R$ 235 | **R$ 325** |
| **C** — Marketing ativo | 5.000 marketing | **R$ 1.550** | R$ 235 | **R$ 1.785** |

*Infra: VPS ~R$ 100 + Supabase Pro ~R$ 135.*

### Consequência para o modelo de precificação

O material propõe recorrência de R$ 500 a R$ 2.000/mês. Contra os cenários:

- Cenários A e B: margem saudável.
- **Cenário C: o modelo quebra.** R$ 1.550 de custo Meta contra R$ 2.000 de
  receita deixa R$ 450 antes de infraestrutura e do seu trabalho — e piora
  conforme o cliente aumenta os disparos. **Seu cliente mais ativo vira seu
  prejuízo.**

**Correção necessária:** o custo Meta precisa ser repasse direto ao cliente, ou
a recorrência precisa ser escalonada por volume. Mensalidade fixa cobrindo
mensagens de marketing é insustentável.

---

## 3. Limites operacionais (omitido na fonte)

"Escalabilidade ilimitada" é falso. A Meta opera por tiers:

| Situação | Limite / 24h |
|---|---|
| Sem verificação de negócio | 250 contatos |
| Tier 1 (após verificação) | 1.000 |
| Tier 2 | 10.000 |
| Tier 3 | 100.000 |

A progressão exige usar 50% do limite atual em 7 dias mantendo qualidade alta.
Da verificação ao Tier 2 costuma levar de 1 a 4 semanas. **Todo cliente novo
começa limitado** — isso precisa entrar na expectativa comercial, não ser
descoberto depois da venda.

Desde out/2025 os limites são por portfólio, não por número: números novos
herdam o tier mais alto já alcançado — vantagem real para operação de agência.

---

## 4. Esforço e tempo até receita

### Caminho B — CRM de WhatsApp

| Etapa | Prazo |
|---|---|
| Fork, deploy Docker, instância no ar | Dias |
| Verificação de negócio na Meta | Dias a semanas (externo) |
| Branding, ajustes, primeiro cliente | Semanas |
| Ramp de tier até volume útil | 1–4 semanas |

**Gargalo é externo** (aprovação da Meta), não técnico.

### Caminho A — CRM de vendas

| Opção | Prazo | Observação |
|---|---|---|
| Licença comercial Twenty | Semanas | Depende de negociação; custo desconhecido |
| Fork AGPL | Meses | Exige reimplementar billing, RLS e SSO |
| Base própria | Meses | Motor de objetos dinâmicos é o item caro |

---

## 5. Riscos

| Risco | Caminho | Gravidade |
|---|---|---|
| Custo Meta corrói margem em clientes de marketing | B | **Alta** — mitigável com repasse |
| Banimento de número via WAHA/QR (API não oficial) | B | **Alta** — usar apenas Cloud API oficial |
| Mudança unilateral de preço/política da Meta | B | Média — sem mitigação real |
| Tier inicial frustra expectativa do cliente | B | Média — mitigável no contrato |
| Licença AGPL contamina o SaaS | A | **Alta** — exige validação jurídica |
| Custo de construir motor de objetos | A | Alta |
| Projeto base jovem / manutenção incerta | B | Média — ambos ativos hoje |

**Atenção ao Deskcomm:** ele oferece conexão por QR code (WAHA) além da API
oficial. O caminho QR é exatamente o que expõe o número a banimento — o próprio
material de origem alerta sobre isso. Se seguir por aí, use **somente** a Cloud
API oficial.

---

## 6. Leitura crítica da fonte

O material tem base técnica correta, mas é infoproduto — o guia serve de isca
para vender implementação (setup de R$ 5–20k). Distorções encontradas:

| Afirmação | Realidade |
|---|---|
| "Klarna demitiu 700 funcionários; queda de 22% na satisfação; prejuízo de US$ 99 mi" | A IA fez o trabalho *equivalente* a ~700 atendentes; a redução veio de congelamento e rotatividade. A Klarna recuou em mai/2025 e recontratou. **Os números de 22% e US$ 99 mi não têm respaldo** — o documentado é projeção de +US$ 40 mi em 2024 e, no Q3/2025, equivalência de 853 atendentes com satisfação "em paridade" |
| "Escalabilidade ilimitada" | Falso — tiers de 250 a 100.000 |
| "Custo de licenciamento zero" | Verdade só para o software; Meta cobra por mensagem |
| "Encryption Key é exigência da Meta" | Não — é decisão de projeto do app |
| "VPS NVME4" | Não existe; NVMe é protocolo, PCIe Gen4 é interface |
| "10.000 rúpias" | Conteúdo indiano repackaged; régua de preço de outro mercado |
| "Automação de até 90%" | Alegação comercial, não verificável |

A lição de fundo do caso Klarna — não substituir humanos por completo, construir
handoff estruturado — **é sólida** e vale como requisito de produto. Os números
usados para sustentá-la, não.

O material também recomenda Hostinger/HostGator; o Deskcomm tem parceria
comercial com a HostGator. Provável receita de afiliado.

---

## 7. Recomendação

**Os dois caminhos não competem — resolvem problemas diferentes.** A pergunta
não é "qual é melhor", é **qual dor você vende**.

- Se o cliente-alvo sofre com **atendimento por WhatsApp** → caminho B.
  Licença livre, produção em semanas, receita rápida.
- Se sofre com **gestão de pipeline e dados de vendas** → caminho A.
  Investimento maior, ativo mais defensável.

**Se o objetivo é gerar receita e aprender o mercado**, o caminho B é
estratégica­mente superior *hoje*: MIT remove o bloqueio jurídico, o tempo até o
primeiro cliente pagante é de semanas, e a operação ensina o que os clientes
realmente pedem — informação que reduz o risco de errar no caminho A depois.

**Condição para seguir por B:** acertar a precificação antes do primeiro
contrato. Recorrência fixa cobrindo mensagens de marketing é o erro que
transforma crescimento em prejuízo.

---

## 8. Pendências

| Item | Situação |
|---|---|
| Definir a dor vendida (atendimento vs pipeline) | **Decisão do negócio** |
| Modelo de repasse do custo Meta | Pendente — pré-requisito de B |
| Conta Meta Business verificada | Não iniciado — prazo externo |
| Cotação da licença comercial Twenty | Pendente — pré-requisito de A |
| Validação jurídica da AGPL | Pendente — só se A por fork |
| Escolha entre Deskcomm e wacrm | Pendente — exige teste prático dos dois |
