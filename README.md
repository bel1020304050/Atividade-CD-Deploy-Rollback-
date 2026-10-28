# Plano de Deploy e Rollback — Sistema Web de Biblioteca

**Disciplina:** Cloud Computing e DevOps Avançado  
**Professor:** Ranghetti  
**Equipe:** Isabely e Felipe do Nascimento  
**Trabalho:** T6 — Estratégia de CD, deploy e rollback

---

## 1. Introdução

A entrega contínua (Continuous Delivery — CD) permite preparar e validar versões de um sistema para implantação de forma controlada. Neste trabalho, será abordada a aplicação dessas práticas em um sistema web de biblioteca, considerando a disponibilidade do serviço, a segurança das alterações e a possibilidade de recuperação em caso de falhas.

## 2. Relação com o Trabalho 6

Esta atividade constitui a etapa inicial do Trabalho 6, relacionado à estratégia de Continuous Delivery (CD), deploy, rollback e observabilidade. O planejamento será utilizado como base para detalhar o fluxo de entrega, os critérios de sucesso, as evidências de validação e os procedimentos de recuperação do sistema.

## 3. Objetivo

Definir uma estratégia de deploy e rollback para o sistema web de biblioteca, estabelecendo etapas de implantação, critérios de validação, monitoramento e procedimentos de recuperação, com o objetivo de manter a indisponibilidade dentro do limite máximo de 5 minutos.

## 4. Cenário do Sistema

| Item | Descrição |
|------|-----------|
| Sistema | Aplicação web de biblioteca |
| Período de maior utilização | Início de cada semestre |
| Indisponibilidade máxima tolerada | 5 minutos |
| Banco de dados | Compartilhado entre as versões da aplicação |
| Mudança principal | Alteração da funcionalidade de empréstimos (versão 1.0.0 → 1.1.0) |
| Métricas disponíveis | Taxa de erros e latência |

## 5. Estratégia Escolhida

Será utilizada a estratégia **Blue/Green**, com ambientes equivalentes para a versão atual (Blue) e a nova versão (Green). A nova versão é validada antes de receber o tráfego principal, e a estratégia é acompanhada de testes, monitoramento e plano de rollback.

Como o banco de dados é compartilhado, as alterações são feitas de forma compatível com a versão anterior da aplicação (padrão *expandir e contrair*).

**Modelo adotado:** entrega contínua com aprovação humana antes da troca de tráfego em produção.

A comparação com Recreate, Rolling e Canary está em [01-plano-de-deploy.md](01-plano-de-deploy.md).

## 6. Estrutura do Repositório

| Arquivo | Conteúdo |
|---------|----------|
| [README.md](README.md) | Apresentação do trabalho |
| [01-plano-de-deploy.md](01-plano-de-deploy.md) | Versionamento, ambientes, estratégia, banco de dados, smoke tests, critérios de promoção e riscos |
| [02-plano-de-rollback.md](02-plano-de-rollback.md) | Gatilhos, procedimento de recuperação, banco de dados e riscos |
| [03-fluxo-do-pipeline.md](03-fluxo-do-pipeline.md) | Diagramas do pipeline e da arquitetura, gates e evidências |

Os documentos são complementares e devem ser analisados em conjunto.

## 7. Onde encontrar cada entregável do T6

| Entregável | Onde está |
|------------|-----------|
| Contexto e requisitos do serviço | Seção 4 deste README |
| Diagrama do fluxo de entrega | [03](03-fluxo-do-pipeline.md), seções 2 e 3 |
| Estratégia de deploy escolhida | [01](01-plano-de-deploy.md), seção 4 |
| Política de versionamento e promoção | [01](01-plano-de-deploy.md), seção 2 |
| Critérios de sucesso | [01](01-plano-de-deploy.md), seção 9 |
| Smoke tests | [01](01-plano-de-deploy.md), seção 8 |
| Plano de rollback ou roll forward | [02](02-plano-de-rollback.md) |
| Riscos e mitigações | [01](01-plano-de-deploy.md), seção 12, e [02](02-plano-de-rollback.md), seção 12 |
| Evidências que orientam a decisão | [01](01-plano-de-deploy.md), seção 13, e [03](03-fluxo-do-pipeline.md), seção 6 |

## 8. Resultado Esperado

Realizar a implantação de maneira controlada, validar as operações de empréstimo e manter a indisponibilidade dentro do limite de 5 minutos, com um procedimento de recuperação previamente definido.

## 9. Observação sobre os valores numéricos

Os limites (taxa de erros, latência, tempos de recuperação) são **metas iniciais propostas** para o exercício e devem ser confirmados com a linha de base real do sistema.

## 10. Referências

- Aula 11 — *Entrega Contínua, Estratégias de Deploy e Observabilidade*. Prof. Ranghetti. Cloud Computing e DevOps Avançado.
- KOLBE JÚNIOR, Armando. *Computação em Nuvem*. Contentus, 2020.
- MUNIZ, Antonio; IRIGOYEN, Analia. *Jornada DevOps*. Brasport, 2020.
- KUBERNETES. *Deployments*. <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>
- AWS. *Overview of deployment options*. <https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/overview-deployment-options.html>
- GOOGLE. *Site Reliability Engineering*. <https://sre.google/books/>
- OPENTELEMETRY. *Observability Primer*. <https://opentelemetry.io/docs/concepts/observability-primer/>
