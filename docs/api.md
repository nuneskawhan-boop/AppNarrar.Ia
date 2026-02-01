# API — FastAPI (rascunho)

## Convenções
- Auth via **Supabase Auth** (Bearer JWT).
- IDs UUID.
- Tudo versionado em `/v1`.

## Endpoints principais
### Assets
- **POST /v1/assets/presign**
  - Cria URL para upload no S3.
- **POST /v1/assets**
  - Cria registro do asset após upload.
- **GET /v1/assets/{asset_id}**
  - Detalhes do asset e status de processamento.

### Projetos
- **POST /v1/projects**
  - Cria projeto com asset.
- **GET /v1/projects/{project_id}**
  - Dados completos (descrição, timeline, roteiro).

### Análise
- **POST /v1/projects/{project_id}/analyze**
  - Enfileira análise multimodal.

### Roteiro
- **POST /v1/projects/{project_id}/scripts**
  - Cria nova versão de roteiro.
- **GET /v1/projects/{project_id}/scripts/{script_id}**
  - Retorna versão específica.

### TTS
- **POST /v1/projects/{project_id}/tts**
  - Gera narração PT‑BR a partir de roteiro.

### Render final
- **POST /v1/projects/{project_id}/render**
  - Render final (MP4) com filtros/legendas.

## Modelos essenciais
### Asset
- id, owner_id, type (photo|video)
- source_url, duration, width, height, audio_present
- status (uploaded|processing|ready|failed)

### Project
- id, owner_id, asset_id
- title, status

### AnalysisResult
- description_certain
- description_seems
- description_uncertain
- summary
- timeline[]

### TimelineSegment
- start_ms, end_ms
- text_certain
- text_seems
- text_uncertain
- evidence[] (frame_refs, asr_refs)

### ScriptVersion
- id, project_id, version
- segments[] (com timestamps e texto)

### RenderJob
- id, project_id, script_id, status
- output_url

