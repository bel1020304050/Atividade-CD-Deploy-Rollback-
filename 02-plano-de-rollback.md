
 02 — Plano de Rollback

 1. Objetivo

Definir um procedimento de recuperação para retornar o sistema web de biblioteca a uma versão estável caso a nova versão apresente falhas após a implantação.

O objetivo é reduzir o impacto sobre os usuários e manter a indisponibilidade dentro do limite máximo tolerado de 5 minutos.

 2. Gatilhos para Rollback

O rollback deverá ser iniciado quando ocorrer pelo menos uma das seguintes situações:

 Taxa de erros igual ou superior a 2% durante 5 minutos consecutivos, após a troca de tráfego;
 Latência p95 superior a 1 segundo por 5 minutos consecutivos, quando esse limite representar degradação em relação à linha de base aprovada;
 Falha crítica nos testes de empréstimo ou devolução;
 Erros que impeçam login, consulta de livros ou registro de empréstimos;
 Inconsistência ou corrupção de dados confirmada;
 Falha de saúde da aplicação que impeça o atendimento das requisições;
 Indisponibilidade próxima do limite de 5 minutos, sem perspectiva de recuperação imediata;

Para falhas críticas, como corrupção de dados ou impossibilidade de registrar empréstimos corretamente, a equipe não deverá esperar 5 minutos para iniciar a recuperação.

Os limites numéricos são critérios iniciais propostos para o plano e deverão ser confirmados com as métricas reais do sistema.

 3. Procedimento de Rollback:

 Confirmar a falha por meio das métricas, logs e testes;
 Interromper novas etapas de implantação;
 Comunicar a decisão ao responsável pelo deploy e ao restante da equipe;
 Redirecionar o tráfego principal para o ambiente Blue, que mantém a versão estável 1.0.0;
 Confirmar que a versão anterior está saudável e recebendo requisições;
 Desativar a funcionalidade nova por meio de uma feature flag, caso essa opção esteja implementada e seja necessária;
 Verificar login, consulta de livros, empréstimos e devoluções;
 Conferir os registros e a integridade dos dados no banco compartilhado;
 Monitorar as métricas após a recuperação;
 Registrar o incidente, a causa identificada, as ações executadas e os resultados;

O ambiente Blue somente poderá ser utilizado como destino de recuperação se continuar íntegro, disponível e compatível com o estado atual do banco de dados.

 4. Recuperação da Aplicação:

A recuperação da aplicação será feita pelo redirecionamento do tráfego para o ambiente Blue, que executa a versão 1.0.0.

O artefato anterior deverá permanecer identificado e disponível durante o deploy. A equipe não deverá depender da reconstrução da versão antiga durante o incidente.

 5. Recuperação da Configuração:

As configurações anteriores deverão estar registradas e disponíveis para restauração;

Segredos e credenciais deverão permanecer em mecanismos protegidos, sem serem publicados no repositório;

Se uma alteração de configuração causar a falha, a equipe deverá restaurar a configuração aprovada da versão anterior;

 6. Banco de Dados Compartilhado:

O rollback da aplicação não significa desfazer automaticamente as alterações no banco de dados.

Para reduzir o risco:

Realizar backup antes da implantação;
 Verificar a compatibilidade da versão 1.0.0 com o banco atualizado;
Preferir migrações aditivas e compatíveis com a versão anterior;
 Não remover colunas ou estruturas antigas no mesmo momento da implantação;
 Evitar alterações destrutivas que impeçam o funcionamento da versão anterior;
 Verificar empréstimos criados ou alterados durante a janela de falha;
 Preservar registros válidos e investigar inconsistências antes de qualquer correção;

Se houver corrupção de dados, a equipe deverá interromper as operações afetadas e avaliar uma recuperação específica do banco. Restaurar um backup completo pode eliminar operações legítimas realizadas depois dele; portanto, essa ação exige análise e autorização do responsável pelos dados.

Se a nova versão introduzir uma alteração incompatível ou irreversível no banco, poderá ser mais seguro corrigir o problema com uma nova versão (roll forward) do que retornar imediatamente à versão anterior.

 7. Tempo de Recuperação:

Meta proposta: concluir a troca de tráfego de volta para a versão anterior em até 2 minutos após a decisão de rollback.

A meta de recuperação completa é de até 5 minutos de indisponibilidade, conforme o limite do cenário.

Esses tempos são objetivos de planejamento, não resultados já comprovados. A equipe deverá realizar um teste de rollback em ambiente controlado e medir o tempo real de recuperação.

O tempo de detecção da falha também deverá ser considerado no planejamento.

 8. Comunicação

Durante o incidente, o responsável deverá:

1. Avisar a equipe sobre a falha e a decisão de rollback.
2. Informar o responsável pelo sistema e os usuários afetados, conforme o procedimento da organização.
3. Registrar o horário de início, a identificação da versão e o impacto observado.
4. Comunicar quando o serviço estiver novamente estável.
5. Apresentar um resumo da causa e das medidas preventivas.

 9. Evidências do Incidente

Devem ser registrados:

- Versão implantada e versão restaurada.
- Horário de detecção e de início do rollback.
- Taxa de erros e latência observadas.
- Logs relevantes, sem expor senhas ou dados pessoais.
- Resultado dos testes após a recuperação.
- Tempo real de indisponibilidade.
- Estado do banco de dados.
- Responsáveis pelas decisões.
- Causa identificada e ações preventivas.

 10. Critérios de Encerramento

O incidente poderá ser encerrado quando:

- A versão estável estiver atendendo às requisições.
- Os smoke tests forem aprovados.
- As operações de empréstimo e devolução estiverem funcionando.
- A integridade dos dados tiver sido verificada.
- As métricas estiverem dentro dos limites aprovados.
- As evidências e a comunicação do incidente estiverem registradas.

 11. Rollback ou Roll Forward?

O rollback será a primeira opção quando a versão anterior continuar compatível com o banco de dados e os dados estiverem íntegros.

O roll forward será considerado quando a mudança no banco for irreversível, quando a versão anterior não for compatível ou quando uma correção rápida e segura representar menor risco do que a reversão.

## Riscos e Mitigações

| Risco | Mitigação |
|-------|-----------|
| Falha durante o deploy | Executar rollback imediato. |
| Aumento da latência | Monitorar métricas e reverter se necessário. |
| Erro após publicação | Restaurar a versão anterior. |
| Incompatibilidade no banco | Utilizar alterações compatíveis e validar antes da produção. |
