# ATIVIDADE HANDS ON DATABRICKS

# Versão Técnica — Documentação do Projeto

## Visão Geral

Este projeto implementa um pipeline de dados ponta a ponta no Databricks seguindo a arquitetura Medalhão (Bronze, Silver e Gold), com o objetivo de transformar dados brutos em uma base analítica estruturada para suporte à política de concessão de crédito estudantil.

O resultado final é a tabela `workspace.gold.final_credit_policy`, contendo:

- `application_id`
- `student_id`
- `risk_segment`
- `recommended_action`
- `recommended_limit`
- `rationale_features`

---

## Arquitetura e Pipeline

### Camada Bronze

Responsável pela ingestão dos arquivos CSV:

- academic
- applications
- loans
- students
- transactions_monthly
- repayments
- data_dictionary

Processo:
- Leitura via Pandas
- Conversão para DataFrame Spark
- Persistência em formato Delta
- Escrita no schema `workspace.bronze`

Objetivo: manter os dados brutos como fonte de verdade, com mínima transformação.

---

### Camada Silver

Responsável pela curadoria, deduplicação e agregações.

Principais transformações:

#### students_clean
- Remoção de duplicidades via `SELECT DISTINCT`.

#### applications_clean
- Seleção do registro mais recente por `application_id` usando `ROW_NUMBER()`.

#### loans_agg
Agregação por `student_id`:
- `last_loan_status`
- `ever_default`
- `total_loans`

#### transactions_agg
Agregações financeiras:
- `avg_inflows`
- `avg_outflows`
- `avg_balance`
- `total_overdraft_events`
- `total_late_fees`
- `avg_net_cashflow`

#### consolidated_table_applications
Junção de estudantes, aplicações, histórico de crédito e métricas financeiras agregadas.

Objetivo: entregar uma base consistente, deduplicada e enriquecida por estudante.

---

### Camada Gold

Responsável pela implementação da lógica de risco e política de crédito.

#### credit_policy_base
Criação de variáveis estratégicas:
- `avg_net_cashflow` (capacidade mensal estimada)
- `overdraft_risk` (baixo, médio, alto)
- `ever_default`
- `balance_profile`
- `total_loans`
- `total_late_fees`

#### credit_risk_segmented
Segmentação em:
- `high_risk`
- `medium_risk`
- `low_risk`

Regras baseadas em:
- Histórico de default
- Frequência de overdraft
- Fluxo de caixa médio

#### final_credit_policy
Geração de:
- `recommended_action` (aprovar, aprovar com limite reduzido, rejeitar/exigir fiador)
- `recommended_limit` (0x, 3x ou 6x do fluxo líquido)
- `rationale_features` (transparência da decisão)

---

## Critérios de Qualidade Aplicados

- Deduplicação com `ROW_NUMBER`
- Controle de granularidade por `student_id`
- Regras declarativas em SQL
- Campo de justificativa explícita para auditoria
- Separação clara entre camadas bruta, curada e analítica

---

## Limitações e Próximos Passos Técnicos

1. Implementar tratamento estruturado de nulos e outliers (IQR, Z-Score).
2. Criar validações automatizadas de qualidade (ex.: expectativas Delta).
3. Expandir feature engineering:
   - Debt-to-Income ratio
   - Tempo desde último default
   - Indicadores acadêmicos combinados
4. Evoluir de regras heurísticas para modelo supervisionado:
   - Regressão Logística
   - Random Forest
   - Score de probabilidade de default
5. Tornar o pipeline incremental e agendado via jobs.
6. Implementar versionamento de regras de crédito.
7. Criar dashboard analítico em Tableau ou Databricks SQL.

O projeto está estruturado para suportar essa evolução sem necessidade de refatoração estrutural.

---

# Versão Executiva — Documento para Diretoria

## Objetivo

Construir uma base analítica confiável para fundamentar a política de concessão de crédito estudantil, a partir da consolidação de dados acadêmicos, financeiros e históricos de crédito.

---

## O Que Foi Entregue

1. Pipeline completo de ingestão e transformação de dados no Databricks.
2. Estrutura em três camadas (bruta, tratada e analítica).
3. Consolidação de informações por estudante e aplicação.
4. Segmentação de risco baseada em critérios objetivos.
5. Recomendação automática de:
   - Aprovação
   - Aprovação com limite reduzido
   - Rejeição ou exigência de fiador
6. Definição de limite sugerido com base na capacidade financeira.
7. Transparência da decisão por meio da explicação das variáveis utilizadas.

---

## Benefícios para o Negócio

- Padronização da análise de crédito.
- Redução de subjetividade na decisão.
- Rastreabilidade e auditabilidade das regras.
- Base estruturada para evolução futura para modelos preditivos.
- Capacidade de simular cenários de política de crédito.

---

## Como a Política Funciona

A decisão considera:

- Histórico de inadimplência
- Frequência de uso de cheque especial
- Fluxo de caixa médio mensal
- Estabilidade de saldo
- Experiência prévia com crédito

Com base nisso, o sistema classifica o solicitante em três níveis de risco e sugere ação e limite compatíveis com sua capacidade financeira.

---

## Próximos Passos Estratégicos

1. Incorporar modelo estatístico para estimar probabilidade de default.
2. Medir taxa histórica de inadimplência por segmento.
3. Simular impacto financeiro de mudanças na política.
4. Criar dashboard executivo para acompanhamento contínuo.
5. Automatizar atualização periódica da base.

---

## Conclusão

O projeto entrega uma base estruturada e operacional para suportar decisões de crédito com maior consistência, transparência e escalabilidade. A arquitetura implementada permite evolução rápida para modelos mais sofisticados, mantendo governança e rastreabilidade das decisões.
