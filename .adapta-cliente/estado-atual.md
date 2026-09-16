# Estado atual — Adapta Cliente

- task_id: F1-T002
- champion: Sistemas Hardlink
- spec: 04_fase-atual/specs/spec-fase-1-001-ingestao-dados-carteira.md
- etapa: em_correcao
- autorizacao_implementacao: confirmada em 2026-09-16T13:27:00-03:00 — "Pode implementar a F1-T002 conforme o plano analisado."
- teste_humano: falhou em 2026-09-16T15:17:00-03:00 — "Carga não concluída / Something went wrong"; arquivos informados somam 74.710 KB e o maior tem 38.461 KB
- verificacao_automatica: QA v0.0.15 passou; HTTP 413 foi reproduzido no Nginx para 76.503.047 bytes e para um arquivo de 38.384.064 bytes; teste parcelado pequeno chegou ao backend, mas falhou ao salvar chunks; finalize registrou ordenação inválida `chunk_index ASC`
- aprendizado: capturado:06_notas/aprendizado-continuo/AP-2026-09-16-1429-upload-multipart.md
- ultima_acao: diagnóstico consolidado em artifacts/f1-t002-debug-status.md; upload parcelado preparado, porém armazenamento das partes ainda não validado
- proxima_acao: corrigir ordenação e persistência do campo `chunk_file`, repetir teste pequeno e só então validar volume real
- atualizado_em: 2026-09-16T15:46:00-03:00
