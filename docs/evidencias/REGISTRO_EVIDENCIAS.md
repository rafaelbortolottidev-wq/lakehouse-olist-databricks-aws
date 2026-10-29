# P04 — Registro de evidências

## Evidência 01 — Upload dos arquivos Olist
- Etapa: ingestão inicial no Unity Catalog Volume
- Arquivo: `01_upload_olist_9_csvs_unity_catalog.png`
- Resultado comprovado: 9 arquivos CSV enviados ao Volume `workspace.default.olist_raw`, com indicadores de sucesso na interface.
- Status: validado por captura de tela; ainda não versionado no GitHub.

## Evidência 02 — Leitura de pedidos Olist com PySpark
- Etapa: validação da fonte CSV antes da criação da Bronze
- Arquivo: `02_leitura_orders_pyspark_99441_registros.png`
- Resultado comprovado: execução concluída; `df_orders_raw.count()` = 99.441; schema com 8 colunas do dataset de pedidos, inicialmente todas `string`; amostra de 5 linhas exibida.
- Fonte: `/Volumes/workspace/default/olist_raw/olist_orders_dataset.csv`
- Status: validado por captura de tela; ainda não versionado no GitHub.
- Nota: a captura não comprova ainda criação de tabela Delta Bronze nem testes de qualidade.

## Evidência 03 — Criação da tabela Bronze Delta Lake
- Etapa: persistência inicial dos pedidos Olist em Delta Lake, via notebook no Databricks Free Edition.
- Data da captura: 28/10/2026 (horário de Brasília).
- Arquivo: `03_tabela_bronze_delta_orders_99441_registros.png`
- Resultado comprovado: criação de `workspace.bronze.orders`; 99.441 registros; `DESCRIBE DETAIL` mostra formato `delta`, `numFiles` = 1, `sizeInBytes` = 5.731.919.
- Execução: notebook na branch `feature/estrutura-inicial`, com sinal de sucesso na interface.
- Observação: código capturado usa `.mode("overwrite")`, uma estratégia de substituição da tabela; revisar antes de tornar o pipeline recorrente.
- Status: validado por captura de tela; ainda não versionado no GitHub.

## Evidência 04 — Validação da Bronze sem sobrescrita
- Etapa: validação de reexecução segura da carga inicial da Bronze.
- Data da captura: 28/10/2026 (horário de Brasília).
- Arquivo: `04_validacao_bronze_sem_sobrescrita.png`
- Resultado comprovado: código com `spark.catalog.tableExists(tabela)` detecta tabela existente e informa `Tabela Bronze já existe. Nenhuma gravação realizada.`; `workspace.bronze.orders` contém 99.441 registros e `DESCRIBE DETAIL` aponta `delta`, 1 arquivo e 5.731.919 bytes.
- Limite da validação: impede sobrescrita nesta execução, mas não implementa ingestão incremental nem testa atualização de novos arquivos.
- Status: validado por captura de tela; ainda não versionado no GitHub.

## Evidência 05 — Tabela Bronze no Catalog Explorer
- Etapa: validação da tabela persistida no Unity Catalog.
- Data da captura: 28/10/2026 (horário de Brasília).
- Arquivo: `05_catalogo_tabela_bronze_orders_unity_catalog.png`
- Resultado comprovado: `workspace.bronze.orders` aparece no Catalog Explorer como tabela gerenciada (*Managed*), com fonte/formato Delta; 8 colunas visíveis, todas do tipo `string`.
- Observação: o painel informa que o recurso de Data Quality Monitoring não está habilitado. Esta captura não comprova regras de qualidade nem sua execução.
- Status: validado por captura de tela; ainda não versionado no GitHub.

## Evidência 06 — Commit e push do notebook Bronze no GitHub
- Etapa: versionamento do código PySpark seguindo GitHub Flow.
- Data da captura: 28/10/2026 (horário de Brasília).
- Arquivo: `06_commit_push_bronze_feature_github.png`
- Resultado comprovado: repositório `lakehouse-olist-databricks-aws` exibe `notebooks/Bronze Orders Pipeline.ipynb` na branch `feature/estrutura-inicial`, com o commit `feat: adiciona pipeline bronze de pedidos Olist` e indicador de **1 commit à frente da `main`**.
- Observação: push confirmado; Pull Request e merge ainda não comprovados.
- Status: validado por captura de tela; esta evidência ainda não foi versionada no GitHub.

Arquivos de evidência estão disponíveis para posterior inclusão no repositório em `docs/evidencias/`, utilizando branch de desenvolvimento e Pull Request.
