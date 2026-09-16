# Debug Summary — F1-T002 — 2026-09-16

**Task e problema:** F1-T002; upload dos cinco CSVs exibiu `Carga não concluída / Something went wrong` em tentativas humanas.

**Reprodução:** o transporte JSON original excedeu o limite do proxy e retornou HTTP 413 antes da rota. Após a migração para multipart, o fluxo completo no navegador com cinco fixtures pequenos chegou ao backend, recebeu HTTP 200 e renderizou o relatório `Publicado`. Uma carga multipart grande anterior chegou à rota, mas consumiu cerca de 169 segundos no parser caractere a caractere.

**Causa raiz:** houve duas causas técnicas: (1) serialização JSON dos arquivos ultrapassava o proxy; (2) o parser CSV genérico caractere a caractere era lento para arquivos grandes sem campos quoted.

**Correção:** frontend envia `FormData`/`multipart/form-data`; hook lê `e.findUploadedFiles(...)` e `readerToString(...)`; erros 413 recebem mensagem específica; Skip v0.8 adiciona caminho rápido por linhas para CSVs sem aspas e mantém parser completo para CSVs quoted.

**Verificação automática:** QA oficial v0.8 passou. Multipart válido retornou 200 e publicou cinco linhas; multipart inválido retornou 400 com `missing_headers: ["E-mail"]`; fluxo real no navegador com cinco fixtures retornou 200 e mostrou o relatório publicado; contas temporárias foram removidas. O volume exato dos cinco CSVs reais ainda não foi validado pelo executor.

**Gate atual:** aguardando teste humano.
