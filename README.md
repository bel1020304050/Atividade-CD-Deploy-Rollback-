
 Plano de Deploy e Rollback — Sistema Web de Biblioteca


 1. Introdução

A entrega contínua (Continuous Delivery — CD) permite preparar e validar versões de um sistema para implantação de forma controlada. Neste trabalho, será abordada a aplicação dessas práticas em um sistema web de biblioteca, considerando a disponibilidade do serviço, a segurança das alterações e a possibilidade de recuperação em caso de falhas.

 2. Relação com o Trabalho 6

Esta atividade constitui a etapa inicial do Trabalho 6, relacionado à estratégia de Continuous Delivery (CD), deploy, rollback e observabilidade. O planejamento será utilizado como base para detalhar o fluxo de entrega, os critérios de sucesso, as evidências de validação e os procedimentos de recuperação do sistema.

 3. Objetivo

Definir uma estratégia de deploy e rollback para o sistema web de biblioteca, estabelecendo etapas de implantação, critérios de validação, monitoramento e procedimentos de recuperação, com o objetivo de manter a indisponibilidade dentro do limite máximo de 5 minutos.

 4. Cenário do Sistema

- Sistema: aplicação web de biblioteca.
- Período de maior utilização: início de cada semestre.
- Indisponibilidade máxima tolerada: 5 minutos.
- Banco de dados: compartilhado entre as versões da aplicação.
- Mudança principal: alteração da funcionalidade de empréstimos.
- Métricas disponíveis: taxa de erros e latência.

 5. Estrutura do Repositório

- README.md – apresentação do trabalho.
- 01-plano-de-deploy.md – planejamento da implantação.
- 02-plano-de-rollback.md – procedimento de recuperação.
- 03-fluxo-do-pipeline.md – fluxo do pipeline de CD.

  Os documentos são complementares e devem ser analisados em conjunto para compreender o processo de implantação, validação, monitoramento e recuperação da aplicação.

 6. Estratégia Escolhida

Será utilizada a estratégia Blue/Green, com ambientes equivalentes para a versão atual e a nova versão. A nova versão será validada antes de receber o tráfego principal. A estratégia será acompanhada de testes, monitoramento e um plano de rollback.

Como o banco de dados é compartilhado, as alterações serão realizadas de forma compatível com a versão anterior da aplicação.

 7. Resultado Esperado

Realizar a implantação de maneira controlada, validar as operações de empréstimo e manter a indisponibilidade dentro do limite de 5 minutos, com um procedimento de recuperação previamente definido.

 8. Equipe

- Integrantes: Isabely e Felipe Do Nascimento 
- Disciplina: Cloud Computing e DevOps Avançado.
- Professor: Ranghetti.
