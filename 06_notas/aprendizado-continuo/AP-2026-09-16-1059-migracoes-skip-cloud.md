# AP-2026-09-16-1059 — Migração de schema no Skip Cloud

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T001 / SPEC-1-001
- Sinal: novas coleções base do PocketBase precisam carregar explicitamente os campos de auditoria `created` e `updated`, as cinco regras de acesso e índices que incluam apenas campos declarados; a migração ordinal deve ser a próxima disponível.
- Evidência: `pocketbase/migrations/0002_model_carteira_staging.js`; aplicação observada da migração `0002_model_carteira_staging` no Skip v0.0.4; seis coleções e seus índices confirmados no backend.
- Regra reutilizável: ao criar schema novo no Skip Cloud, usar uma migração ordinal inédita, declarar `created`/`updated`, as cinco regras e os índices na mesma definição da coleção, e validar o schema aplicado antes de iniciar a carga de dados.
- Quando aplicar: toda nova coleção base ou migração de schema do projeto Carteiras Comerciais.
- Quando não aplicar: alterações de importação, validação de CSV ou consultas que não criem/modifiquem coleções.
- Confiança: alta — a regra foi aplicada e confirmada no backend pelo QA e pela inspeção independente do schema.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
