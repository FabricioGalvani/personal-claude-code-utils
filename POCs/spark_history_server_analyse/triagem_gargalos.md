Preciso que você analise um AWS Glue job e identifique os principais gargalos de performance, com evidência concreta para cada um.

## Contexto do job
- Nome do job: <NOME>
- O que ele faz (em 1-2 linhas): <ex: lê parquet do bucket X, faz join com tabela Y, agrega e grava em Z>
- Glue version: <ex: 4.0>
- Worker type / número: <ex: G.1X x 10>
- Tempo de execução atual: <ex: 45min> | Tempo desejado: <ex: 15min>
- Volume de dados (input): <ex: ~200GB parquet>

## Recursos que você tem acesso
- Script do job: <caminho no repo ou s3://...>
- Job parameters (default_arguments / Terraform / CloudFormation): <colar bloco ou apontar arquivo>
- Spark event logs: s3://<bucket>/spark-logs/ (job tem --enable-spark-ui=true)
- Job run ID de uma execução representativa: <jr_xxxxx>
- Região AWS: <ex: us-east-1>
- Como autenticar: <perfil AWS / role / etc>

## O que fazer
1. Suba o Spark History Server localmente via Docker usando o Dockerfile de aws-samples/aws-glue-samples (utilities/Spark_UI), apontando spark.history.fs.logDirectory para o caminho s3a:// dos meus event logs.
2. Abra o run mencionado e examine: stages mais lentos, distribuição de task duration (procure skew), shuffle read/write, spill (memory e disk), GC time, executor utilization ao longo do tempo.
3. Cheque também as métricas do CloudWatch no namespace `Glue` para o run: glue.driver.aggregate.numCompletedTasks, glue.ALL.jvm.heap.usage, glue.driver.ExecutorAllocationManager.executors.numberAllExecutors.
4. Cruze com o script: encontre o trecho de código responsável por cada gargalo identificado.

## Formato da resposta
Markdown, ordenado por impacto estimado (maior primeiro). Para cada gargalo:
- **Sintoma**: o que você viu no Spark UI (cite stage ID, número da task, métrica numérica)
- **Causa**: por que está acontecendo, com referência à linha/bloco do script
- **Fix**: mudança concreta (diff de código OU parâmetro de job OU spark conf), não conselho genérico
- **Impacto estimado**: ganho aproximado de tempo/custo, e o risco da mudança

## O que NÃO quero
- Recomendações sem evidência ("considere broadcast join" sem mostrar que existe um join com tabela pequena)
- Refactor amplo — foco em mudanças cirúrgicas
- Sugestões que exigem mudar a arquitetura (trocar Glue por EMR, etc.) — fica fora de escopo nesta análise

Limite: 5 gargalos principais. Se encontrar mais, lista no fim como "observações adicionais".