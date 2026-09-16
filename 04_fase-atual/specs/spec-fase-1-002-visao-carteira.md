# SPEC-1-002 — Visão unificada de carteira

**Fase:** 1  
**Status:** planejada  
**Dono:** Produto/Sistemas Hardlink  
**Origem no escopo:** RT-001, RT-003, Fase 1  
**Degrau da solução:** construção mínima — entregar uma consulta operacional palpável sobre os dados carregados.

## Contexto e decisões fechadas

- **Estado atual:** dados comerciais existem em arquivos/views, mas a leitura consolidada depende de Power BI, CRM e filtros manuais.
- **Estado desejado:** usuário autorizado encontra cliente, responsável, forecasts, itens e leads associados em uma visão única.
- **Decisões já fechadas:** a associação inicial usa identificadores existentes; casos ambíguos aparecem como exceção.
- **Bloqueios:** nenhum para MVP com dados importados.

## Resultado observável

Uma tela ou rota de consulta permite filtrar carteira por cliente, responsável, status, campanha/origem e data de atualização, exibindo fonte e vínculo dos dados.

## Limites e dependências

- **Inclui:** listagem, filtros, detalhe do cliente e indicadores simples por carteira.
- **Fora de escopo:** edição de cliente, criação de oportunidade, scoring avançado e IA.
- **Entradas e pré-condições:** dados publicados pela SPEC-1-001.
- **Saídas/artefatos:** tela/endpoint de carteira e relatório de exceções de vínculo.
- **Dependências e responsáveis:** Sistemas Hardlink implementa; gestor valida amostra.
- **Atores e permissões mínimas:** executivo vê sua carteira; gestor vê sua alçada; administrador vê tudo.
- **Superfícies/arquivos/configurações afetadas:** backend de consulta, UI de carteira, regras de acesso.
- **Risco e plano B:** se a UI atrasar, entregar endpoint/relatório navegável com os mesmos campos.
- **Rollback ou reversão:** desativar rota/tela e manter dados carregados.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| staging -> visão carteira | dados publicados da SPEC-1-001 | cliente, responsável, forecasts, itens, leads, origem, atualização | perfil de usuário | consulta somente leitura | mostrar exceções sem ocultar erro |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | usuário executivo | exibir somente registros vinculados ao usuário/carteira | sem vínculo vai para exceção | escopo Fase 1 |
| RN-02 | gestor | exibir registros da alçada | alçada ausente exige admin | escopo Fase 1 |
| RN-03 | cliente com forecast/lead associado | exibir vínculo e origem | vínculo ambíguo aparece como exceção | matriz RT-003 |
| RN-04 | dado sem atualização | exibir data/fonte como ausente | não inferir atualização | critérios globais |

## Fluxo e regras

1. Usuário acessa a visão de carteira.
2. Sistema aplica perfil de acesso.
3. Usuário filtra por cliente, responsável, status, origem/campanha e atualização.
4. Sistema apresenta lista consolidada.
5. Usuário abre um cliente e visualiza forecasts, itens e leads associados.
6. Exceções aparecem separadas com motivo.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | cliente com forecast e lead | detalhe mostra vínculos e fontes | nenhum |
| Limite | cliente sem forecast | detalhe mostra cliente sem oportunidade associada | estado vazio útil |
| Falha | usuário sem perfil | acesso negado com mensagem clara | solicitar configuração |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-1-001 e `02-Escopo-Definitivo.md`, Fase 1.
2. **Alterar somente:** consultas, UI/rota de carteira e permissões de leitura.
3. **Não alterar:** regras de sinal, status comercial, fontes externas.
4. **Executar nesta ordem:** consulta base -> filtros -> detalhe -> perfil -> exceções -> demonstração.
5. **Parar e pedir validação quando:** regra de alçada não puder ser inferida de `vwTimes`/carteira.
6. **Estado válido ao parar:** dados continuam consultáveis por admin.

## Checklist de execução

- [ ] Consulta consolidada criada.
- [ ] Filtros principais implementados.
- [ ] Detalhe do cliente implementado.
- [ ] Perfil executivo/gestor/admin aplicado.
- [ ] Exceções de vínculo exibidas.

## Critérios de aceite

- [ ] **CA-1-004:** usuário autorizado filtra carteira e vê fonte/data de atualização.
- [ ] **CA-1-005:** detalhe do cliente mostra forecasts, itens e leads associados quando existem.
- [ ] **CA-1-006:** usuário sem perfil autorizado não acessa dados fora da alçada.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | consultar carteira sem dados publicados | abrir visão/endpoint | estado vazio ou erro controlado | captura/log |
| GREEN | consultar com fixtures vinculados | filtrar cliente e abrir detalhe | vínculos aparecem com fonte | captura/JSON |
| REFACTOR/REGRESSÃO | testar perfis e exceções | acessar como executivo/gestor/admin | alçada respeitada | captura/log |

**Dados/fixtures:** carga mínima com dois clientes, dois forecasts, um lead, um item e dois usuários.  
**Caminhos de erro obrigatórios:** sem dados, sem perfil, vínculo ambíguo.  
**Evidência exigida:** captura da visão ou resposta do endpoint com filtros e detalhe.

## Handoff e operação

- **Como demonstrar:** abrir carteira, filtrar por responsável e abrir cliente com forecast.
- **Como operar depois:** gestores validam amostra e apontam exceções de vínculo.
- **Como monitorar:** quantidade de registros sem vínculo e acessos por perfil.
- **Pendência conhecida:** alçadas definitivas podem ser refinadas na Fase 2/3.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T004 | Criar consulta consolidada e filtros de carteira | Sistemas Hardlink | SPEC-1-002 | filtros retornam dados vinculados | CA-1-004 / GREEN | captura/JSON | dados publicados | ☐ |
| F1-T005 | Criar detalhe do cliente com forecasts, itens e leads | Sistemas Hardlink | SPEC-1-002 | detalhe mostra vínculos e exceções | CA-1-005 / GREEN | captura/JSON | F1-T004 | ☐ |
| F1-T006 | Aplicar perfis de acesso executivo, gestor e admin | Sistemas Hardlink | SPEC-1-002 | alçada é respeitada | CA-1-006 / REGRESSÃO | log/captura | F1-T004 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|

