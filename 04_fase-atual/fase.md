# Fase 1 — Tarefas gerais

**Status:** pronta para execução  
**Escopo:** base operacional, ingestão dos dados atuais, visão de carteira e dicionário de qualidade.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|
| F1-T001 | Modelar lote e staging das cinco entidades | Sistemas Hardlink | SPEC-1-001 | tabelas e chaves de carga existem | CA-1-001 / RED | schema/migration | banco operacional definido | schema criado sem importação parcial | ☐ |
| F1-T002 | Implementar validação de cabeçalho e importação CSV | Sistemas Hardlink | SPEC-1-001 | carga válida importa e carga inválida rejeita | CA-1-001, CA-1-002 / GREEN | relatório de carga | F1-T001 | importador rejeita erro crítico | ☐ |
| F1-T003 | Implementar relatório de qualidade e rollback por lote | Sistemas Hardlink | SPEC-1-001 | rollback remove apenas o lote selecionado | CA-1-003 / REGRESSÃO | log + consulta | F1-T002 | rollback demonstrado | ☐ |
| F1-T004 | Criar consulta consolidada e filtros de carteira | Sistemas Hardlink | SPEC-1-002 | filtros retornam dados vinculados | CA-1-004 / GREEN | captura/JSON | dados publicados | consulta disponível para admin | ☐ |
| F1-T005 | Criar detalhe do cliente com forecasts, itens e leads | Sistemas Hardlink | SPEC-1-002 | detalhe mostra vínculos e exceções | CA-1-005 / GREEN | captura/JSON | F1-T004 | detalhe mostra estados vazio e com vínculo | ☐ |
| F1-T006 | Aplicar perfis de acesso executivo, gestor e admin | Sistemas Hardlink | SPEC-1-002 | alçada é respeitada | CA-1-006 / REGRESSÃO | log/captura | F1-T004 | usuário sem perfil é bloqueado | ☐ |
| F1-T007 | Mapear campos e usos das cinco entidades | Sistemas Hardlink | SPEC-1-003 | dicionário lista origem, uso e obrigatoriedade | CA-1-007 / GREEN | documento/tela | cabeçalhos disponíveis | dicionário versão inicial publicado | ☐ |
| F1-T008 | Vincular regras de qualidade ao relatório de carga | Sistemas Hardlink | SPEC-1-003 | relatório referencia regra do dicionário | CA-1-008 / GREEN | relatório | F1-T007 | erros apontam regra do dicionário | ☐ |
| F1-T009 | Marcar campos sensíveis e restrições de acesso | Sistemas Hardlink | SPEC-1-003 | campos pessoais têm regra de acesso | CA-1-009 / REGRESSÃO | dicionário/captura | F1-T007 | campos pessoais não saem em export público | ☐ |
