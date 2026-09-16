# Changelog

## Inicial

- Pasta operacional criada a partir do handoff da consultoria.

## 2026-09-16

- 2026-09-16 · Sistemas Hardlink · Task F1-T001 concluída: migração `0002_model_carteira_staging` aplicada no Skip v0.0.4; schema das seis coleções validado, QA automático aprovado e teste humano confirmado.
- 2026-09-16 · Sistemas Hardlink · DEBUG task F1-T002: upload retornava erro genérico porque o corpo JSON excedia o limite do proxy → transporte migrado para multipart, correção aplicada no Skip v0.0.7 e reteste automatizado aprovado; aguardando novo teste humano.
- 2026-09-16 · Sistemas Hardlink · DEBUG task F1-T002: parser CSV caractere a caractere era lento em arquivos grandes → caminho rápido por linhas aplicado no Skip v0.0.8; QA e fluxo de navegador com fixtures passaram; aguardando novo teste humano.
- 2026-09-16 · Sistemas Hardlink · F1-T002: upload parcelado concluído no Skip v0.0.21; corrigidos limite do proxy, ordenação de chunks, índice zero e conversão multipart; QA passou, cinco fixtures foram publicados e volume sintético de 76.503.045 bytes atravessou 27 chunks sem 413; aguardando teste humano com os CSVs reais.
