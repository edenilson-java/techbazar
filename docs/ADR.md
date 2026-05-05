# ADR — Arquitetura e stack do techbazar

**Data:** 2026-05-04
**Status:** Aceito
**Autor:** Equipe techbazar (4 membros)
**Escopo:** Mini-Projeto Avaliativo — Hands-on 1 "A Gênese", IA para DEVs (Módulo 1, Semana 5, Aula 1 — SESI/SENAI-SC).

---

## 1. Contexto

O techbazar é um marketplace simplificado para compra e venda de eletrônicos usados entre pessoas físicas (estilo OLX/Mercado Livre enxuto). O escopo do MVP cobre quatro domínios principais:

- **Cadastro de usuários** (compradores e vendedores na mesma base).
- **Anúncio e listagem de produtos** com busca e filtros.
- **Carrinho** simples por usuário.
- **Checkout** com confirmação de pedido.

A equipe é composta por **4 membros** com background majoritário em Java. A trilha da disciplina segue o ecossistema Spring/Maven nas semanas posteriores, então a escolha tecnológica precisa preservar continuidade pedagógica e operacional.

Este ADR consolida as decisões arquiteturais e de stack que orientarão o projeto. É o ADR único do MVP — decisões pontuais futuras serão registradas em ADRs adicionais somente se justificadas por mudança de escopo.

---

## 2. Decisão — Estilo arquitetural

Adotar **Monólito Modular** com separação por domínio em uma única aplicação Spring Boot:

```
techbazar (1 deployable)
├── user      — autenticação, cadastro, perfil
├── catalog   — produtos, categorias, busca
├── cart      — carrinho do usuário
└── checkout  — pedido, pagamento, notificação
```

Cada módulo expõe uma fronteira clara (pacote/serviço próprio), comunica-se com os demais por chamadas in-process e compartilha um único banco PostgreSQL com schema lógico segregado por domínio.

A justificativa central é o tamanho da equipe: **4 membros é pequeno demais para absorver o overhead operacional de microsserviços** (CI/CD múltiplo, observabilidade distribuída, contratos versionados entre serviços, ambientes locais com vários processos), **mas grande o suficiente para se beneficiar da divisão por módulos/domínios** — permite que duplas trabalhem em features distintas sem pisar no mesmo código. A modularização interna mantém a porta aberta para uma futura migração para microsserviços, caso o produto cresça e a equipe se expanda.

---

## 3. Alternativas consideradas

| Alternativa | Por que foi descartada |
|---|---|
| **Microsserviços desde o MVP** | Equipe de 4 membros não absorve o overhead (CI/CD múltiplo, service discovery, observabilidade distribuída, contratos versionados). Complexidade incompatível com escopo de aula e prazo. |
| **Serverless (AWS Lambda + API Gateway + DynamoDB)** | O slide da aula cita como exemplo válido, porém exige modelagem NoSQL (incompatível com o domínio fortemente relacional Usuário ↔ Produto ↔ Pedido) e ferramental fora do que será visto nas semanas seguintes da disciplina. Quebraria a continuidade com Spring/Maven. |
| **Node.js Monolito** | Java/Spring é o padrão da disciplina e o background majoritário da equipe. Trocar para Node introduziria curva de aprendizado sem ganho técnico relevante para o escopo. |

---

## 4. Decisão — Stack tecnológica

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Backend | Java 21 + Spring Boot 3 | Familiaridade da equipe; ecossistema maduro; alinhado com a trilha da disciplina. |
| Persistência | PostgreSQL + JPA/Hibernate | Domínio relacional (Usuário ↔ Produto ↔ Pedido); JPA reduz boilerplate de DAO. |
| API | REST + OpenAPI/Swagger | Padrão da disciplina; documentação automática facilita avaliação e integração. |
| Frontend (mockup) | HTML5 + Tailwind via CDN | Atende ao Hands-on sem build; permite iteração rápida do layout. |
| Frontend (futuro) | React | Mencionado em aula como evolução natural após o mockup; fora do escopo deste Hands-on. |
| Autenticação | Spring Security + JWT | Stateless; integra naturalmente com Spring Boot; preparado para múltiplos clientes. |
| Build / CI | Maven + GitHub Actions | Padrão Java; pipeline de build/test visto na disciplina. |

---

## 5. Consequências

**Habilitadas pela decisão:**

- Deploy único e simples — um único JAR, um banco, um pipeline.
- Onboarding rápido de novos membros — toda a aplicação em um único repositório, sem necessidade de orquestração local.
- Separação por domínio facilita trabalho paralelo da equipe e prepara o caminho para uma futura extração de microsserviços.
- Transações cruzando módulos (ex.: checkout consultando carrinho e produto) ficam consistentes via JPA, sem coordenação distribuída.

**Limitações aceitas:**

- Não é possível escalar módulos independentemente — todo o monólito sobe junto.
- Acoplamento de release: uma alteração no módulo `cart` exige redeploy do `catalog` e demais.
- Falha em um módulo pode degradar a aplicação inteira (mitigado por testes e isolamento de pacotes; aceitável para o MVP).
