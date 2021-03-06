A Unidade de Transação de Banco de Dados (DTU) é a unidade de medida no Banco de Dados SQL que representa a potência relativa dos bancos de dados com base em uma medida real: a transação do banco de dados. Usamos um conjunto de operações típicas para uma solicitação OLTP (processamento de transação online) e calculamos quantas transações foram concluídas por segundo sob condições de carga total (essa é a versão curta, você pode ler os detalhes assustadores na [Visão geral de parâmetro de comparação](../articles/sql-database/sql-database-benchmark-overview.md)).

Um banco de dados Básico tem cinco DTUs, o que significa que ele pode concluir cinco transações por segundo, enquanto um banco de dados Premium P11 tem 1750 DTUs.

![Introdução ao Banco de Dados SQL: DTUs de banco de dados individual por camada e por nível.](./media/sql-database-understanding-dtus/single_db_dtus.png)

>[AZURE.NOTE]Se você estiver migrando um banco de dados do SQL Server existente, poderá usar uma ferramenta de terceiros, [a Calculadora de DTU do Banco de Dados SQL do Azure](http://dtucalculator.azurewebsites.net/), para obter uma estimativa do nível de desempenho e a camada de serviço que seu banco de dados poderá exibir no Banco de Dados SQL do Azure.

### DTU versus eDTU

DTU para bancos de dados individuais é o mesmo que eDTU para bancos de dados elásticos. Por exemplo, um banco de dados em um pool de bancos de dados elásticos Basic oferece até cinco eDTUs. Esse é o mesmo desempenho de um banco de dados individual Basic. A diferença é que o banco de dados elástico não consumirá eDTUs do pool até que isso seja necessário.

![Introdução ao Banco de Dados SQL: pools elásticos por camada.](./media/sql-database-understanding-dtus/sqldb_elastic_pools.png)

Um exemplo simples pode ajudar: pegue um pool de banco de dados elástico Basic com 1000 DTUs e solte 800 bancos de dados nele. Contanto que apenas 200 bancos de dados, do total de 800, estejam sendo usados a qualquer momento (5 DTU X 200 = 1000), você não atingirá a capacidade do pool, e o desempenho do banco de dados não será afetado. Este exemplo foi simplificado para manter a clareza. O cálculo real é um pouco mais detalhado. O portal faz os cálculos para você e faz também uma recomendação com base no histórico de uso do banco de dados. Confira [Considerações de preço e desempenho para um pool de banco de dados elástico](../articles/sql-database/sql-database-elastic-pool-guidance.md) para aprender como as recomendações funcionam ou para fazer os cálculos você mesmo.

<!---HONumber=AcomDC_1223_2015-->