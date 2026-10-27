
 01 — Plano de Deploy 

 1. Objetivo

Implantar uma nova versão do sistema web de biblioteca de forma segura, mantendo a indisponibilidade dentro do limite máximo de 5 minutos e evitando falhas nas operações de empréstimo de livros.

 2. Artefato e Versionamento

 Versão atual: 1.0.0.
 Nova versão: 1.1.0.
 Identificação do artefato: versão, tag Git e SHA do commit.
 Política: o artefato aprovado nos testes será o mesmo promovido para homologação e produção, sem recompilação entre os ambientes.
 Registro: cada implantação deverá ter data, versão, responsável e resultado.

A versão 1.1.0 representa uma atualização funcional na operação de empréstimos.

 3. Ambientes

 Desenvolvimento
Ambiente utilizado para implementar e verificar as alterações.

 Testes
Ambiente destinado à execução dos testes automatizados e funcionais.

 Homologação
Ambiente semelhante à produção, utilizado para validar a nova versão antes da implantação definitiva.

 Produção — Blue/Green
- Blue: ambiente que mantém a versão atual 1.0.0.
- Green: ambiente preparado para executar a nova versão 1.1.0.

Os ambientes deverão possuir configurações adequadas e acesso controlado ao banco de dados compartilhado.

Compatibilidade do banco de dados compartilhado

Durante a implantação, os ambientes Blue e Green poderão acessar o mesmo banco de dados. Por isso, qualquer alteração no esquema ou nos dados deverá ser compatível com a versão atual e com a nova versão da aplicação.

As migrações deverão ser planejadas para permitir a convivência temporária das duas versões. Antes do deploy, a equipe deverá validar as alterações em ambiente de testes e confirmar que o rollback da aplicação continuará possível. O backup deverá ser verificado, e qualquer restauração do banco deverá seguir um procedimento específico, considerando o risco de perda de dados.

 4. Estratégia de Deploy

A estratégia escolhida é Blue/Green.

Justificativa:
- Permite validar a nova versão antes de direcionar o tráfego principal.
- Possibilita uma troca controlada entre as versões.
- Facilita o retorno à versão anterior da aplicação.
- Reduz o risco de indisponibilidade durante a implantação.

A estratégia exige capacidade para manter os dois ambientes disponíveis. A troca de tráfego deverá ser testada previamente para respeitar o limite de indisponibilidade de 5 minutos.

 5. Pré-condições:

 Ter a versão 1.1.0 identificada e aprovada.
 Ter os testes obrigatórios aprovados.
 Confirmar que o ambiente Green está disponível e saudável.
 Confirmar que o banco de dados está acessível.
 Verificar se as alterações no banco são compatíveis com a versão 1.0.0.
 Criar e verificar um backup do banco de dados.
 Confirmar que a configuração e os segredos estão protegidos.
 Verificar as métricas atuais de erro e latência.
 Confirmar a disponibilidade dos responsáveis pelo deploy e rollback.
 Evitar a implantação durante o pico de utilização no início do semestre, sempre que possível.

Se uma condição obrigatória não for atendida, a implantação deverá ser interrompida.

 6. Etapas da Implantação

1. Registrar a mudança e identificar o commit da versão 1.1.0.
2. Executar o pipeline de integração contínua.
3. Executar os testes e as verificações de segurança definidos pela equipe.
4. Gerar e identificar o artefato da versão 1.1.0.
5. Promover o mesmo artefato para o ambiente de homologação.
6. Executar os testes funcionais, especialmente os de empréstimos.
7. Validar a compatibilidade da aplicação com o banco de dados compartilhado.
8. Obter a aprovação do responsável pela implantação.
9. Implantar a versão 1.1.0 no ambiente Green.
10. Verificar os health checks e executar os smoke tests.
11. Monitorar erros e latência antes de direcionar o tráfego.
12. Direcionar o tráfego principal para o ambiente Green.
13. Monitorar o sistema por pelo menos 15 minutos após a troca.
14. Registrar as métricas, o resultado e a versão implantada.
15. Manter o ambiente Blue disponível durante o período inicial de validação, até que seja seguro desativá-lo.

 7. Smoke Tests

Os smoke tests são verificações rápidas realizadas após a implantação.

 ST01: verificar se a página inicial do sistema abre corretamente.
 ST02: verificar se o usuário consegue realizar login.
ST03: verificar se o catálogo de livros pode ser consultado.
ST04: verificar se um empréstimo pode ser registrado corretamente em ambiente de teste.
 ST05: verificar se a devolução de um livro continua funcionando.
ST06: verificar se o sistema consulta e grava os dados corretamente no banco.
ST07: verificar se os endpoints de saúde da aplicação respondem adequadamente.

Os testes de empréstimo devem utilizar dados de teste controlados, evitando a criação de empréstimos reais ou duplicados.

 8. Critérios de Promoção

A versão somente poderá receber o tráfego principal quando todos os critérios forem atendidos:

 Testes obrigatórios aprovados.
Nenhum erro crítico ou bloqueador conhecido.
 Health checks aprovados.
 Smoke tests aprovados.
 Operações de empréstimo e devolução funcionando corretamente.
 Banco de dados acessível e consistente.
 Taxa de erros abaixo de 2% durante a janela de validação.
 Latência p95 inferior a 1 segundo, como meta inicial proposta, ou dentro do limite operacional previamente aprovado.
Aprovação registrada pelo responsável pelo deploy.

A taxa de erros e a latência deverão ser avaliadas por pelo menos 5 minutos antes da promoção. Os limites propostos devem ser ajustados caso a linha de base real do sistema indique valores diferentes.

Condições para interromper a promoção

A promoção deverá ser interrompida caso a taxa de erros atinja ou ultrapasse 2%, a latência p95 atinja ou ultrapasse 1 segundo, os health checks falhem ou alguma operação crítica de empréstimo ou devolução apresente erro.

Esses limites são metas iniciais propostas para o exercício e deverão ser confirmados com base na linha de base do sistema. Se houver erro crítico, inconsistência no banco de dados ou risco de perda de dados, a equipe deverá interromper a implantação e avaliar a execução do rollback conforme o plano definido.

 9. Observabilidade

Durante e após o deploy, a equipe deverá acompanhar:

 Taxa de erros HTTP e erros nas operações de empréstimo.
 Latência das requisições, principalmente o percentil p95.
 Disponibilidade e resultado dos health checks.
 Logs de falhas e exceções.
 Resultado dos smoke tests.
Erros de conexão e inconsistências no banco de dados.

As métricas serão comparadas com os valores observados antes da implantação.

 10. Responsáveis

 Responsável pelo deploy: coordena a implantação e autoriza a troca de tráfego.
Responsável pelos testes: executa os testes e registra as evidências.
 Responsável pelo banco de dados: verifica backup, migrações e compatibilidade.
 Responsável pela observabilidade: acompanha métricas, logs e alertas.
 Responsável pelo rollback: coordena a recuperação quando os critérios de sucesso não forem atendidos.

Em uma equipe pequena, uma pessoa poderá acumular funções, desde que as responsabilidades estejam definidas.

 11. Critério de Conclusão

O deploy será considerado concluído quando os testes forem aprovados, as operações críticas funcionarem corretamente, as métricas permanecerem dentro dos limites definidos e a implantação estiver registrada.

Caso os critérios não sejam atendidos, o tráfego deverá retornar para a versão anterior conforme o plano de rollback.

