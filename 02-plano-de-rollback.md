# 02 — Plano de Rollback

## 1. Objetivo

Definir um procedimento de recuperação para retornar o sistema web de biblioteca a uma versão estável caso a nova versão apresente falhas após a implantação.

O objetivo é reduzir o impacto sobre os usuários e manter a indisponibilidade dentro do limite máximo tolerado de 5 minutos.

## 2. Gatilhos para Rollback

O rollback é iniciado quando ocorrer pelo menos uma das situações abaixo. Os limites numéricos vêm da tabela da [seção 9 do plano de deploy](01-plano-de-deploy.md).

| Gatilho | Quando iniciar |
|---------|----------------|
| Taxa de erros ≥ 2% | Após 5 minutos consecutivos, depois da troca de tráfego |
| Latência p95 > 1 s | Após 5 minutos consecutivos, quando representar degradação frente à linha de base aprovada |
| Falha crítica em empréstimo ou devolução | Imediatamente |
| Erros que impeçam login, consulta de livros ou registro de empréstimos | Imediatamente |
| Inconsistência ou corrupção de dados confirmada | Imediatamente |
| Falha de saúde que impeça o atendimento das requisições | Imediatamente |
| Indisponibilidade total confirmada | Imediatamente, pois detecção e troca precisam caber nos 5 minutos |

Para falhas críticas, a equipe **não espera** 5 minutos para iniciar a recuperação. A janela de 5 minutos vale para **degradação** (erros ou lentidão); no caso de sistema fora do ar, o rollback começa assim que a falha é confirmada.

Os limites são metas iniciais propostas e devem ser confirmados com as métricas reais do sistema.

## 3. Procedimento de Rollback

1. Confirmar a falha por meio das métricas, logs e testes.
2. Interromper novas etapas de implantação.
3. Comunicar a decisão ao responsável pelo deploy e ao restante da equipe.
4. Redirecionar o tráfego principal para o ambiente Blue, que mantém a versão estável 1.0.0.
5. Confirmar que a versão anterior está saudável e recebendo requisições.
6. Desativar a funcionalidade nova por meio de feature flag, caso essa opção esteja implementada e seja necessária.
7. Verificar login, consulta de livros, empréstimos e devoluções.
8. Conferir os registros e a integridade dos dados no banco compartilhado.
9. Monitorar as métricas após a recuperação.
10. Registrar o incidente, a causa identificada, as ações executadas e os resultados.

O ambiente Blue só pode ser usado como destino se continuar íntegro, disponível e compatível com o estado atual do banco.

### Exemplos de como executar o passo 4

Exemplos ilustrativos: o comando real depende da infraestrutura usada.

```bash
# Nginx: trocar o servidor ativo do upstream de Green para Blue e recarregar
sudo nginx -t && sudo nginx -s reload

# Kubernetes: apontar o Service de volta para os pods da versão Blue
kubectl patch service biblioteca -p '{"spec":{"selector":{"versao":"blue"}}}'
```

Esse passo deve ser **ensaiado antes** do deploy, para que o rollback seja realmente executável.

## 4. Tempo de Recuperação

| Etapa | Meta proposta |
|-------|---------------|
| Detecção | Imediata para falhas críticas; até 5 min de janela para degradação |
| Decisão | Até 1 min após a confirmação |
| Troca de tráfego de volta para o Blue | Até 2 min após a decisão |
| Verificação rápida (smoke tests) | Logo após a troca |

A meta de recuperação completa é de até 5 minutos de **indisponibilidade**, conforme o limite do cenário.

Esses tempos são objetivos de planejamento, não resultados comprovados. A equipe deve realizar um teste de rollback em ambiente controlado e medir o tempo real.

## 5. Recuperação da Aplicação

A recuperação é feita pelo redirecionamento do tráfego para o ambiente Blue, que executa a versão 1.0.0.

O artefato anterior permanece identificado e disponível durante o deploy. A equipe não depende da reconstrução da versão antiga durante o incidente.

## 6. Recuperação da Configuração

- As configurações anteriores ficam registradas e disponíveis para restauração.
- Segredos e credenciais permanecem em mecanismos protegidos, sem serem publicados no repositório.
- Se uma alteração de configuração causou a falha, restaura-se a configuração aprovada da versão anterior.

## 7. Banco de Dados Compartilhado

O rollback da aplicação **não** desfaz automaticamente as alterações no banco.

Para reduzir o risco:
- realizar backup antes da implantação;
- verificar a compatibilidade da versão 1.0.0 com o banco atualizado;
- preferir migrações aditivas (padrão expandir e contrair do [plano de deploy](01-plano-de-deploy.md));
- não remover colunas ou estruturas antigas no mesmo momento da implantação;
- evitar alterações destrutivas que impeçam o funcionamento da versão anterior;
- verificar empréstimos criados ou alterados durante a janela de falha;
- preservar registros válidos e investigar inconsistências antes de qualquer correção.

Se houver corrupção de dados, a equipe interrompe as operações afetadas e avalia uma recuperação específica do banco. Restaurar um backup completo pode eliminar operações legítimas feitas depois dele, por isso essa ação exige análise e autorização do responsável pelos dados.

## 8. Rollback ou Roll Forward?

| Situação | Decisão |
|----------|---------|
| Versão anterior compatível com o banco e dados íntegros | **Rollback** (primeira opção) |
| Mudança no banco irreversível ou versão anterior incompatível | **Roll forward** |
| Falha pequena e bem compreendida, com correção rápida e segura | **Roll forward** |
| Reversão causaria impacto maior que a correção | **Roll forward** |

O roll forward exige capacidade de produzir e validar uma correção rapidamente.

## 9. Comunicação

Durante o incidente, o responsável deve:

1. Avisar a equipe sobre a falha e a decisão de rollback.
2. Informar o responsável pelo sistema e os usuários afetados, conforme o procedimento da organização.
3. Registrar o horário de início, a identificação da versão e o impacto observado.
4. Comunicar quando o serviço estiver novamente estável.
5. Apresentar um resumo da causa e das medidas preventivas.

## 10. Evidências do Incidente

Devem ser registrados:

- versão implantada e versão restaurada;
- horário de detecção e de início do rollback;
- taxa de erros e latência observadas;
- logs relevantes, sem expor senhas ou dados pessoais;
- resultado dos testes após a recuperação;
- tempo real de indisponibilidade;
- estado do banco de dados;
- responsáveis pelas decisões;
- causa identificada e ações preventivas.

## 11. Critérios de Encerramento

O incidente pode ser encerrado quando:

- a versão estável estiver atendendo às requisições;
- os smoke tests forem aprovados;
- as operações de empréstimo e devolução estiverem funcionando;
- a integridade dos dados tiver sido verificada;
- as métricas estiverem dentro dos limites aprovados;
- as evidências e a comunicação do incidente estiverem registradas.

## 12. Riscos e Mitigações

| Risco | Impacto | Mitigação |
|-------|---------|-----------|
| Falha durante o deploy | Versão nova com erro | Abortar antes da troca; após a troca, executar rollback imediato |
| Aumento da latência | Lentidão para os usuários | Monitorar as métricas e reverter se passar do limite por 5 min |
| Erro após a publicação | Funcionalidade quebrada | Redirecionar o tráfego para o Blue |
| Incompatibilidade no banco | Blue não consegue operar | Alterações compatíveis (expandir e contrair) e validação antes da produção |
| Blue desligado ou desatualizado | Sem destino de rollback | Manter o Blue ativo até o fim da validação |
| Rollback mais lento que o previsto | Indisponibilidade acima de 5 min | Ensaiar o rollback e medir o tempo real |
| Perda de empréstimos registrados durante a falha | Dados legítimos perdidos | Conferir os registros da janela e evitar restaurar backup completo sem análise |
| Sem feature flag para desativar a função nova | Rollback só por troca de ambiente | Usar o redirecionamento ao Blue como mecanismo principal |
