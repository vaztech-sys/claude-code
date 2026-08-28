# Decisão de arquitetura — CRM

**Status:** em aberto (aguardando decisão)
**Data:** 2026-08-28
**Contexto:** avaliação de bases open-source para construir um CRM inspirado no Attio

---

## 1. Objetivo e restrições

Construir um CRM com o modelo de produto do [Attio](https://attio.com): objetos
flexíveis, views tipo planilha, campos customizáveis por workspace.

Restrições definidas:

| Dimensão | Definição |
|---|---|
| Modelo de negócio | **SaaS multi-cliente** (produto próprio, acesso vendido a terceiros) |
| Stack preferida | Node/TypeScript + PostgreSQL |
| Escopo do MVP | Contatos & Empresas · Pipeline de vendas · Views tipo planilha · Automações |

O modelo de negócio é a restrição dominante: ele é o que torna a análise de
licença decisiva, e não uma formalidade.

---

## 2. Ponto de partida: o Attio não é open-source

O Attio é SaaS proprietário. Não existe "usar o Attio como base" — a referência
é conceitual. A questão real é qual base open-source implementa o mesmo modelo.

---

## 3. Opções avaliadas

| Projeto | Stack | Licença | Aderência ao modelo Attio |
|---|---|---|---|
| **Twenty** | NestJS + React + PostgreSQL | AGPL-3.0 + carve-out comercial | Alta — foi construído como alternativa ao Attio |
| SuiteCRM | PHP | AGPL-3.0 | Baixa — CRM tradicional, UI datada |
| EspoCRM | PHP | AGPL-3.0 (GPL v3 no core) | Média — extensível, mas não é object-first |
| ERPNext | Python/Frappe | GPL-3.0 | Baixa — ERP com módulo de CRM acoplado |
| Odoo | Python | LGPL-3.0 (community) | Baixa — ERP modular; LGPL é a licença mais permissiva do grupo |

Nenhuma alternativa com licença permissiva (MIT/Apache) implementa o modelo de
objetos dinâmicos. **Esse é o achado central da pesquisa:** a licença viral não é
uma característica do Twenty, é o padrão do segmento. Quem resolveu o problema
difícil escolheu copyleft.

O Twenty foi escolhido para avaliação prática por ser o único com aderência alta.

---

## 4. O que foi testado (evidência)

Twenty **v2.37**, instalado a partir do código-fonte e executado localmente:
servidor, front-end, worker de filas, PostgreSQL 16 e Redis, com dados de seed.

**Stack confirmada:** NestJS 11 · React 19 · Apollo Client 4 · TypeORM 0.3.31
(com patch próprio) · PostgreSQL · Redis + BullMQ · GraphQL.
Requer Node 24 e Yarn 4.13.

**APIs expostas:** `/graphql` (dados), `/metadata` (schema), `/admin-panel`,
`/mcp` (integração com agentes de IA).

**Extensões PostgreSQL exigidas:** `uuid-ossp`, `unaccent`, `citext`.
Todas de contrib padrão — não exige imagem customizada, ao contrário do que a
documentação de Docker sugere.

### Cobertura do MVP

Os quatro itens do escopo já existem prontos:

| Requisito | Situação |
|---|---|
| Contatos & Empresas | `People` (29 campos, 1.200 registros) · `Companies` (26 campos, 599) |
| Pipeline de vendas | `Opportunities` (18 campos, 150) — valor, data de fechamento, empresa, contato; view Kanban |
| Views tipo planilha | Interface padrão: filtros, ordenação, colunas ajustáveis, agregações no rodapé |
| Automações | Objeto `Workflows` nativo |

### O modelo de objetos dinâmicos

É o núcleo técnico e o que justifica a avaliação. A UI distingue objetos
`Standard` (nativos) de `Custom` (criados pelo usuário). Na instância de teste
convivem cinco objetos customizados — `Pets` (29 campos), `Survey results` (19),
`Rockets` (12), `Employment Histories` (11), `Pet Care Agreements` (11) — criados
pela seed através da mesma API que um usuário final usaria.

Um objeto criado pela interface ganha tabela real no PostgreSQL, entra
automaticamente nas APIs GraphQL e REST, e aparece na navegação. **É o item mais
caro de construir do zero** e o que separa "um CRM" de "uma plataforma de CRM".

---

## 5. Análise de licença — o ponto decisivo

O `LICENSE` do Twenty é AGPL-3.0 **com um carve-out comercial**: arquivos
marcados `/* @license Enterprise */` ficam fora da AGPL e sob termos
proprietários.

### 5.1 Dimensão do carve-out

Contagem feita sobre o código-fonte da v2.37:

| Pacote | Arquivos fora da AGPL |
|---|---|
| `twenty-server` | 766 |
| `twenty-front` | 52 |
| `twenty-shared` | 5 |
| **Total** | **823** |

Por tema (ocorrências nos nomes de arquivo):

| Tema | Arquivos |
|---|---|
| **Billing** | 282 |
| **Row-level permissions** | 129 |
| **Permissions / Roles** | ~140 |
| **SSO / SAML** | ~85 |

O núcleo do CRM — modelo de objetos, ORM, migrations, views — **é AGPL**. O
carve-out concentra-se em billing, permissões em nível de linha e SSO.

### 5.2 Por que isso importa especificamente para SaaS

Essa distribuição é desfavorável ao caso de uso definido. Billing, permissões
granulares e SSO não são extras de um SaaS multi-tenant — são requisitos
estruturais. **O carve-out cobre justamente a camada que transforma o CRM em
produto vendável.**

Os termos comerciais são explícitos: modificações feitas no código Enterprise
permanecem propriedade da Twenty, e seu uso exige assinatura válida por host e
por assento. Cópia, distribuição, sublicenciamento e venda são vedados fora
desses termos. Desenvolvimento e testes estão isentos — produção, não.

### 5.3 A AGPL sobre o restante

Para os ~99% do código sob AGPL, a cláusula de rede se aplica: oferecer o
software modificado pela rede a terceiros obriga a disponibilizar o código das
modificações a esses usuários. É exatamente o cenário de um SaaS.

> **Ressalva:** esta é uma leitura técnica do texto da licença, não
> aconselhamento jurídico. Antes de fechar qualquer caminho que dependa de
> interpretação da AGPL, valide com um advogado.

---

## 6. Caminhos possíveis

### A. Fork do Twenty aceitando a AGPL
Suas modificações ficam abertas aos usuários do SaaS. Billing, RLS e SSO
precisariam ser reimplementados do zero — não podem ser reaproveitados sem
assinatura.
*Viável se o diferencial for o serviço, não o código. Mas o esforço de
reimplementar a camada Enterprise reduz boa parte da economia do fork.*

### B. Licença comercial com a Twenty
Resolve AGPL e Enterprise de uma vez. Time to market muito menor.
*Custo a negociar e dependência de fornecedor. Recomendado obter a cotação antes
de descartar — é o dado que falta para comparar honestamente com C.*

### C. Base própria, arquitetura inspirada
Next.js + PostgreSQL, com modelo de objetos dinâmicos próprio. Sem restrição de
licença, controle total.
*O item mais caro: o motor de objetos dinâmicos é medido em meses, não semanas.*

### Recomendação

**Buscar a cotação da opção B antes de decidir entre A e C.** A comparação hoje
está incompleta: sem o número da licença comercial, A e C são escolhidas no
escuro. A opção B pode sair mais barata que reimplementar billing + RLS + SSO
(caminho A) ou a plataforma inteira (caminho C).

Se a cotação for proibitiva, a escolha entre A e C depende de uma pergunta de
negócio: **o código do CRM é o seu diferencial competitivo?** Se sim, C. Se o
diferencial é atendimento, integrações locais ou nicho de mercado, A.

---

## 7. Plano de MVP em fases

Aplicável a qualquer um dos caminhos.

| Fase | Entrega | Observação |
|---|---|---|
| **0** | Decisão de licença + ambiente de deploy real | Bloqueia todo o resto |
| **1** | Contatos & Empresas com campos customizáveis | Núcleo do modelo de dados |
| **2** | Views tipo planilha: filtros, ordenação, colunas | Onde o usuário passa o dia |
| **3** | Pipeline/Deals com Kanban por estágio | Fecha o ciclo de vendas |
| **4** | Multi-tenancy, autenticação e permissões | Primeiro requisito exclusivo de SaaS |
| **5** | Automações/Workflows | Diferencial, não fundação |
| **6** | Billing e onboarding self-service | Transforma em produto vendável |

Fases 4 e 6 são as que o carve-out Enterprise do Twenty atinge diretamente.

---

## 8. Riscos e pendências

| Item | Situação |
|---|---|
| Validação jurídica da AGPL | **Pendente** — pré-requisito da opção A |
| Cotação da licença comercial | **Pendente** — pré-requisito da recomendação |
| Ambiente de deploy | Indefinido — a avaliação rodou em container efêmero |
| Dependência de fornecedor (opção B) | A avaliar na negociação |
| Maturidade do Twenty | Projeto ativo e bem mantido; ainda em evolução rápida (v2.37) |

---

## Anexo — reprodutibilidade

A instalação de avaliação usou código-fonte em vez de Docker, porque os
registries de container (Docker Hub, GHCR, ECR Public, Quay) estavam bloqueados
por política de rede no ambiente de teste. Em infraestrutura própria, o
`docker compose` oficial do Twenty é o caminho normal e mais simples.

Requisitos para reproduzir: Node 24 · Yarn 4.13 · PostgreSQL 16 (com `uuid-ossp`,
`unaccent`, `citext`) · Redis.
