# AP-2026-09-16-1429 — Upload de CSV no Skip Cloud

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T002 / SPEC-1-001
- Sinal: serializar arquivos CSV inteiros em JSON pode exceder o limite do proxy antes da rota do hook, produzindo 413 e uma mensagem genérica no frontend.
- Evidência: reprodução com payload JSON grande retornou HTTP 413 sem log do hook; após migrar para multipart, `POST /backend/v1/carteira/import` retornou 200 para carga válida e 400 para cabeçalho ausente no Skip v0.0.7.
- Regra reutilizável: enviar arquivos do navegador como `FormData`/multipart e ler `e.findUploadedFiles(...)` no hook; manter conteúdo textual em JSON apenas para payloads pequenos e controlados.
- Quando aplicar: uploads de CSV ou outros arquivos cujo tamanho possa ultrapassar o limite do proxy.
- Quando não aplicar: metadados pequenos sem arquivo ou payloads JSON comprovadamente abaixo do limite operacional.
- Confiança: alta — causa reproduzida e correção exercitada no backend real.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
