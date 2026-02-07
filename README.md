# AI Network Guardian
**Plataforma de IA para prever, diagnosticar e resolver falhas de conectividade — reduzindo custos operacionais e melhorando SLA.**

> O que este projeto faz: monitora telemetria e eventos de rede, detecta anomalias, classifica causa provável (massiva vs individual), prevê risco de falha (2h/6h/24h), recomenda ou executa ações (reboot, abrir incidente, agendar ou cancelar visita) e exibe tudo em um dashboard com métricas de economia operacional.

---

## Sumário
- [Visão geral](#visao-geral)
- [Principais funcionalidades](#principais-funcionalidades)
- [Arquitetura](#arquitetura)
- [Fluxogramas](#fluxogramas)
- [Modelos de IA](#modelos-de-ia)
- [Decision Engine](#decision-engine)
- [Estrutura do repositório](#estrutura-do-repositorio)
- [Banco de dados](#banco-de-dados)
- [API](#api)
- [Dashboard](#dashboard)
- [Métricas e economia](#metricas-e-economia)
- [Como rodar localmente](#como-rodar-localmente)
- [Roadmap](#roadmap)
- [Segurança e auditoria](#seguranca-e-auditoria)
- [Licença](#licenca)

---

## Visao geral

No fluxo tradicional de ISP:

1. Cliente perde sinal  
2. Cliente abre chamado  
3. Agenda técnico  
4. Alto custo operacional

O **AI Network Guardian** antecipa o problema:

- detecta degradação antes da queda completa
- separa queda massiva de problema individual
- executa auto correção quando possível
- comunica o cliente automaticamente
- agenda visita apenas quando necessário
- cancela visitas quando o problema normaliza
- mede economia e melhora de SLA e MTTR

---

## Principais funcionalidades

### Monitoramento e Inteligência
- Ingestão de dados em batch ou near real time:
  - telemetria de sinal
  - status de ONU ou modem
  - eventos de rede
  - histórico de chamados e OS
  - contexto de região e vizinhos
  - clima

- Feature Store com agregações:
  - janelas de 5, 15 e 60 minutos
  - histórico diário e semanal

- Detecção de anomalias
- Classificação de causa provável
- Previsão de risco de falha

### Ações automáticas
- reboot controlado
- abertura de incidente massivo
- pausa de agendamentos em regiões afetadas
- agendamento inteligente
- notificação ao cliente

### Operação e gestão
Dashboard com:
- fila de clientes em risco
- heatmap por região
- incidentes ativos
- auditoria das decisões da IA
- economia estimada
- métricas de SLA e MTTR

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

  I1 --> R1{Pode autocorrigir}
  R1 -->|Sim| B1[Reboot controlado]
  R1 -->|Nao| T1[Agendar tecnico]

  B1 --> V1{Estabilizou}
  V1 -->|Sim| C1[Fechar caso]
  V1 -->|Nao| T1
```

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

### Detecção de anomalia
- Isolation Forest
- One Class SVM

Saída:
```
anomaly_score
```

### Classificação
- LightGBM ou XGBoost

Classes:
- MASSIVE_OUTAGE
- CUSTOMER_PREMISES
- EQUIPMENT_FAIL
- EXTERNAL_NETWORK
- INTERMITTENT

### Previsão de risco
Saídas:
```
risk_2h
risk_6h
risk_24h
```

---

## Decision Engine

Decide baseado em:
- resultados dos modelos
- contexto do cliente
- regras de segurança
- custo esperado

Exemplo:
- massiva detectada → abrir incidente
- individual → tentar reboot
- estabilizou → cancelar OS
- persistiu → agendar visita

Todas decisões são auditadas.

---

## Estrutura do repositorio

```
ai-network-guardian/
├─ apps/
│  ├─ api/
│  └─ dashboard/
├─ services/
│  ├─ collectors/
│  ├─ feature-store/
│  ├─ ml-core/
│  ├─ decision-engine/
│  ├─ action-service/
│  └─ notification-service/
├─ infra/
├─ docs/
├─ notebooks/
└─ README.md
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
- NOC Overview
- Fila de risco
- Dossie do cliente
- KPIs e economia
- Explainability

---

## Metricas e economia

KPIs:
- Precision e Recall
- MTTR
- SLA
- OS evitadas

Economia:
```
economia = OS_evitadas * custo_medio_visita
```

---

## Como rodar localmente

Pré requisitos:
- Docker
- Python 3.11
- Node 18

### Subir infra
```
docker compose up -d
```

### API
```
cd apps/api
pip install -r requirements.txt
uvicorn main:app --reload
```

### Dashboard
```
cd apps/dashboard
npm install
npm run dev
```

---

## Roadmap

### Fase 1
- ingestao
- feature store
- anomalia
- dashboard basico

### Fase 2
- previsao risco
- explainability
- economia estimada

### Fase 3
- integracoes reais
- monitoramento de drift
- otimizacao de agendamento

---

## Seguranca e auditoria
- rate limit de acoes
- modo simulacao
- auditoria completa das decisoes

---

## Licenca
MIT ou Apache 2.0
