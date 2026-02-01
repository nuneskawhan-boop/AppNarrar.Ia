# Arquitetura detalhada — NarrAIVídeo

## Componentes principais
- **Web (Next.js + Tailwind)**: upload, timeline, editor de roteiro, preview e exportação.
- **API (FastAPI)**: autenticação, orquestração de jobs, validação, geração de presigned URLs, controle de projetos.
- **Worker (Celery + FFmpeg)**: extração de mídia, renderização, mux de áudio, filtros, geração de legendas.
- **IA multimodal**: visão + áudio (ASR) + sumarização + geração de roteiro narrável.
- **PostgreSQL**: dados de usuários, projetos, assets, roteiros, jobs e versões.
- **Storage S3**: assets brutos e outputs (áudios, mp4, legendas).

## Fluxos críticos
### 1) Upload e análise
1. Web solicita presigned URL.
2. Upload direto para S3.
3. API cria registro do asset e enfileira `analyze_asset`.
4. Worker executa:
   - **FFmpeg**: extrai frames e áudio, normaliza taxa de amostragem.
   - **ASR (PT‑BR)**: se houver áudio.
   - **Visão**: descrição de frames + detecção de eventos.
   - **Fusão multimodal**: consolida em timeline.
5. API registra descrição, resumo, timeline e roteiro inicial.

### 2) Edição e geração de narração
1. Usuário edita roteiro (com timestamps).
2. Web envia `script_version`.
3. API enfileira `tts_render`.
4. Worker gera **áudio PT‑BR** e **SRT**.

### 3) Render final
1. Web solicita render final com parâmetros (trim, filtros, texto, legendas).
2. API enfileira `render_final`.
3. Worker usa FFmpeg para aplicar filtros, mix de áudio e exporta MP4.

## Anti-alucinação (camadas)
- **Camada 1 – Extração**: coleta de evidências explícitas (frames e ASR).
- **Camada 2 – Consolidação**: classifica evidências em:
  - **Certo** (observável/necessariamente audível).
  - **Parece** (baixa confiança/ambiguidade).
  - **Não confirmável** (não visível/inaudível/fora de quadro).
- **Camada 3 – Resposta**: descrição final e timeline sempre com marcações explícitas.

## Timeline e roteiro narrável
- Timeline formada por **segmentos** com timestamps precisos.
- Roteiro narrável segue a timeline, com texto editável e controle de pausas.
- Cada segmento mantém **evidências** (frames + trechos ASR) para auditoria.

## Dados e versões
- Todo roteiro possui **versões**.
- Outputs de TTS e render final ficam ligados à versão do roteiro.
- Reprocessamentos mantêm histórico para rastreabilidade.

## Escalabilidade e custos
- Jobs longos executam em **workers** isolados.
- **Auto‑scaling** por fila (Redis + Celery + autoscaling).
- Separação de filas: `analysis`, `tts`, `render`.
- **Cache** de resultados por hash do asset + versão do roteiro.

## Observabilidade
- Logs por job + tracing (correlation id).
- Métricas: duração por etapa, custo por minuto de mídia, falhas por tipo.

## Segurança
- Presigned URLs com expiração curta.
- ACL mínima no S3.
- Consentimento explícito para voz do usuário (com registro imutável).

