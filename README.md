﻿# AI Network Guardian
**Plataforma de IA para prever, diagnosticar e resolver falhas de conectividade, reduzindo custos operacionais e melhorando SLA.**

> O que este projeto faz: monitora telemetria e eventos de rede, detecta anomalias, classifica causa provável (massiva vs. individual), prevê risco de falha (2h/6h/24h), recomenda ou executa ações (reboot, abrir incidente, agendar ou cancelar visita) e exibe tudo em um dashboard com métricas de economia operacional.

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
- [Outras alavancas de economia](#outras-alavancas-de-economia)
- [Requisitos e dependências](#requisitos-e-dependencias)
- [Configuração](#configuracao)
- [Como rodar localmente](#como-rodar-localmente)
- [Testes e qualidade](#testes-e-qualidade)
- [Observabilidade](#observabilidade)
- [Segurança, privacidade e LGPD](#seguranca-privacidade-e-lgpd)
- [Governança de modelos](#governanca-de-modelos)
- [Deploy e operação](#deploy-e-operacao)
- [Roadmap](#roadmap)
- [Suporte e contato](#suporte-e-contato)
- [Licença](#licenca)

---

<a id="visao-geral"></a>
## Visão geral
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

<a id="problema-que-resolve"></a>
## Problema que resolve
- Reduz visitas técnicas desnecessárias.
- Diminui MTTR e melhora SLA percebido.
- Aumenta capacidade do NOC sem inflar equipe.
- Cria previsibilidade com risco de falha por janela.

---

<a id="status-do-projeto"></a>
## Status do projeto
- Em desenvolvimento.
- Escopo atual: POC funcional com pipeline de dados, modelos iniciais e dashboard.
- Próximo marco: integração com sistemas reais de OSS/BSS.

---

<a id="principais-funcionalidades"></a>
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
- Proteção de Cliente em Janela de Ativação/Mudança (CTO).

### Operação e gestão

Dashboard com:

- Fila de clientes em risco.
- Heatmap por região.
- Incidentes ativos.
- Auditoria das decisões da IA.
- Economia estimada.
- Métricas de SLA e MTTR.

---

<a id="arquitetura"></a>
## Arquitetura
Stack sugerida:

- Data e ETL: Python
- Stream: Redis Streams ou RabbitMQ
- Banco: PostgreSQL ou MySQL
- API: FastAPI ou Laravel
- ML: scikit-learn, LightGBM ou XGBoost
- Frontend: React + Tailwind

---

<a id="fluxogramas"></a>
## Fluxogramas
### 1) Fluxo geral

```mermaid
flowchart TB
  A[Ingestão de dados] --> B[Camada Raw]
  B --> C[Camada Silver]
  C --> D[Feature Store]
  D --> E[ML Core]
  E --> F[Decision Engine]
  F --> G[Ações automaticas]
  F --> H[Notificações]
  E --> I[Dashboard]
  F --> I
  G --> J[Feedback Loop]
  J --> D
```

---

### 2) Fluxo de decisão

```mermaid
flowchart TD
  S[Evento cliente offline] --> F1[Calcular features]
  F1 --> M1[Modelo anomalia]
  F1 --> M2[Modelo classificação]
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

### 2.1) Fluxo de economia e preveno

```mermaid
flowchart TD
  A[Evento de rede ou cliente] --> B{Reabertura de OS <= X dias?}
  B -->|Sim| B1[Priorizar RCA e validao tcnica]
  B -->|No| C{Qualidade ps-ativao <= X horas?}
  C -->|Ruim| C1[Retorno preventivo antes da reclamao]
  C -->|Ok| D{Linha estabilizou >= X min?}
  D -->|Sim| D1[Cancelar OS e notificar cliente]
  D -->|No| E{Risco x impacto - SLA/cliente crtico?}
  E -->|Alto| E1[Prioridade alta na fila]
  E -->|Normal| E2[Prioridade normal]
  E1 --> F{CTO/OLT reincidente?}
  E2 --> F
  F -->|Sim| F1[Acionar manuteno preventiva]
  F -->|No| G[Fluxo padro de atendimento]
```



---

### 3) Sequência de monitoramento

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
  ML->>DEC: scores e previsões
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

<a id="modelos-de-ia"></a>
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

<a id="decision-engine"></a>
## Decision Engine
Decide baseado em:

- Resultados dos modelos.
- Contexto do cliente.
- Regras de segurança.
- Custo esperado.

Exemplo:

- Massiva detectada: abrir incidente.
- Individual: tentar reboot.
- Queda individual na mesma CTO durante janela de ativação/manutenção: acionar técnico no local para corrigir antes de sair.
- Estabilizou: cancelar OS.
- Persistiu: agendar visita.

Todas as decisões são auditadas.

Regra específica:
- Proteção de Cliente em Janela de Ativação/Mudança (CTO): se um cliente da mesma CTO cair dentro do intervalo de início/fim de uma ativação ou manutenção, o técnico responsável deve validar e corrigir no local.

---

<a id="estrutura-do-repositorio"></a>
## Estrutura do repositório
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

<a id="banco-de-dados"></a>
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

<a id="api"></a>
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

<a id="dashboard"></a>
## Dashboard
Telas:

- NOC Overview.
- Fila de risco.
- Dossiê do cliente.
- KPIs e economia.
- Explainability.

---

<a id="metricas-e-economia"></a>
## Métricas e economia
KPIs:

- Precision e Recall.
- MTTR.
- SLA.
- OS evitadas.

Economia:

```
economia = OS_evitadas * custo_medio_visita
```

Premissas:
- Custo médio de manutenção: R$90.
- Média de 2.000 reparos por dia.

---

<a id="outras-alavancas-de-economia"></a>
## Outras alavancas de economia
- Detecção de retorno de OS: se um cliente reabre OS em até X dias, prioriza análise de causa raiz e evita visitas repetidas.
- Prioridade inteligente de fila: ordena visitas por risco e impacto (clientes críticos, empresas, SLA premium).
- Pré-diagnóstico antes da visita: testes remotos automáticos e checklist do técnico com provável causa.
- Cancelamento automático de OS: se a linha estabilizar por X minutos, cancela e notifica o cliente.
- Controle de reincidência por CTO/OLT: identifica CTOs que caem com frequência e indica manutenção preventiva.
- Monitoramento de qualidade pós-ativação: sinal ruim nas primeiras X horas aciona retorno preventivo do técnico.

---

<a id="requisitos-e-dependencias"></a>
## Requisitos e dependências
- Docker.
- Python 3.11.
- Node 18.
- Banco de dados: PostgreSQL ou MySQL.

---

<a id="configuracao"></a>
## Configuração
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

<a id="como-rodar-localmente"></a>
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

<a id="testes-e-qualidade"></a>
## Testes e qualidade
- Testes unitários e de integração com `pytest`.
- Linters e formatação: `ruff`, `black` e `isort`.
- Cobertura mínima recomendada: 80%.

---

<a id="observabilidade"></a>
## Observabilidade
- Logs estruturados com correlação por `trace_id` e `customer_id`.
- Métricas no padrão Prometheus.
- Alertas para aumento de risco agregado, falhas de ingestão e drift de modelo.

---

<a id="seguranca-privacidade-e-lgpd"></a>
## Segurança, privacidade e LGPD
- Minimização de dados pessoais.
- Pseudonimização de identificadores de cliente.
- Criptografia em trânsito (TLS) e em repouso.
- Controle de acesso por função (RBAC).
- Retenção de dados com políticas por tipo de dado.
- Trilhas de auditoria para decisões automatizadas.

---

<a id="governanca-de-modelos"></a>
## Governança de modelos
- Versionamento de modelos com validação antes de produção.
- Monitoramento de drift e performance.
- Rollback automático se o modelo degradar.
- Registro de features e datasets usados no treino.

---

<a id="deploy-e-operacao"></a>
## Deploy e operação
- Ambientes: `dev`, `staging`, `prod`.
- Pipeline CI/CD com testes e validações.
- Infra como código em `infra/`.
- Estratégia de rollout: canary ou blue/green.

---

<a id="roadmap"></a>
## Roadmap
### Fase 1

- Ingestão.
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

<a id="suporte-e-contato"></a>
## Suporte e contato
- Email: murylobrayan@gmail.com
- Comercial: murylobrayan@gmail.com

---

<a id="licenca"></a>
## Licença
MIT ou Apache 2.0

---



- Detecção de retorno de OS: se um cliente reabre OS em até X dias, prioriza análise de causa raiz e evita visitas repetidas.
- Prioridade inteligente de fila: ordena visitas por risco e impacto (clientes críticos, empresas, SLA premium).
- Pré-diagnóstico antes da visita: testes remotos automáticos e checklist do técnico com provável causa.
- Cancelamento automático de OS: se a linha estabilizar por X minutos, cancela e notifica o cliente.
- Controle de reincidência por CTO/OLT: identifica CTOs que caem com frequência e indica manutenção preventiva.
- Monitoramento de qualidade pós-ativação: sinal ruim nas primeiras X horas aciona retorno preventivo do técnico.
