# SPEC-1-003 — Dicionário de dados e qualidade da Fase 1

**Fase:** 1  
**Status:** planejada  
**Dono:** Sistemas Hardlink / Consultoria  
**Origem no escopo:** RT-003, RT-011, Fase 1  
**Degrau da solução:** construção mínima — tornar campos, fontes, qualidade e exceções explícitos para sustentar fases seguintes.

## Contexto e decisões fechadas

- **Estado atual:** os arquivos possuem cabeçalhos ricos, mas fonte oficial, obrigatoriedade e qualidade ainda não estão formalizadas.
- **Estado desejado:** existe um dicionário consultável com entidades, campos, fonte, obrigatoriedade, regra de qualidade e uso no sistema.
- **Decisões já fechadas:** para execução sem bloqueio, a Fase 1 assume os CSVs/views como fonte operacional inicial.
- **Bloqueios:** nenhum para dicionário inicial.

## Resultado observável

Um dicionário da Fase 1 lista entidades e campos usados pela aplicação, mostra quais são obrigatórios, quais alimentam filtros/vínculos/perfis e quais exceções bloqueiam publicação.

## Limites e dependências

- **Inclui:** dicionário das cinco entidades, qualidade mínima, regras de exceção e matriz campo -> uso.
- **Fora de escopo:** governança corporativa completa, catálogo de dados global e integração com ferramenta externa de data catalog.
- **Entradas e pré-condições:** cabeçalhos dos CSVs anexados e decisões do escopo definitivo.
- **Saídas/artefatos:** `dicionario-dados-fase-1` ou tela equivalente, relatório de qualidade.
- **Dependências e responsáveis:** Sistemas Hardlink valida nomes/campos; gestor valida uso operacional.
- **Atores e permissões mínimas:** admin/sistemas editam; usuários consultam definições relevantes.
- **Superfícies/arquivos/configurações afetadas:** documentação/tabela de metadados, relatório de qualidade.
- **Risco e plano B:** se houver divergência de nomenclatura, registrar alias sem renomear fonte.
- **Rollback ou reversão:** versionar dicionário e manter versão anterior.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| CSV headers -> dicionário | cabeçalhos anexados | entidade, campo, tipo inferido, obrigatório, uso, qualidade | edição admin | versionamento por revisão | divergência vira exceção |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | campo usado como chave | marcar obrigatório | se ausente, bloqueia publicação | SPEC-1-001 |
| RN-02 | campo usado em filtro | registrar entidade e origem | se ausente, filtro fica indisponível | SPEC-1-002 |
| RN-03 | campo sensível/pessoal | marcar acesso restrito | não exportar em relatórios públicos | critérios globais |
| RN-04 | alias de sistema | registrar alias e nome canônico | não alterar dado original | análise crítica |

## Fluxo e regras

1. Ler cabeçalhos das cinco entidades.
2. Classificar campos por chave, exibição, filtro, vínculo, perfil ou auditoria.
3. Marcar obrigatoriedade e regra de qualidade.
4. Publicar dicionário para consulta.
5. Vincular relatório de carga aos itens do dicionário.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | campo obrigatório presente | campo aparece como válido | nenhum |
| Limite | campo opcional ausente | campo aparece como indisponível | funcionalidade dependente fica oculta |
| Falha | chave obrigatória ausente | publicação bloqueada | relatório orienta correção |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-1-001, SPEC-1-002 e seção 5 do escopo definitivo.
2. **Alterar somente:** dicionário, metadados e relatório de qualidade.
3. **Não alterar:** nomes nas fontes originais, regras de fases futuras.
4. **Executar nesta ordem:** mapear campos -> classificar uso -> definir qualidade -> publicar versão -> testar divergência.
5. **Parar e pedir validação quando:** campo novo mudar chave, permissão ou fonte oficial.
6. **Estado válido ao parar:** versão anterior do dicionário permanece disponível.

## Checklist de execução

- [ ] Entidades e campos mapeados.
- [ ] Campos obrigatórios e opcionais classificados.
- [ ] Campos sensíveis marcados.
- [ ] Regras de qualidade vinculadas à carga.
- [ ] Versão do dicionário publicada.

## Critérios de aceite

- [ ] **CA-1-007:** dicionário lista entidades, campos, origem, obrigatoriedade e uso.
- [ ] **CA-1-008:** relatório de qualidade referencia as regras do dicionário.
- [ ] **CA-1-009:** campo sensível ou pessoal possui restrição de acesso registrada.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | carga sem dicionário | abrir relatório de qualidade | não há regra explicável | captura/log |
| GREEN | publicar dicionário inicial | consultar campo obrigatório | regra e uso aparecem | captura/documento |
| REFACTOR/REGRESSÃO | simular coluna nova/ausente | rodar validação | divergência registrada | relatório |

**Dados/fixtures:** cabeçalhos dos cinco CSVs anexados.  
**Caminhos de erro obrigatórios:** campo obrigatório ausente, campo novo, dado sensível.  
**Evidência exigida:** dicionário publicado e relatório de qualidade vinculado.

## Handoff e operação

- **Como demonstrar:** abrir dicionário e explicar por que um campo bloqueia ou não uma carga.
- **Como operar depois:** revisar dicionário quando a fonte mudar.
- **Como monitorar:** número de campos divergentes e cargas rejeitadas por regra.
- **Pendência conhecida:** fonte oficial corporativa pode substituir o dicionário inicial.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T007 | Mapear campos e usos das cinco entidades | Sistemas Hardlink | SPEC-1-003 | dicionário lista origem, uso e obrigatoriedade | CA-1-007 / GREEN | documento/tela | cabeçalhos disponíveis | ☐ |
| F1-T008 | Vincular regras de qualidade ao relatório de carga | Sistemas Hardlink | SPEC-1-003 | relatório referencia regra do dicionário | CA-1-008 / GREEN | relatório | F1-T007 | ☐ |
| F1-T009 | Marcar campos sensíveis e restrições de acesso | Sistemas Hardlink | SPEC-1-003 | campos pessoais têm regra de acesso | CA-1-009 / REGRESSÃO | dicionário/captura | F1-T007 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|

