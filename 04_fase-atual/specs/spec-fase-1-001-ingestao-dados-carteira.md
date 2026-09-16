# SPEC-1-001 — Ingestão de dados da carteira atual

**Fase:** 1  
**Status:** planejada  
**Dono:** Sistemas Hardlink  
**Origem no escopo:** RT-002, RT-003, Fase 1  
**Degrau da solução:** construção mínima — carregar as entidades iniciais sem acoplar ao CRM novo.

## Contexto e decisões fechadas

- **Estado atual:** a Hardlink disponibilizou amostras CSV de clientes, forecasts, itens, leads e times.
- **Estado desejado:** o ambiente da aplicação possui tabelas/staging para carregar esses arquivos, validar colunas obrigatórias e registrar origem, data e contagem.
- **Decisões já fechadas:** a Fase 1 usa leitura controlada; não escreve no CRM, Power BI ou fontes externas.
- **Bloqueios:** nenhum para execução inicial com CSVs anexados.

## Resultado observável

Ao final, uma carga da Fase 1 importa os cinco arquivos de dados iniciais, produz um relatório de qualidade e deixa os registros consultáveis para a visão de carteira.

## Limites e dependências

- **Inclui:** ingestão de `vwClientesPbi`, `vwForecasts`, `vwForecastsItem`, `vwLeadsPbi` e `vwTimes`.
- **Fora de escopo:** integração em tempo real, escrita em sistemas de origem, CRM novo, WhatsApp, Outlook e RD Station via API.
- **Entradas e pré-condições:** arquivos CSV com cabeçalhos iguais aos anexados.
- **Saídas/artefatos:** tabelas/staging carregadas, log de carga, relatório de contagem e erros.
- **Dependências e responsáveis:** Sistemas Hardlink fornece arquivos e ambiente; executor implementa carga.
- **Atores e permissões mínimas:** leitura dos arquivos e escrita no banco operacional da aplicação.
- **Superfícies/arquivos/configurações afetadas:** camada de banco, scripts/serviço de importação, configuração de diretórios de entrada.
- **Risco e plano B:** se houver coluna ausente, rejeitar arquivo e gerar erro claro.
- **Rollback ou reversão:** apagar lote importado por `load_id` sem afetar cargas anteriores.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| CSV clientes -> staging clientes | `vwClientesPbi` | `Codigo`, `Nome`, `Inativo`, `Usuario`, `Carteira`, `CNPJ`, `Estado`, `Cidade`, `E-mail`, `Segmento` | leitura de arquivo | idempotente por `load_id` + `Codigo` | rejeitar lote se cabeçalho obrigatório faltar |
| CSV forecasts -> staging forecasts | `vwForecasts` | `Código`, `Status`, `Andamento`, `Empresa`, `Usuario`, `Criação`, `Fechamento`, `VendedorID`, `Cód. Cliente`, `Margem`, `Venda`, `Vencimento`, `Origem do Negocio`, `Pré-Vendas`, `Vendedor` | leitura de arquivo | idempotente por `load_id` + `Código` | registrar linhas inválidas |
| CSV itens -> staging itens | `vwForecastsItem` | `forecast_id`, `item_id`, `produto`, `categoria`, `departamento`, `custo`, `margem`, `valor_venda`, `total` | leitura de arquivo | idempotente por `load_id` + `forecast_id` + `item_id` | registrar órfãos |
| CSV leads -> staging leads | `vwLeadsPbi` | `Código`, `Criado em`, `Status`, `ID Forecast`, `Fonte do lead`, `Campanha`, `ID do responsável` | leitura de arquivo | idempotente por `load_id` + `Código` | registrar duplicidades |
| CSV times -> staging usuarios | `vwTimes` | `VendedorID`, `ConsultantName`, `Email`, `UserActive`, `Leader` | leitura de arquivo | idempotente por `load_id` + `VendedorID` | registrar usuários sem e-mail |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | arquivo sem coluna obrigatória | rejeitar lote inteiro | nenhuma | escopo Fase 1 |
| RN-02 | linha com chave vazia | registrar erro e não publicar a linha | manter demais linhas válidas | escopo Fase 1 |
| RN-03 | nova carga do mesmo arquivo | criar novo `load_id` e preservar histórico | rollback por lote | escopo Fase 1 |
| RN-04 | dados pessoais de contato | manter acesso restrito por perfil | não exportar fora do sistema | critérios globais |

## Fluxo e regras

1. Receber os cinco arquivos CSV no local configurado.
2. Validar cabeçalho, codificação e colunas obrigatórias.
3. Criar um `load_id` para o conjunto da carga.
4. Importar linhas válidas para staging.
5. Gerar relatório com total de linhas, linhas válidas, erros, órfãos e duplicidades.
6. Publicar somente cargas sem erro crítico.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | cinco CSVs válidos | carga publicada com contagens | registrar `load_id` |
| Limite | itens sem forecast correspondente | linha marcada como órfã | manter para auditoria, sem associar |
| Falha | cabeçalho obrigatório ausente | lote rejeitado | relatório informa coluna ausente |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** `03-Projeto/02-Escopo-Definitivo.md`, seção 5 e Fase 1.
2. **Alterar somente:** estrutura de carga, staging, validação e relatório de ingestão.
3. **Não alterar:** fontes originais, CRM, Power BI, SPECs de fases futuras.
4. **Executar nesta ordem:** modelar lote -> validar cabeçalhos -> importar -> gerar relatório -> testar rollback.
5. **Parar e pedir validação quando:** surgir campo obrigatório ausente que não esteja mapeado nesta SPEC.
6. **Estado válido ao parar:** nenhuma carga parcialmente publicada.

## Checklist de execução

- [ ] Modelo de lote e staging criado.
- [ ] Validação de cabeçalhos obrigatórios implementada.
- [ ] Importação dos cinco arquivos implementada.
- [ ] Relatório de qualidade gerado.
- [ ] Rollback por `load_id` demonstrado.

## Critérios de aceite

- [ ] **CA-1-001:** carga válida importa os cinco arquivos e mostra contagem por entidade.
- [ ] **CA-1-002:** arquivo com cabeçalho obrigatório ausente é rejeitado com mensagem clara.
- [ ] **CA-1-003:** rollback por `load_id` remove a carga sem afetar cargas anteriores.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | executar carga sem tabelas/staging | rodar importador com fixtures | falha por estrutura ausente | log de teste |
| GREEN | executar carga com fixtures válidos | rodar importador com cinco CSVs | contagens e relatório gerados | relatório de carga |
| REFACTOR/REGRESSÃO | CSV sem coluna obrigatória e rollback | rodar carga inválida e rollback | rejeição clara e estado limpo | log + consulta |

**Dados/fixtures:** amostras pequenas derivadas dos cinco CSVs anexados.  
**Caminhos de erro obrigatórios:** arquivo ausente, coluna ausente, chave vazia, item órfão, rollback.  
**Evidência exigida:** relatório de carga e resultado dos testes.

## Handoff e operação

- **Como demonstrar:** carregar fixtures e abrir relatório com contagens.
- **Como operar depois:** Sistemas Hardlink disponibiliza novos arquivos e acompanha erros.
- **Como monitorar:** contagem por entidade, erros por arquivo e data da última carga.
- **Pendência conhecida:** trocar CSV por views/API quando a Hardlink disponibilizar acesso contínuo.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T001 | Modelar lote e staging das cinco entidades | Sistemas Hardlink | SPEC-1-001 | tabelas e chaves de carga existem | CA-1-001 / RED | schema/migration | banco operacional definido | ☐ |
| F1-T002 | Implementar validação de cabeçalho e importação CSV | Sistemas Hardlink | SPEC-1-001 | carga válida importa e carga inválida rejeita | CA-1-001, CA-1-002 / GREEN | relatório de carga | F1-T001 | ☐ |
| F1-T003 | Implementar relatório de qualidade e rollback por lote | Sistemas Hardlink | SPEC-1-001 | rollback remove apenas o lote selecionado | CA-1-003 / REGRESSÃO | log + consulta | F1-T002 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|

