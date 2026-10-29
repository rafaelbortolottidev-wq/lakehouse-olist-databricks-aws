# DOCUMENTAÇÃO MASTER — P04 Lakehouse Olist Databricks + AWS

## 1. Objetivo e escopo
Desenvolver um Lakehouse de e-commerce com Databricks, PySpark, Delta Lake e Unity Catalog, evoluindo de carga batch para arquitetura Bronze/Silver/Gold e, futuramente, integração com AWS.

**Estado comprovado em 28/10/2026 (horário de Brasília):** primeira ingestão Bronze (`orders`) executada e validada em Databricks Free Edition; os demais oito conjuntos ainda não foram convertidos em tabelas Bronze.

## 2. Arquitetura implementada

```text
Olist (9 CSVs no computador)
    → upload manual
Unity Catalog Volume: workspace.default.olist_raw
    → leitura PySpark no notebook
Notebook: notebooks/Bronze Orders Pipeline.ipynb
    → escrita inicial e validação de existência
Tabela gerenciada Delta: workspace.bronze.orders
    → futura Silver / Gold (não implementadas ainda)
```

O armazenamento dos arquivos originais está no Volume gerenciado pelo Unity Catalog. Os notebooks ficam no Workspace/Git folder. A tabela Delta é armazenada e gerenciada pela plataforma Databricks. O compute da execução foi Serverless. **Não há ingestão automática do S3 nem recursos AWS implantados nesta etapa.**

## 3. Dados de origem
- Dataset: Olist Brazilian E-Commerce (arquivos CSV originais).
- Volume de destino: `/Volumes/workspace/default/olist_raw/`.
- Upload: **9 CSVs com êxito**, conforme evidência `01_upload_olist_9_csvs_unity_catalog.png`.
- Primeiro arquivo trabalhado: `olist_orders_dataset.csv`.
- Leitura da fonte: 99.441 linhas e 8 campos, todos inicialmente interpretados como `string` (`inferSchema=false`).

## 4. Notebook e processamento Bronze
- Repositório: `lakehouse-olist-databricks-aws`.
- Arquivo: `notebooks/Bronze Orders Pipeline.ipynb`.
- Leitura: `spark.read.format("csv").option("header", "true").option("inferSchema", "false")`.
- Catálogo/schema/tabela: `workspace.bronze.orders`.
- Criação do schema: `CREATE SCHEMA IF NOT EXISTS workspace.bronze`.
- Persistência: `df_orders_raw.write.format("delta").saveAsTable(tabela)` **apenas quando** `spark.catalog.tableExists(tabela)` retorna falso.
- Quando a tabela existe, o notebook não executa gravação e valida os dados existentes. Isso protege a carga inicial contra sobrescrita, **mas ainda não constitui um processo incremental nem uma política completa de idempotência**.

## 5. Resultados validados
| Verificação | Resultado | Evidência |
|---|---|---|
| Upload de CSVs | 9 arquivos no Volume | 01 |
| Leitura do dataset orders | 99.441 registros, 8 colunas | 02 |
| Criação da tabela Bronze | `workspace.bronze.orders` | 03 |
| Formato de armazenamento | `delta`; 1 arquivo; 5.731.919 bytes (no momento da captura) | 03 e 04 |
| Reexecução com tabela existente | Sem nova gravação; 99.441 registros preservados | 04 |
| Unity Catalog | Tabela gerenciada; 8 colunas tipo `string` | 05 |
| GitHub Flow | Commit e push na `feature/estrutura-inicial`; 1 commit à frente da `main` | 06 |

## 6. Versionamento
Foi utilizado GitHub Flow: `main` de base → `feature/estrutura-inicial` → notebook desenvolvido e testado → commit `feat: adiciona pipeline bronze de pedidos Olist` → push para a branch remota. **Pull Request e merge ainda pendentes** neste registro.

## 7. Custos e segurança
- Ambiente utilizado: Databricks Free Edition (sujeito a cotas e limitações da oferta).
- Não foi provisionado Databricks pago ou AWS especificamente para esta etapa.
- Dados brutos e tabelas **não** devem ser incluídos no repositório Git.
- Nunca armazenar tokens, segredos ou credenciais em notebooks, commits ou evidências.
- **Pendência a confirmar:** revogação de um token GitHub mostrado em captura anterior; não há confirmação no registro atual.

## 8. Limites atuais / próximas entregas
- Carga manual dos CSVs; sem Auto Loader e sem Lakeflow Jobs configurados.
- Apenas a tabela `orders` possui Bronze implementada.
- Não há transformações Silver, tabelas Gold, testes automatizados, DQ monitoring ou CI/CD final.
- Próximo passo de versionamento: incluir esta documentação e as evidências na branch, abrir PR, revisar e integrar na `main`.
- Próximo passo técnico: planejamento da Bronze dos outros CSVs e transformação Silver com tipagem, verificações de qualidade e deduplicação.

## 9. Evidências
Arquivo auxiliar: `docs/evidencias/REGISTRO_EVIDENCIAS.md`. As capturas 01–06 documentam os marcos acima. Devem ser versionadas na branch antes da integração à `main`.
