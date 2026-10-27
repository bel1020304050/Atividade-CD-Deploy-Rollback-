
 Plano de Deploy e Rollback — Sistema Web de Biblioteca

 1. Introdução

Este trabalho apresenta uma estratégia de Continuous Delivery (CD) para um sistema de biblioteca, contemplando planejamento de deploy, rollback, fluxo do pipeline e critérios para tomada de decisão durante uma implantação.

 2. Objetivo

Este trabalho apresenta uma estratégia de Continuous Delivery (CD) para um sistema de biblioteca, contemplando planejamento de deploy, rollback, fluxo do pipeline e critérios para tomada de decisão durante uma implantação.

 3. Cenário do Sistema

- Sistema: aplicação web de biblioteca.
- Período de maior utilização: início de cada semestre.
- Indisponibilidade máxima tolerada: 5 minutos.
- Banco de dados: compartilhado entre as versões da aplicação.
- Mudança principal: alteração da funcionalidade de empréstimos.
- Métricas disponíveis: taxa de erros e latência.

 4. Estrutura do Repositório

- README.md – apresentação do trabalho.
- 01-plano-de-deploy.md – planejamento da implantação.
- 02-plano-de-rollback.md – procedimento de recuperação.
- 03-fluxo-do-pipeline.md – fluxo do pipeline de CD.

## 5. Estratégia Escolhida

Será utilizada a estratégia Blue/Green, com ambientes equivalentes para a versão atual e a nova versão. A nova versão será validada antes de receber o tráfego principal. A estratégia será acompanhada de testes, monitoramento e um plano de rollback.

Como o banco de dados é compartilhado, as alterações serão realizadas de forma compatível com a versão anterior da aplicação.

## 6. Resultado Esperado

Realizar a implantação de maneira controlada, validar as operações de empréstimo e manter a indisponibilidade dentro do limite de 5 minutos, com um procedimento de recuperação previamente definido.

## 7. Equipe

- Integrantes: Isabely e Felipe Do Nascimento 
- Disciplina: Cloud Computing e DevOps Avançado.
- Professor: Ranghetti.
