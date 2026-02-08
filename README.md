ï»¿ï»¿ï»¿# AI Network Guardian
**Plataforma de IA para prever, diagnosticar e resolver falhas de conectividade, reduzindo custos operacionais e melhorando SLA.**

> O que este projeto faz: monitora telemetria e eventos de rede, detecta anomalias, classifica causa provÃ¡vel (massiva vs individual), prevÃª risco de falha (2h/6h/24h), recomenda ou executa aÃ§Ãµes (reboot, abrir incidente, agendar ou cancelar visita) e exibe tudo em um dashboard com mÃ©tricas de economia operacional.

---

## SumÃ¡rio
- [VisÃ£o geral](#visao-geral)
- [Problema que resolve](#problema-que-resolve)
- [Status do projeto](#status-do-projeto)
- [Principais funcionalidades](#principais-funcionalidades)
- [Arquitetura](#arquitetura)
- [Fluxogramas](#fluxogramas)
- [Modelos de IA](#modelos-de-ia)
- [Decision Engine](#decision-engine)
- [Estrutura do repositÃ³rio](#estrutura-do-repositorio)
- [Banco de dados](#banco-de-dados)
- [API](#api)
- [Dashboard](#dashboard)
- [MÃ©tricas e economia](#metricas-e-economia)
- [Outras alavancas de economia](#outras-alavancas-de-economia)
- [Requisitos e dependÃªncias](#requisitos-e-dependencias)
- [ConfiguraÃ§Ã£o](#configuracao)
- [Como rodar localmente](#como-rodar-localmente)
- [Testes e qualidade](#testes-e-qualidade)
- [Observabilidade](#observabilidade)
- [SeguranÃ§a, privacidade e LGPD](#seguranca-privacidade-e-lgpd)
- [GovernanÃ§a de modelos](#governanca-de-modelos)
- [Deploy e operaÃ§Ã£o](#deploy-e-operacao)
- [Roadmap](#roadmap)
- [ContribuiÃ§Ã£o](#contribuicao)
- [Suporte e contato](#suporte-e-contato)
- [LicenÃ§a](#licenca)

---

## Visao geral

No fluxo tradicional de ISP:

1. Cliente perde sinal
2. Cliente abre chamado
3. Agenda tÃ©cnico
4. Alto custo operacional

O **AI Network Guardian** antecipa o problema:

- Detecta degradaÃ§Ã£o antes da queda completa.
- Separa queda massiva de problema individual.
- Executa autocorreÃ§Ã£o quando possÃ­vel.
- Comunica o cliente automaticamente.
- Agenda visita apenas quando necessÃ¡rio.
- Cancela visitas quando o problema normaliza.
- Mede economia e melhora de SLA e MTTR.

---

## Problema que resolve

- Reduz visitas tÃ©cnicas desnecessÃ¡rias.
- Diminui MTTR e melhora SLA percebido.
- Aumenta capacidade do NOC sem inflar equipe.
- Cria previsibilidade com risco de falha por janela.

---

## Status do projeto

- Em desenvolvimento.
- Escopo atual: POC funcional com pipeline de dados, modelos iniciais e dashboard.
- PrÃ³ximo marco: integraÃ§Ã£o com sistemas reais de OSS/BSS.

---

## Principais funcionalidades

### Monitoramento e inteligÃªncia

- IngestÃ£o de dados em batch ou near real time.
- Fontes: telemetria de sinal, status de ONU/modem, eventos de rede, histÃ³rico de chamados/OS, contexto de regiÃ£o e vizinhos, clima.
- Feature Store com agregaÃ§Ãµes em janelas de 5, 15 e 60 minutos, alÃ©m de histÃ³rico diÃ¡rio e semanal.
- DetecÃ§Ã£o de anomalias.
- ClassificaÃ§Ã£o de causa provÃ¡vel.
- PrevisÃ£o de risco de falha.

### AÃ§Ãµes automÃ¡ticas

- Reboot controlado quando elegÃ­vel.
- Abertura de incidente massivo.
- Pausa de agendamentos em regiÃµes afetadas.
- Agendamento inteligente.
- NotificaÃ§Ã£o ao cliente.
- ProteÃ§Ã£o de Cliente em Janela de AtivaÃ§Ã£o/MudanÃ§a (CTO).

### OperaÃ§Ã£o e gestÃ£o

Dashboard com:

- Fila de clientes em risco.
- Heatmap por regiÃ£o.
- Incidentes ativos.
- Auditoria das decisÃµes da IA.
- Economia estimada.
- MÃ©tricas de SLA e MTTR.

---

## Arquitetura

Stack sugerida:

- Data e ETL: Python
- Stream: Redis Streams ou RabbitMQ
- Banco: PostgreSQL ou MySQL
- API: FastAPI ou Laravel
- ML: scikit-learn, LightGBM ou XGBoost
- Frontend: React + Tailwind

---

## Fluxogramas

### 1) Fluxo geral

```mermaid
flowchart TB
  A[Ingestao de dados] --> B[Camada Raw]
  B --> C[Camada Silver]
  C --> D[Feature Store]
  D --> E[ML Core]
  E --> F[Decision Engine]
  F --> G[Acoes automaticas]
  F --> H[Notificacoes]
  E --> I[Dashboard]
  F --> I
  G --> J[Feedback Loop]
  J --> D
```

---

### 2) Fluxo de decisao

```mermaid
flowchart TD
  S[Evento cliente offline] --> F1[Calcular features]
  F1 --> M1[Modelo anomalia]
  F1 --> M2[Modelo classificacao]
  F1 --> M3[Modelo risco]

  M2 -->|Massiva alta| X1[Verificar cluster vizinhos]
  X1 -->|Cluster alto| A1[Abrir incidente massivo]
  X1 -->|Cluster baixo| I1[Tratar individual]

  M2 -->|Individual| I1

  I1 --> R1{Pode autocorrigir?}
  R1 -->|Sim| B1[Reboot controlado]
  R1 -->|Nao| T0{Tecnico em ativacao/manutencao na CTO?}
  T0 -->|Sim| L1[Acionar tecnico no local]
  T0 -->|Nao| T1[Agendar tecnico]

  B1 --> V1{Estabilizou?}
  V1 -->|Sim| C1[Fechar caso]
  V1 -->|Nao| T0
```

---

### 2.1) Fluxo de economia e prevenção

```mermaid
flowchart TD
  A[Evento de rede ou cliente] --> B{Reabertura de OS <= X dias?}
  B -->|Sim| B1[Priorizar RCA e validação técnica]
  B -->|Não| C{Qualidade pós-ativação <= X horas?}
  C -->|Ruim| C1[Retorno preventivo antes da reclamação]
  C -->|Ok| D{Linha estabilizou >= X min?}
  D -->|Sim| D1[Cancelar OS e notificar cliente]
  D -->|Não| E{Risco x impacto - SLA/cliente crítico?}
  E -->|Alto| E1[Prioridade alta na fila]
  E -->|Normal| E2[Prioridade normal]
  E1 --> F{CTO/OLT reincidente?}
  E2 --> F
  F -->|Sim| F1[Acionar manutenção preventiva]
  F -->|Não| G[Fluxo padrão de atendimento]
```

---


---










---

### 3) Sequencia monitoramento

```mermaid
sequenceDiagram
  participant COL as Collector
  participant FEAT as FeatureStore
  participant ML as MLCore
  participant DEC as DecisionEngine
  participant ACT as ActionService
  participant AUD as AuditLog
  participant UI as Dashboard

  COL->>FEAT: envia telemetria
  FEAT->>ML: envia features
  ML->>DEC: scores e previsoes
  DEC->>AUD: registra decisao
  DEC->>ACT: executa acao
  ACT->>AUD: resultado
  AUD->>UI: atualiza dashboard
```

---

### 4) Arquitetura de componentes

```mermaid
flowchart LR
  subgraph DataSources
    S1[Telemetria]
    S2[Eventos rede]
    S3[Chamados]
    S4[Clima]
  end

  subgraph Ingestion
    C1[Collectors]
    Q1[Queue]
  end

  subgraph Storage
    R[Raw]
    SI[Silver]
    FS[Feature Store]
    DB1[DB Operacional]
    DB2[DB Analytics]
  end

  subgraph Intelligence
    ML[ML Core]
    DR[Drift Monitor]
  end

  subgraph Decision
    DE[Decision Engine]
    AS[Action Service]
    NS[Notification Service]
    AL[Audit Log]
  end

  subgraph Product
    API[API]
    UI[Dashboard]
  end

  S1 --> C1
  S2 --> C1
  S3 --> C1
  S4 --> C1
  C1 --> Q1
  Q1 --> R
  R --> SI
  SI --> FS
  FS --> ML
  ML --> DE
  DE --> AS
  DE --> NS
  DE --> AL
  API --> DB1
  API --> DB2
  UI --> API
  AL --> DB2
  DR --> ML
```

---

## Modelos de IA

### DetecÃ§Ã£o de anomalia

- Isolation Forest.
- One Class SVM.

SaÃ­da:

```
anomaly_score
```

### ClassificaÃ§Ã£o

- LightGBM ou XGBoost.

Classes:

- MASSIVE_OUTAGE
- CUSTOMER_PREMISES
- EQUIPMENT_FAIL
- EXTERNAL_NETWORK
- INTERMITTENT

### PrevisÃ£o de risco

SaÃ­das:

```
risk_2h
risk_6h
risk_24h
```

---

## Decision Engine

Decide baseado em:

- Resultados dos modelos.
- Contexto do cliente.
- Regras de seguranÃ§a.
- Custo esperado.

Exemplo:

- Massiva detectada: abrir incidente.
- Individual: tentar reboot.
- Queda individual na mesma CTO durante janela de ativaÃ§Ã£o/manutenÃ§Ã£o: acionar tÃ©cnico no local para corrigir antes de sair.
- Estabilizou: cancelar OS.
- Persistiu: agendar visita.

Todas as decisÃµes sÃ£o auditadas.

Regra especÃ­fica:
- ProteÃ§Ã£o de Cliente em Janela de AtivaÃ§Ã£o/MudanÃ§a (CTO): se um cliente da mesma CTO cair dentro do intervalo de inÃ­cio/fim de uma ativaÃ§Ã£o ou manutenÃ§Ã£o, o tÃ©cnico responsÃ¡vel deve validar e corrigir no local.

---

## Estrutura do repositorio

```
ai-network-guardian/
ââ apps/
â  ââ api/
â  ââ dashboard/
ââ services/
â  ââ collectors/
â  ââ feature-store/
â  ââ ml-core/
â  ââ decision-engine/
â  ââ action-service/
â  ââ notification-service/
ââ infra/
ââ docs/
ââ notebooks/
ââ README.md
```

---

## Banco de dados

Tabelas principais:

- telemetry_raw
- events_raw
- customers
- service_orders
- incidents
- features_timeseries
- ml_predictions
- decisions
- actions
- notifications

---

## API

Endpoints:

```
GET /health
GET /customers/:id/timeline
GET /risk
GET /incidents/active
POST /decision/simulate
POST /decision/execute
GET /metrics/kpis
```

---

## Dashboard

Telas:

- NOC Overview.
- Fila de risco.
- DossiÃª do cliente.
- KPIs e economia.
- Explainability.

---

## Metricas e economia

KPIs:

- Precision e Recall.
- MTTR.
- SLA.
- OS evitadas.

Economia:

```
economia = OS_evitadas * custo_medio_visita
```

---

## Outras alavancas de economia

- DetecÃ§Ã£o de retorno de OS: se um cliente reabre OS em atÃ© X dias, prioriza anÃ¡lise de causa raiz e evita visitas repetidas.
- Prioridade inteligente de fila: ordena visitas por risco e impacto (clientes crÃ­ticos, empresas, SLA premium).
- PrÃ©-diagnÃ³stico antes da visita: testes remotos automÃ¡ticos e checklist do tÃ©cnico com provÃ¡vel causa.
- Cancelamento automÃ¡tico de OS: se a linha estabilizar por X minutos, cancela e notifica o cliente.
- Controle de reincidÃªncia por CTO/OLT: identifica CTOs que caem com frequÃªncia e indica manutenÃ§Ã£o preventiva.
- Monitoramento de qualidade pÃ³s-ativaÃ§Ã£o: sinal ruim nas primeiras X horas aciona retorno preventivo do tÃ©cnico.

---

## Requisitos e dependencias

- Docker.
- Python 3.11.
- Node 18.
- Banco de dados: PostgreSQL ou MySQL.

---

## Configuracao

Crie um `.env` na raiz com os valores necessÃ¡rios. Exemplo mÃ­nimo:

```
ENV=dev
DB_HOST=localhost
DB_PORT=5432
DB_NAME=ai_network_guardian
DB_USER=postgres
DB_PASSWORD=postgres
REDIS_URL=redis://localhost:6379
RABBITMQ_URL=amqp://guest:guest@localhost:5672
MODEL_REGISTRY_PATH=/models
NOTIFICATION_PROVIDER=mock
```

---

## Como rodar localmente

Subir infra:

```
docker compose up -d
```

API:

```
cd apps/api
pip install -r requirements.txt
uvicorn main:app --reload
```

Dashboard:

```
cd apps/dashboard
npm install
npm run dev
```

---

## Testes e qualidade

- Testes unitÃ¡rios e de integraÃ§Ã£o com `pytest`.
- Linters e formataÃ§Ã£o: `ruff`, `black` e `isort`.
- Cobertura mÃ­nima recomendada: 80%.

---

## Observabilidade

- Logs estruturados com correlaÃ§Ã£o por `trace_id` e `customer_id`.
- MÃ©tricas no padrÃ£o Prometheus.
- Alertas para aumento de risco agregado, falhas de ingestÃ£o e drift de modelo.

---

## Seguranca, privacidade e LGPD

- MinimizaÃ§Ã£o de dados pessoais.
- PseudonimizaÃ§Ã£o de identificadores de cliente.
- Criptografia em trÃ¢nsito (TLS) e em repouso.
- Controle de acesso por funÃ§Ã£o (RBAC).
- RetenÃ§Ã£o de dados com polÃ­ticas por tipo de dado.
- Trilhas de auditoria para decisÃµes automatizadas.

---

## Governanca de modelos

- Versionamento de modelos com validaÃ§Ã£o antes de produÃ§Ã£o.
- Monitoramento de drift e performance.
- Rollback automÃ¡tico se o modelo degradar.
- Registro de features e datasets usados no treino.

---

## Deploy e operacao

- Ambientes: `dev`, `staging`, `prod`.
- Pipeline CI/CD com testes e validaÃ§Ãµes.
- Infra como cÃ³digo em `infra/`.
- EstratÃ©gia de rollout: canary ou blue/green.

---

## Roadmap

### Fase 1

- Ingestao.
- Feature store.
- Anomalia.
- Dashboard bÃ¡sico.

### Fase 2

- PrevisÃ£o de risco.
- Explainability.
- Economia estimada.

### Fase 3

- IntegraÃ§Ãµes reais.
- Monitoramento de drift.
- OtimizaÃ§Ã£o de agendamento.

---

## Contribuicao

- Abra uma issue descrevendo o problema ou proposta.
- Envie um PR com testes e documentaÃ§Ã£o atualizada.

---

## Suporte e contato

- Email: murylobrayan@gmail.com
- Comercial: murylobrayan@gmail.com

---

## Licenca

MIT ou Apache 2.0

---



- DetecÃ§Ã£o de retorno de OS: se um cliente reabre OS em atÃ© X dias, prioriza anÃ¡lise de causa raiz e evita visitas repetidas.
- Prioridade inteligente de fila: ordena visitas por risco e impacto (clientes crÃ­ticos, empresas, SLA premium).
- PrÃ©-diagnÃ³stico antes da visita: testes remotos automÃ¡ticos e checklist do tÃ©cnico com provÃ¡vel causa.
- Cancelamento automÃ¡tico de OS: se a linha estabilizar por X minutos, cancela e notifica o cliente.
- Controle de reincidÃªncia por CTO/OLT: identifica CTOs que caem com frequÃªncia e indica manutenÃ§Ã£o preventiva.
- Monitoramento de qualidade pÃ³s-ativaÃ§Ã£o: sinal ruim nas primeiras X horas aciona retorno preventivo do tÃ©cnico.
