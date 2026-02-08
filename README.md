# AI Network Guardian
**Plataforma de IA para prever, diagnosticar e resolver falhas de conectividade, reduzindo custos operacionais e melhorando SLA.**

> O que este projeto faz: monitora telemetria e eventos de rede, detecta anomalias, classifica causa provável (massiva vs individual), prevê risco de falha (2h/6h/24h), recomenda ou executa ações (reboot, abrir incidente, agendar ou cancelar visita) e exibe tudo em um dashboard com métricas de economia operacional.

---

## Sumário
- [Visão geral](#visao-geral)
- [Problema que resolve](#problema-que-resolve)
- [Status do projeto](#status-do-projeto)
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
- [Requisitos e dependências](#requisitos-e-dependencias)
- [Configuração](#configuracao)
- [Como rodar localmente](#como-rodar-localmente)
- [Testes e qualidade](#testes-e-qualidade)
- [Observabilidade](#observabilidade)
- [Segurança, privacidade e LGPD](#seguranca-privacidade-e-lgpd)
- [Governança de modelos](#governanca-de-modelos)
- [Deploy e operação](#deploy-e-operacao)
- [Roadmap](#roadmap)
- [Contribuição](#contribuicao)
- [Suporte e contato](#suporte-e-contato)
- [Licença](#licenca)

---

## Visao geral

No fluxo tradicional de ISP:

1. Cliente perde sinal
2. Cliente abre chamado
3. Agenda técnico
4. Alto custo operacional

O **AI Network Guardian** antecipa o problema:

- Detecta degradação antes da queda completa.
- Separa queda massiva de problema individual.
- Executa autocorreção quando possível.
- Comunica o cliente automaticamente.
- Agenda visita apenas quando necessário.
- Cancela visitas quando o problema normaliza.
- Mede economia e melhora de SLA e MTTR.

---

## Problema que resolve

- Reduz visitas técnicas desnecessárias.
- Diminui MTTR e melhora SLA percebido.
- Aumenta capacidade do NOC sem inflar equipe.
- Cria previsibilidade com risco de falha por janela.

---

## Status do projeto

- Em desenvolvimento.
- Escopo atual: POC funcional com pipeline de dados, modelos iniciais e dashboard.
- Próximo marco: integração com sistemas reais de OSS/BSS.

---

## Principais funcionalidades

### Monitoramento e inteligência

- Ingestão de dados em batch ou near real time.
- Fontes: telemetria de sinal, status de ONU/modem, eventos de rede, histórico de chamados/OS, contexto de região e vizinhos, clima.
- Feature Store com agregações em janelas de 5, 15 e 60 minutos, além de histórico diário e semanal.
- Detecção de anomalias.
- Classificação de causa provável.
- Previsão de risco de falha.

### Ações automáticas

- Reboot controlado quando elegível.
- Abertura de incidente massivo.
- Pausa de agendamentos em regiões afetadas.
- Agendamento inteligente.
- Notificação ao cliente.

### Operação e gestão

Dashboard com:

- Fila de clientes em risco.
- Heatmap por região.
- Incidentes ativos.
- Auditoria das decisões da IA.
- Economia estimada.
- Métricas de SLA e MTTR.

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

- Isolation Forest.
- One Class SVM.

Saída:

```
anomaly_score
```

### Classificação

- LightGBM ou XGBoost.

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

- Resultados dos modelos.
- Contexto do cliente.
- Regras de segurança.
- Custo esperado.

Exemplo:

- Massiva detectada: abrir incidente.
- Individual: tentar reboot.
- Estabilizou: cancelar OS.
- Persistiu: agendar visita.

Todas as decisões são auditadas.

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

- NOC Overview.
- Fila de risco.
- Dossiê do cliente.
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

## Requisitos e dependencias

- Docker.
- Python 3.11.
- Node 18.
- Banco de dados: PostgreSQL ou MySQL.

---

## Configuracao

Crie um `.env` na raiz com os valores necessários. Exemplo mínimo:

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

- Testes unitários e de integração com `pytest`.
- Linters e formatação: `ruff`, `black` e `isort`.
- Cobertura mínima recomendada: 80%.

---

## Observabilidade

- Logs estruturados com correlação por `trace_id` e `customer_id`.
- Métricas no padrão Prometheus.
- Alertas para aumento de risco agregado, falhas de ingestão e drift de modelo.

---

## Seguranca, privacidade e LGPD

- Minimização de dados pessoais.
- Pseudonimização de identificadores de cliente.
- Criptografia em trânsito (TLS) e em repouso.
- Controle de acesso por função (RBAC).
- Retenção de dados com políticas por tipo de dado.
- Trilhas de auditoria para decisões automatizadas.

---

## Governanca de modelos

- Versionamento de modelos com validação antes de produção.
- Monitoramento de drift e performance.
- Rollback automático se o modelo degradar.
- Registro de features e datasets usados no treino.

---

## Deploy e operacao

- Ambientes: `dev`, `staging`, `prod`.
- Pipeline CI/CD com testes e validações.
- Infra como código em `infra/`.
- Estratégia de rollout: canary ou blue/green.

---

## Roadmap

### Fase 1

- Ingestao.
- Feature store.
- Anomalia.
- Dashboard básico.

### Fase 2

- Previsão de risco.
- Explainability.
- Economia estimada.

### Fase 3

- Integrações reais.
- Monitoramento de drift.
- Otimização de agendamento.

---

## Contribuicao

- Abra uma issue descrevendo o problema ou proposta.
- Envie um PR com testes e documentação atualizada.

---

## Suporte e contato

- Email: murylobrayan@gmail.com
- Comercial: murylobrayan@gmail.com

---

## Licenca

MIT ou Apache 2.0


