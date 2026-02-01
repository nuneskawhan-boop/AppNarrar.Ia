# NarrAIVídeo (MVP) — Visão técnica

Aplicativo web para criação de vídeos narrados a partir de foto/vídeo com **leitura fiel** do conteúdo, geração de roteiro editável, narração em PT‑BR e exportação final em MP4.

## Objetivos do MVP
- Upload de **foto ou vídeo**.
- **Leitura fiel** (sem inventar detalhes) com separação de certeza vs. hipótese vs. não confirmável.
- Geração de **descrição**, **resumo**, **linha do tempo** e **roteiro narrável** com timestamps.
- **Narração em áudio PT‑BR** sincronizada e opção de legendas.
- **Edição simples** (trim, filtros básicos, texto na tela, ajuste de voz).
- Exportação **MP4** com narração e legendas opcionais.

## Stack definida
- **Frontend**: Next.js (App Router) + TypeScript + Tailwind.
- **Backend API**: FastAPI (Python).
- **DB**: PostgreSQL (Supabase ou RDS).
- **Storage**: S3 compatível (AWS S3 ou equivalente).
- **Fila**: Redis + Celery.
- **Worker de mídia**: Python + FFmpeg.
- **Auth**: **Supabase Auth** (e‑mail + Google).

### Por que Supabase Auth?
- Reduz tempo de integração (MVP), com **OAuth Google + e‑mail** prontos.
- SDK bem suportado em Next.js + Postgres nativo.
- Alinha com uso opcional de **Supabase Postgres** no MVP.

> Auth.js é ótimo quando há infraestrutura própria e maior controle; para **velocidade** e **menor esforço**, Supabase Auth é mais adequado ao MVP.

## Princípios críticos (anti-alucinação)
- Separar sempre: **“O que é certo”**, **“O que parece”**, **“Não dá para confirmar”**.
- Nunca inferir **marca, local, identidade, idade, etnia, religião** etc.
- Se o conteúdo estiver **fora de quadro, desfocado, sem áudio**, declarar explicitamente.
- “Modo Criativo” é **funcionalidade separada** e fora do MVP.

## Visão geral da arquitetura
```
[Next.js] -> [FastAPI] -> [Postgres]
     |            |          |
     |            |          +-- metadados, roteiros, timelines
     |            +-- orquestra jobs (Celery)
     |
     +-- Upload -> [S3]
                    |
                 [Workers: FFmpeg + IA]
                    |
                 Outputs (áudio, legendas, MP4)
```

### Fluxo principal (foto/vídeo)
1. **Upload** via frontend para **S3** (presigned URL).
2. API registra o asset e enfileira job **análise multimodal**.
3. Worker extrai **frames, áudio e metadata** (FFmpeg).
4. Pipelines de IA geram:
   - **Descrição fiel**
   - **Resumo curto**
   - **Linha do tempo por timestamps**
   - **Roteiro narrável com timestamps**
   - **Transcrição ASR** (se houver áudio)
5. Usuário edita roteiro; API enfileira **TTS**.
6. Worker gera **áudio PT‑BR** e **legendas** (SRT/VTT).
7. Worker renderiza **MP4 final** com mix de trilhas e filtros.

## Fases para vozes brasileiras (PT‑BR)
### Fase 1 — MVP (curto prazo)
- **TTS via provedor comercial** com vozes PT‑BR de alta qualidade.
- Foco em baixa latência, custo previsível, e múltiplas vozes.
- Ex.: provedores de TTS com PT‑BR natural (avaliar preço e qualidade).

### Fase 2 — Evolução (médio prazo)
- **Custom voice** (com consentimento explícito) e clonagem controlada.
- Gestão de **consentimento**, auditoria e revogação.
- Modelo/fine‑tuning próprio ou third‑party dedicado.

## Segurança, consentimento e compliance
- Fluxo de **consentimento explícito** para voz do usuário.
- Armazenar **aceite** (timestamp, IP, versão do termo).
- Auditoria de uso e possibilidade de remoção definitiva.

## Estrutura inicial sugerida do repositório
```
/ (monorepo)
  /apps/web        # Next.js
  /apps/api        # FastAPI
  /apps/worker     # Celery + FFmpeg
  /packages/shared # Tipos e utilitários
  /infra           # IaC / docker / k8s
  /docs            # documentação
```

## Entregáveis do MVP
- Upload e análise multimodal com anti‑alucinação.
- Edição simples do roteiro e geração de TTS.
- Exportação MP4 com legendas opcionais.
- UI simples e funcional (timeline + editor).

## Próximos passos
- Refinar **modelo de dados** e **APIs** (ver docs).
- Definir provedor TTS/ASR final baseado em qualidade e custo.
- Desenhar jornada do usuário e telas principais.
