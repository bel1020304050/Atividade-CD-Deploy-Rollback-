
# Plano de Deploy e Rollback — Sistema Web de Biblioteca

## 1. Introdução

Esta atividade apresenta um plano de Entrega Contínua (CD), implantação (deploy), observabilidade e recuperação (rollback) para um sistema web de biblioteca. A proposta considera a necessidade de manter o serviço disponível, reduzir os riscos de falhas e garantir a segurança das operações de empréstimo de livros.

## 2. Objetivo

Definir uma estratégia segura para implantar uma nova versão do sistema, estabelecer critérios de validação e determinar procedimentos de recuperação caso a implantação apresente problemas.

## 3. Cenário do Sistema

- Sistema: aplicação web de biblioteca.
- Período de maior utilização: início de cada semestre.
- Indisponibilidade máxima tolerada: 5 minutos.
- Banco de dados: compartilhado entre as versões da aplicação.
- Mudança principal: alteração da funcionalidade de empréstimos.
- Métricas disponíveis: taxa de erros e latência.

## 4. Arquivos do Trabalho

- `01-plano-de-deploy.md`: estratégia e etapas da implantação.
- `02-plano-de-rollback.md`: critérios e procedimentos de recuperação.
- `03-fluxo-do-pipeline.md`: fluxo visual do processo de entrega.

## 5. Estratégia Escolhida

Será utilizada a estratégia Blue/Green, com ambientes equivalentes para a versão atual e a nova versão. A nova versão será validada antes de receber o tráfego principal. A estratégia será acompanhada de testes, monitoramento e um plano de rollback.

Como o banco de dados é compartilhado, as alterações serão realizadas de forma compatível com a versão anterior da aplicação.

## 6. Resultado Esperado

Realizar a implantação de maneira controlada, validar as operações de empréstimo e manter a indisponibilidade dentro do limite de 5 minutos, com um procedimento de recuperação previamente definido.

## 7. Equipe

- Integrantes: Isabely e Felipe Do Nascimento 
- Disciplina: Cloud Computing e DevOps Avançado.
- Professor: Ranghetti.
