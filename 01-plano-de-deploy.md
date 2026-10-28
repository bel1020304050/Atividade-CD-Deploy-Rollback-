    # 01 — Plano de Deploy

    ## 1. Objetivo

    Implantar uma nova versão do sistema web de biblioteca de forma segura, mantendo a indisponibilidade dentro do limite máximo de 5 minutos e evitando falhas nas operações de empréstimo de livros.

    O modelo adotado é de **entrega contínua**: o pipeline deixa a versão pronta para produção, mas a troca de tráfego depende de aprovação humana registrada.

    ## 2. Artefato, Versionamento e Promoção

    ### 2.1 Identificação do artefato

    | Item | Definição |
    |------|-----------|
    | Versão atual | 1.0.0 |
    | Nova versão | 1.1.0 (alteração funcional em empréstimos, compatível com a anterior) |
    | Política de versão | Versão semântica `MAJOR.MINOR.PATCH` |
    | Tag Git | `v1.1.0` |
    | Rastreabilidade | Versão + tag Git + SHA do commit + número do pipeline |
    | Registro | Cada implantação registra data, versão, responsável e resultado (modelo na seção 13) |

    ### 2.2 Política de promoção

    O artefato é gerado **uma única vez** pelo pipeline de CI. O mesmo artefato aprovado nos testes é promovido para homologação e produção, **sem recompilação** entre os ambientes. Recompilar geraria uma saída diferente da testada e quebraria a rastreabilidade.

    ```mermaid
    flowchart LR
        A["Artefato v1.1.0 (tag + SHA)"] --> D[Desenvolvimento]
        D --> T[Testes]
        T --> H[Homologação]
        H --> P["Produção (Green)"]
    ```

    O que muda entre os ambientes é apenas a **configuração**, aplicada de fora do artefato.

    ### 2.3 Configuração e segredos

    - A configuração varia por ambiente e fica fora do artefato (variáveis de ambiente ou arquivo de configuração do ambiente).
    - Segredos e credenciais vêm de mecanismo protegido (ex.: variáveis protegidas do pipeline ou cofre de segredos), nunca do repositório nem do artefato.
    - A configuração do Green é validada antes da promoção.
    - Alterações de configuração são registradas (quem e quando).
    - A configuração aprovada da versão anterior é preservada para uso no rollback.

    ## 3. Ambientes

    | Ambiente | Finalidade |
    |----------|-----------|
    | Desenvolvimento | Implementar e verificar as alterações |
    | Testes | Executar testes automatizados e funcionais |
    | Homologação | Ambiente semelhante à produção, para validar a nova versão antes da implantação definitiva |
    | Produção — Blue | Mantém a versão atual 1.0.0 e recebe o tráfego principal |
    | Produção — Green | Preparado para executar a nova versão 1.1.0 |

    Os ambientes devem ter configurações adequadas e acesso controlado ao banco de dados compartilhado. O diagrama da arquitetura Blue/Green está em [03-fluxo-do-pipeline.md](03-fluxo-do-pipeline.md).

    ## 4. Estratégia de Deploy

    ### 4.1 Comparação das estratégias

    | Estratégia | Custo | Risco inicial | Reversão | Complexidade | Adequação ao cenário |
    |------------|-------|---------------|----------|--------------|----------------------|
    | Recreate | Baixo | Alto | Mais lenta | Baixa | Não adequada: encerra a versão antiga antes de subir a nova e pode estourar os 5 minutos |
    | Rolling | Médio | Médio | Gradual | Média | Parcial: exige conviver com versões misturadas e a falha pode demorar a ser percebida |
    | **Blue/Green** | Alto | Baixo | Rápida | Média | **Escolhida** |
    | Canary | Variável | Baixo | Por tráfego | Alta | Boa em teoria, mas exige divisão de tráfego e observabilidade mais sofisticadas do que a equipe dispõe (só taxa de erros e latência) |

    ### 4.2 Por que Blue/Green

    - Permite validar a nova versão (health checks e smoke tests) **antes** de direcionar o tráfego principal.
    - A troca é uma mudança de rota, rápida e controlada, o que ajuda a respeitar o limite de 5 minutos.
    - O retorno à versão anterior é um redirecionamento ao Blue, que continua no ar.
    - Reduz o risco de indisponibilidade durante a implantação.

    ### 4.3 Custo e limitações

    - Exige capacidade para manter os dois ambientes ao mesmo tempo, inclusive para o pico do início do semestre.
    - Não resolve sozinho o banco de dados compartilhado (seção 5).
    - A troca de tráfego deve ser testada previamente para confirmar que cabe nos 5 minutos.

    ## 5. Banco de Dados Compartilhado

    Blue e Green acessam o mesmo banco durante a implantação. Por isso, toda alteração de esquema ou dados deve funcionar com a versão atual **e** com a nova. Para isso será usado o padrão **expandir e contrair** (expand/contract):

    | Fase | O que fazer | Exemplo (hipotético, para empréstimos) |
    |------|-------------|----------------------------------------|
    | 1. Expandir | Adicionar a estrutura nova sem remover nada | Criar coluna opcional (aceita nulo) para o novo dado do empréstimo |
    | 2. Conviver | A versão 1.1.0 passa a usar a estrutura nova; a 1.0.0 continua funcionando e ignora a coluna | Blue e Green rodando juntos sem erro |
    | 3. Migrar | Migrar os dados antigos para o novo formato, se necessário | Preencher a coluna para empréstimos existentes |
    | 4. Contrair | Remover a estrutura antiga **em uma release posterior**, depois que a nova estiver estável | Remoção na versão seguinte, nunca no mesmo deploy |

    Antes do deploy:
    - validar as migrações em ambiente de testes;
    - confirmar que o rollback da aplicação continuará possível;
    - verificar o backup;
    - evitar migrações que travem tabelas durante o pico de uso.

    Restaurar o banco a partir de backup segue procedimento específico, pelo risco de perda de dados (ver [02-plano-de-rollback.md](02-plano-de-rollback.md)).

    ## 6. Pré-condições

    Se uma condição obrigatória não for atendida, a implantação é interrompida.

    - [ ] Versão 1.1.0 identificada e aprovada.
    - [ ] Testes obrigatórios aprovados.
    - [ ] Ambiente Green disponível e saudável.
    - [ ] Banco de dados acessível.
    - [ ] Alterações no banco compatíveis com a versão 1.0.0.
    - [ ] Backup do banco criado e verificado.
    - [ ] Configuração e segredos protegidos.
    - [ ] Métricas atuais de erro e latência registradas (linha de base).
    - [ ] Responsáveis pelo deploy e pelo rollback disponíveis.
    - [ ] Implantação fora do pico de utilização do início do semestre, sempre que possível.

    ## 7. Etapas da Implantação

    1. Registrar a mudança e identificar o commit da versão 1.1.0.
    2. Executar o pipeline de integração contínua.
    3. Executar os testes e as verificações de segurança definidos pela equipe.
    4. Gerar e identificar o artefato da versão 1.1.0.
    5. Promover o mesmo artefato para o ambiente de homologação.
    6. Executar os testes funcionais, especialmente os de empréstimos.
    7. Validar a compatibilidade da aplicação com o banco de dados compartilhado.
    8. Obter a aprovação do responsável pela implantação (registrada com responsável, evidências, critérios e data/hora).
    9. Implantar a versão 1.1.0 no ambiente Green.
    10. Verificar os health checks e executar os smoke tests.
    11. Monitorar erros e latência do Green por pelo menos 5 minutos, com tráfego de teste, antes de direcionar o tráfego principal.
    12. Direcionar o tráfego principal para o ambiente Green.
    13. Monitorar o sistema por pelo menos 15 minutos após a troca.
    14. Registrar as métricas, o resultado e a versão implantada.
    15. Manter o ambiente Blue disponível durante o período inicial de validação, até que seja seguro desativá-lo.

    ## 8. Health Checks e Smoke Tests

    ### 8.1 Health checks

    | Tipo | Pergunta | Exemplo de verificação |
    |------|----------|------------------------|
    | Startup | A aplicação terminou de iniciar? | Processo iniciado e configuração carregada |
    | Liveness | O processo precisa ser reiniciado? | Endpoint `/health/live` responde |
    | Readiness | A instância pode receber tráfego? | Endpoint `/health/ready` só responde OK se o banco de dados estiver acessível |

    O readiness deve refletir condições reais. Um endpoint que sempre retorna `200` dá falsa segurança.

    ### 8.2 Smoke tests

    Verificações rápidas feitas após a implantação no Green. Não substituem os testes completos.

    | ID | Verificação | Resultado esperado |
    |----|-------------|--------------------|
    | ST01 | Abrir a página inicial | Página carrega sem erro |
    | ST02 | Fazer login | Usuário de teste autenticado |
    | ST03 | Consultar o catálogo de livros | Lista de livros exibida |
    | ST04 | Registrar um empréstimo (ambiente de teste) | Empréstimo gravado corretamente |
    | ST05 | Registrar a devolução de um livro | Devolução concluída e estoque atualizado |
    | ST06 | Consultar e gravar dados no banco | Leitura e escrita sem erro |
    | ST07 | Chamar os endpoints de saúde | Respondem adequadamente |

    Os testes de empréstimo usam dados de teste controlados, evitando empréstimos reais ou duplicados.

    ## 9. Critérios de Promoção e Métricas

    Esta é a tabela de referência para promoção, interrupção e rollback. Os planos [02](02-plano-de-rollback.md) e [03](03-fluxo-do-pipeline.md) usam estes mesmos valores.

    | Métrica | Promover (antes da troca) | Abortar a promoção | Rollback (após a troca) |
    |---------|---------------------------|--------------------|-------------------------|
    | Taxa de erros | Abaixo de 2% por pelo menos 5 min | 2% ou mais | 2% ou mais por 5 min consecutivos |
    | Latência p95 | Abaixo de 1 s (ou dentro do limite aprovado) | 1 s ou mais | Acima de 1 s por 5 min consecutivos, se for degradação frente à linha de base |
    | Health checks | Todos aprovados | Qualquer falha | Falha que impeça atender requisições |
    | Smoke tests | ST01–ST07 aprovados | Qualquer falha | Falha crítica em empréstimo ou devolução |
    | Empréstimo e devolução | Funcionando sem inconsistência | Qualquer erro | Erro que impeça registrar empréstimo |
    | Banco de dados | Acessível e consistente | Inconsistência ou risco de perda de dados | Inconsistência ou corrupção confirmada |
    | Saturação (CPU, memória, conexões) | Abaixo de 80% | Acima de 80% | Saturação que cause erros ou lentidão |

    Demais condições para promover: nenhum erro crítico conhecido e aprovação registrada do responsável pelo deploy.

    Os limites são metas iniciais propostas para o exercício e devem ser ajustados caso a linha de base real do sistema indique valores diferentes. Se houver erro crítico, inconsistência no banco ou risco de perda de dados, a equipe interrompe a implantação e avalia o rollback conforme o plano.

    ## 10. Observabilidade

    Durante e após o deploy, a equipe acompanha:

    - **Métricas:** taxa de erros HTTP, latência (principalmente p95), volume de requisições, saturação de recursos e reinicializações inesperadas.
    - **Métrica de negócio:** taxa de conclusão de empréstimos e devoluções.
    - **Logs:** falhas e exceções (sem expor senhas ou dados pessoais).
    - **Saúde:** resultado dos health checks e dos smoke tests.
    - **Banco de dados:** erros de conexão e inconsistências.

    Os valores são comparados com os observados antes da implantação. O deploy não termina no "pipeline verde": é preciso observar uma janela após a troca.

    ## 11. Responsáveis

    | Função | Responsabilidade |
    |--------|------------------|
    | Deploy | Coordena a implantação e autoriza a troca de tráfego |
    | Testes | Executa os testes e registra as evidências |
    | Banco de dados | Verifica backup, migrações e compatibilidade |
    | Observabilidade | Acompanha métricas, logs e alertas |
    | Rollback | Coordena a recuperação quando os critérios não forem atendidos |

    Em uma equipe pequena, uma pessoa pode acumular funções, desde que as responsabilidades estejam definidas.

    ## 12. Riscos e Mitigações

    | Risco | Mitigação |
    |-------|-----------|
    | Falha na nova versão | Testes antes do deploy e Blue mantido disponível para rollback |
    | Incompatibilidade com o banco | Padrão expandir e contrair, validação em testes e backup antes do deploy |
    | Aumento de erros ou latência | Monitorar antes e depois da troca e abortar se os critérios da seção 9 não forem atendidos |
    | Falha em empréstimo ou devolução | Smoke tests com dados controlados antes de liberar o tráfego |
    | Indisponibilidade acima de 5 minutos | Testar a troca de tráfego previamente e definir responsáveis |
    | Perda de dados na recuperação | Avaliar a integridade antes de restaurar backup e preservar operações válidas |
    | Sessões de usuários perdidas na troca Blue → Green | Usar armazenamento de sessão compartilhado ou avisar que será necessário novo login |
    | Tarefas agendadas (ex.: multas, renovações) executando em duplicidade nos dois ambientes | Manter agendadores ativos em apenas um ambiente por vez |
    | Excesso de conexões no banco com os dois ambientes ativos | Limitar o pool de conexões de cada ambiente |
    | Migração que trava tabelas no pico de uso | Aplicar migrações fora do pico e em etapas pequenas |
    | Green sem capacidade para o pico do semestre | Dimensionar o Green igual ao Blue e evitar deploy no pico |

    ## 13. Registro do Deploy

    Modelo a preencher a cada implantação (evidência da decisão):

    | Campo | Valor |
    |-------|-------|
    | Data e hora | |
    | Versão / tag / SHA | |
    | Responsável pelo deploy | |
    | Métricas antes (erros, p95) | |
    | Resultado dos smoke tests | |
    | Métricas depois (15 min) | |
    | Resultado final | Sucesso / Rollback |
    | Observações | |

    ## 14. Critério de Conclusão

    O deploy é considerado concluído quando os testes forem aprovados, as operações críticas funcionarem corretamente, as métricas permanecerem dentro dos limites da seção 9 durante a janela de 15 minutos e a implantação estiver registrada.

    Caso os critérios não sejam atendidos, o tráfego retorna à versão anterior conforme o [plano de rollback](02-plano-de-rollback.md).
