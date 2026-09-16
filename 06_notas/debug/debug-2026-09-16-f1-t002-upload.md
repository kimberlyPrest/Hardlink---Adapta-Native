# Debug Summary — F1-T002 — 2026-09-16

**Task e problema:** F1-T002; upload dos cinco CSVs exibiu `Carga não concluída / Something went wrong` em duas tentativas humanas.

**Reprodução:** o cliente enviava os cinco arquivos serializados em JSON (`fileName` + conteúdo textual). Testes com payloads grandes retornaram HTTP 413 do proxy antes da rota; os logs não registravam o hook. A reprodução multipart com curl, após a correção, retornou 200 para carga válida e 400 para cabeçalho ausente.

**Causa raiz:** o transporte JSON expandia o conteúdo dos cinco CSVs no corpo da requisição e ultrapassava o limite do proxy. Como a requisição era rejeitada antes do `routerAdd`, o frontend recebia a mensagem genérica do SDK (`Something went wrong`) e não havia relatório de carga.

**Correção:** frontend passou a enviar `FormData`/`multipart/form-data` com os cinco `File` reais; hook passou a ler `e.findUploadedFiles(...)` e `readerToString(...)`, preservando validação, carga atômica e relatório. Erros HTTP 413 agora recebem mensagem específica na interface. O formato JSON anterior continua aceito no hook para compatibilidade.

**Verificação automática:** QA oficial passou no Skip v0.0.7 (setup, análise estática, build, integrações e testes). Multipart válido retornou 200 e publicou 5 linhas; multipart inválido retornou 400 com `missing_headers: ["E-mail"]`; contas temporárias foram removidas. Logs confirmam POSTs 200/400 no endpoint após a correção. A medição de payload grande foi interrompida pelo limite do executor, mas não é necessária para o aceite: a causa 413 já foi reproduzida e o novo transporte evita a serialização JSON.

**Gate atual:** aguardando teste humano.
