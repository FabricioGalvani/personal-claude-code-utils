---
mode: lineage
description: Rodar linhagem nos scripts em contexto.
---

Analise os scripts PySpark abaixo seguindo as 4 fases do agente de linhagem.
A tabela de destino é: ${input:tabela_destino:nome da tabela final}.

Se houver mais de um script, trate-os como **um único ETL**: um DataFrame
definido em um script pode ser consumido em outro. Assuma execução na ordem
em que os arquivos foram passados.

Comece pela Fase 1.