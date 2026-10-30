# DOCUMENTAÇÃO MASTER — P04 Lakehouse Olist Databricks + AWS

**Atualização:** 30/10/2026  
**Etapa:** 2 — Camada Bronze completa  
**Status:** Implementação validada; documentação e Pull Request pendentes.

## 1. Objetivo e escopo

Desenvolver um Lakehouse de e-commerce utilizando Databricks, PySpark, Delta Lake e Unity Catalog, com arquitetura Medallion (Bronze, Silver e Gold) e futura integração com serviços AWS.

**Estado atual:** nove arquivos CSV originais carregados, nove tabelas Bronze criadas em Delta Lake e contagens validadas.

A implementação utiliza Databricks Free Edition e compute Serverless.

## 2. Arquitetura implementada

```text
Olist — 9 arquivos CSV
         |
         v
Upload manual
         |
         v
Unity Catalog Volume
workspace.default.olist_raw
         |
         v
Databricks / PySpark
         |
         v
workspace.bronze
         |
         +-- orders
         +-- customers
         +-- geolocation
         +-- order_items
         +-- order_payments
         +-- order_reviews
         +-- products
         +-- sellers
         +-- product_category_name_translation
         |
         v
Delta Lake — 9 tabelas gerenciadas

Silver / Gold — Pendentes
```

Não foi configurada ingestão automática a partir do Amazon S3.

## 3. Dados de origem

Dataset público: Olist Brazilian E-Commerce.

**Volume:** `workspace.default.olist_raw`

**Caminho:** `/Volumes/workspace/default/olist_raw/`

Foram carregados nove arquivos CSV originais, preservados para rastreabilidade e reprocessamento.

## 4. Notebooks e processamento

**Repositório:** `lakehouse-olist-databricks-aws`

**Notebooks:**

- `notebooks/01_Bronze_Orders_Pipeline.ipynb`
- `notebooks/02_bronze_demais_tabelas.ipynb`

O primeiro notebook implementa a carga inicial de `orders`.

O segundo notebook processa os outros oito arquivos CSV utilizando um mapeamento parametrizado de fontes e tabelas de destino.

**Estratégia de ingestão:**

- Leitura com `spark.read.format("csv")`.
- Preservação inicial dos tipos como `string`.
- Gravação de tabelas gerenciadas em Delta Lake.
- Verificação de existência antes da gravação.
- Validação das quantidades e do formato físico das tabelas.

A proteção contra sobrescrita evita recriar tabelas existentes. Não implementa, por si só, cargas incrementais nem verifica alterações posteriores nos arquivos de origem.

## 5. Resultados validados — Bronze

| Tabela | Registros | Formato |
|---|---:|---|
| customers | 99.441 | Delta |
| geolocation | 1.000.163 | Delta |
| order_items | 112.650 | Delta |
| order_payments | 103.886 | Delta |
| order_reviews | 99.224 | Delta |
| orders | 99.441 | Delta |
| product_category_name_translation | 71 | Delta |
| products | 32.951 | Delta |
| sellers | 3.095 | Delta |

**Resultado:** 9 tabelas existentes, com formato Delta confirmado e contagens consistentes com o profiling de referência.

Essas verificações não substituem testes de unicidade, integridade referencial, completude e regras de negócio.

## 6. Incidente técnico — CSV order_reviews

Durante a criação da Bronze, foi identificada divergência na quantidade de registros de avaliações.

| Configuração de leitura | Registros |
|---|---:|
| Leitura inicial | 104.162 |
| `multiLine=true` | 99.249 |
| `multiLine=true`, `quote` e `escape` | 99.224 |

**Diagnóstico:** a interpretação de campos com aspas e quebras de linha afetava a contagem de registros.

Foi validado um DataFrame com 99.224 registros e sete colunas, compatível com o profiling original.

Com autorização, a tabela `workspace.bronze.order_reviews` foi substituída de maneira controlada.

**Resultado:** tabela corrigida para 99.224 registros, formato Delta.

O notebook de ingestão foi posteriormente consolidado com as configurações de leitura:

```python
.option("header", "true")
.option("multiLine", "true")
.option("quote", '"')
.option("escape", '"')
.option("inferSchema", "false")
```

O incidente demonstra a importância de validar os resultados de ingestão mesmo quando a execução termina sem erros.

## 7. Validação e qualidade

Validações executadas:

- Presença dos nove CSVs no Volume.
- Leitura e estrutura do dataset `orders`.
- Criação das nove tabelas Bronze.
- Contagens de registros das nove tabelas.
- Confirmação do formato Delta.
- Proteção contra recriação de tabelas existentes.
- Investigação e correção da contagem de `order_reviews`.
- Reexecução da lógica de existência sem sobrescrever tabelas.

**Ainda pendentes:** testes automatizados, validações de integridade, regras formais de qualidade, observabilidade e ingestão incremental.

## 8. Versionamento — GitHub Flow

### Entrega 1 — Estrutura inicial e Bronze Orders

- Branch: `feature/estrutura-inicial`
- Notebook de pedidos desenvolvido e validado.
- Documentação e primeiras evidências versionadas.
- Pull Request #1 integrado à `main`.
- Commit de merge: `6410a5d`.
- Branch de desenvolvimento excluída após o merge.

**Status:** concluída.

### Entrega 2 — Bronze completa

- Branch: `feature/bronze-demais-tabelas`
- Notebook reutilizável implementado.
- Nove tabelas Bronze validadas.
- Notebook original renomeado para seguir a numeração.
- Commit: `24a5afd`.
- Commit e push confirmados no GitHub.

**Status:** aguardando atualização documental e Pull Request para `main`.

Nunca realizar commits diretamente na `main`.

## 9. Custos e segurança

Ambiente utilizado: Databricks Free Edition, sujeito às cotas e limitações da modalidade.

Nenhuma infraestrutura AWS paga foi provisionada especificamente para o P04 até esta etapa.

Medidas adotadas:

- Dados CSV e tabelas Delta fora do GitHub.
- Controle de versão dos notebooks e documentação.
- Preservação dos arquivos Raw.
- Proteção contra sobrescrita nas cargas iniciais.
- Segredos e credenciais não devem ser publicados.

**Pendência de segurança:** confirmar a revogação do token GitHub anteriormente exposto.

## 10. Próximas entregas

1. Finalizar documentação e evidências da Bronze completa.
2. Abrir e revisar o Pull Request da segunda entrega.
3. Integrar à `main`, sincronizar o Databricks e encerrar a branch.
4. Construir a camada Silver com tipagem, deduplicação e qualidade.
5. Desenvolver a Gold com fatos, dimensões e indicadores.
6. Implementar orquestração com Lakeflow Jobs.
7. Evoluir testes, monitoramento e CI/CD.
8. Avaliar a integração real Databricks + AWS S3, com orçamento previamente aprovado.

## 11. Evidências técnicas

**Local previsto:** `docs/evidencias/`

**Registro:** `docs/evidencias/REGISTRO_EVIDENCIAS.md`

Evidências registradas até o momento:

| Nº | Evidência |
|---|---|
| 01 | Upload dos nove arquivos CSV |
| 02 | Leitura inicial de pedidos com PySpark |
| 03 | Criação da tabela Bronze Orders |
| 04 | Validação sem sobrescrita |
| 05 | Tabela Orders no Unity Catalog |
| 06 | Primeiro commit e push |
| 07 | Código e documentação na branch |
| 08 | Merge do PR #1 |
| 09 | Main selecionada no Databricks |
| 10 | Exclusão da branch após merge |
| 11 | Listagem dos nove arquivos Raw |
| 12 | Criação das oito tabelas adicionais |
| 13 | Validação do DataFrame corrigido de avaliações |
| 14 | Correção da tabela Delta `order_reviews` |
| 15 | Existência das nove tabelas Bronze |
| 16 | Contagens e formato Delta das nove tabelas |

As evidências 01–06 foram incluídas na primeira entrega. As evidências posteriores devem ser verificadas e incorporadas ao repositório antes do encerramento da segunda entrega.

---

**Conclusão técnica:** a camada Bronze do Olist está implementada e validada quanto à existência, formato Delta e quantidade de registros. As próximas etapas devem acrescentar qualidade formal, transformação e automação, sem tratar a carga inicial como uma solução de produção completa.
