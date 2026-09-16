# Debug Summary — F1-T002 — 2026-09-16

**Problema:** upload dos cinco CSVs reais falhava com `Carga não concluída / Something went wrong`.

**Causa confirmada:** os arquivos somam aproximadamente 74.710 KB e o maior tem 38.461 KB. O Nginx rejeitou com HTTP 413 tanto uma carga sintética de 76.503.045 bytes quanto um arquivo individual de aproximadamente 38 MB, antes do hook.

**Correções:** transporte parcelado em multipart com blocos de aproximadamente 3 MiB; coleções temporárias `carteira_uploads` e `carteira_upload_chunks`; finalização atômica; parser rápido para CSV sem campos quoted; ordenação corrigida para `chunk_index`; `chunk_index` tornou-se opcional porque o PocketBase trata zero como blank em campo obrigatório; multipart convertido via `$filesystem.fileFromMultipart(...)`; limpeza dos chunks em rejeição.

**Verificação:** Skip v0.0.21 passou no QA oficial. Cinco fixtures pequenos foram enviados em cinco chunks e finalizados com HTTP 200, status `published` e cinco linhas válidas. A carga sintética de 76.503.045 bytes foi enviada em 27 chunks, todos HTTP 200; a finalização chegou à validação e retornou HTTP 400 estruturado `invalid_csv_headers` para cabeçalhos deliberadamente inválidos, sem 413 ou erro interno. Contas temporárias foram removidas.

**Gate:** aguardando teste humano com os cinco CSVs reais no preview. Nenhum CSV real foi publicado pelos testes automatizados.
