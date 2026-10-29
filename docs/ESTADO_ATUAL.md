# ESTADO ATUAL — P04 Lakehouse Olist Databricks + AWS

**Atualizado:** 28/10/2026 (horário de Brasília)  
**Branch:** `feature/estrutura-inicial`  
**Situação:** Bronze de pedidos validada; documentação/evidências preparadas para inclusão na branch; PR pendente.

## Entregas comprovadas
- Git folder ligada ao repositório GitHub.
- 9 CSVs enviados para `workspace.default.olist_raw`.
- `olist_orders_dataset.csv` lido via PySpark: 99.441 linhas, 8 campos.
- Tabela gerenciada Delta criada: `workspace.bronze.orders`, com 99.441 linhas.
- Reexecução com tabela existente evita nova gravação.
- Notebook `notebooks/Bronze Orders Pipeline.ipynb` commitado e enviado à branch, 1 commit à frente da `main` no momento da captura.
- Evidências 01–06 disponíveis em `docs/evidencias/` neste pacote, ainda não enviadas ao GitHub.

## Próxima ação
Enviar `docs/` para a branch `feature/estrutura-inicial` pelo GitHub, conferir arquivos e só então realizar commit/PR. Não fazer alterações diretamente na `main`.

## Pendências técnicas
- Bronze para os outros oito CSVs.
- Tipagem, qualidade, deduplicação e modelagem Silver/Gold.
- Jobs Lakeflow, testes, observabilidade e CI/CD.
- Avaliar futura integração S3/AWS sob autorização e controle de custos.

## Segurança
Confirmar a revogação de token GitHub exposto anteriormente (ainda não comprovada). Não versionar segredos.
