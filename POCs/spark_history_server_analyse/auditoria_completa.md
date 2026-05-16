Preciso de uma auditoria completa de um AWS Glue job: performance, custo, qualidade de código e adequação da configuração.

## Contexto do job
- Nome do job: <NOME>
- O que ele faz: <descrição mais detalhada>
- Glue version: <ex: 4.0>
- Worker type / número / auto-scaling habilitado?: <ex: G.1X x 10, auto-scaling off>
- Tempo de execução típico: <ex: 45min>
- Frequência de execução: <ex: a cada 1h>
- Volume de input: <ex: ~200GB parquet, ~1B linhas>
- Custo mensal aproximado do job (DPU-hours): <se souber>
- SLA / orçamento: <ex: precisa rodar em <20min, custo <$X/mês>

## Recursos disponíveis
- Script do job: <caminho>
- Job parameters completos (Terraform/CloudFormation/CLI): <colar>
- Spark event logs: s3://<bucket>/spark-logs/
- IDs de 3 runs representativos (sucesso, lento, falha se tiver): <jr_..., jr_..., jr_...>
- Schema dos inputs principais: <colar ou apontar Glue Catalog table>
- Região AWS e credenciais: <...>

## O que analisar

### 1. Performance
Suba o Spark History Server local (Docker, repo aws-samples/aws-glue-samples). Para cada run analisado, identifique:
- Stages com maior wall-clock time
- Skew (diferença entre task p50 e p99/max no mesmo stage)
- Shuffle: bytes lidos/escritos, e se há shuffle desnecessário (ex: múltiplos joins na mesma chave sem repartition prévio)
- Spill: memory spill e disk spill por executor — sinal de memória insuficiente ou partições mal dimensionadas
- GC overhead: tempo de GC / tempo de task
- Executor utilization: gráfico de executores ativos ao longo do tempo — procurar caudas longas com poucos executores ativos
- Operações caras: collect(), toPandas(), count() repetidos, UDFs Python (vs nativas/pandas UDF)

### 2. Custo
- DPU-hours por run × frequência × preço da região
- Worker type está adequado? (G.1X vs G.2X vs G.4X vs G.8X conforme memória/CPU exigida pelos stages)
- Auto-scaling faz sentido para o perfil de carga?
- Tempo de "warm-up" e tempo ocioso pós-conclusão estão pagos sem trabalho útil?
- Comparar com cenário alternativo: mesma carga em Glue 5.0 (se ainda 4.0), ou em EMR Serverless — só estimativa, sem migrar nada

### 3. Configuração
- Job parameters adequados? Faltam parâmetros tipo --enable-observability-metrics, --enable-auto-scaling, --enable-glue-datacatalog conforme o caso
- Spark configs custom (--conf) — alguma está mascarando problema em vez de resolver?
- Rolling logs habilitado para Spark UI?
- Bookmarks usados de forma correta (se aplicável)?

### 4. Código
Código review focado em anti-patterns típicos de Glue/Spark:
- DynamicFrame vs DataFrame: usando o certo no contexto certo?
- Joins sem broadcast hint onde uma das tabelas é pequena
- Repartition/coalesce posicionados em lugares ruins
- Schema inference em vez de schema explícito (custo alto em arquivos grandes)
- Predicates pushdown sendo aproveitados (especialmente Parquet/Iceberg)
- Operações que disparam shuffle escondidas (orderBy global, distinct em colunas alta cardinalidade)

## Formato da resposta
Um documento markdown estruturado:

**Seção 1 — Sumário executivo** (10 linhas máx): top 3 ações com maior ROI, ganho estimado em tempo e custo.

**Seção 2 — Achados de performance**: lista priorizada, cada item com Sintoma / Causa / Fix / Impacto / Risco (como descrito antes).

**Seção 3 — Achados de custo**: cálculo atual vs cenário otimizado, com premissas explicitadas.

**Seção 4 — Achados de configuração e código**: lista de mudanças sugeridas com diff sempre que possível.

**Seção 5 — Plano de execução proposto**: ordem sugerida para aplicar as mudanças, agrupando o que pode ir junto e o que precisa ser validado isoladamente para medir impacto.

**Seção 6 — O que ficou fora**: questões que você identificou mas que dependem de decisão de arquitetura ou contexto de negócio que você não tem.

## Regras
- Toda afirmação quantitativa deve citar a métrica de origem (Spark UI stage X / CloudWatch métrica Y / linha N do script)
- Se uma recomendação tem trade-off (ex: aumenta memória mas custa mais), explicite
- Se faltar informação para uma análise, liste no final como "perguntas para o time" — não invente