<properties
   pageTitle="Criar soluções integradas com o SQL Data Warehouse | Microsoft Azure"
   description="Ferramentas e parceiros com soluções que podem ser integradas ao SQL Data Warehouse"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="02/01/2016"
   ms.author="lodipalm;barbkess;sonyama"/>

#Aproveitar outros serviços com o SQL Data Warehouse
Além de suas funcionalidades principais, o SQL Data Warehouse permite aos usuários aproveitar muitos dos outros serviços no Azure juntamente com ele. Especificamente, atualmente tomamos medidas para maior integração com o seguinte:

+ Power BI
+ Fábrica de dados do Azure
+ Aprendizado de Máquina do Azure
+ Stream Analytics do Azure

Estamos trabalhando para nos conectar com mais serviços em todo o ecossistema do Azure.

##Power BI
A integração do Power BI permite que você aproveite o poder de computação do SQL Data Warehouse com a e visualização e os relatórios dinâmicos do Power BI. Atualmente, a integração do Power BI inclui:

+ **Conexão direta**: uma conexão mais avançada com a aplicação de lógica no SQL Data Warehouse. Isso fornece uma análise mais rápida em uma escala maior.
+ **Abrir no Power BI**: o botão 'Abrir no Power BI' passa informações de instância para Power BI, permitindo uma conexão contínua. 

Consulte [Integrar com o Power BI](./sql-data-warehouse-integrate-power-bi.md) ou [Documentação do Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/06/24/exploring-azure-sql-data-warehouse-with-power-bi.aspx) para obter mais informações.

##Fábrica de dados do Azure
O Azure Data Factory oferece aos usuários uma plataforma gerenciada ao criar pipelines complexos de Extração-Carregamento. A integração do SQL Data Warehouse com o Azure Data Factory inclui o seguinte:

+ **Procedimentos Armazenados**: orquestre a execução de procedimentos armazenados no SQL Data Warehouse.

Confira [Integrar com o Azure Data Factory](./sql-data-warehouse-integrate-azure-data-factory.md) ou a [Documentação do Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) para obter mais informações.

##Aprendizado de Máquina do Azure
O Aprendizado de Máquina do Azure é um serviço de análise totalmente gerenciado que permite aos usuários criar modelos complexos utilizando um grande conjunto de ferramentas de previsão. O SQL Data Warehouse tem suporte como uma origem e um destino para esses modelos com a seguinte funcionalidade:

+ **Ler Dados:** encaminhe modelos em escala usando o T-SQL no SQL Data Warehouse. 
+ **Gravar Dados:** confirme as alterações de qualquer modelo de volta para o SQL Data Warehouse.

Veja [Integrar com o Aprendizado de Máquina do Azure](./sql-data-warehouse-integrate-azure-machine-learning.md) ou a [Documentação do Aprendizado de Máquina do Azure](https://azure.microsoft.com/services/machine-learning/) para obter mais informações.

##Stream Analytics do Azure
O Stream Analytics do Azure é uma infraestrutura complexa, totalmente gerenciada para processamento e consumo de dados de eventos gerados por meio do hub de eventos do Azure. A integração com o SQL Data Warehouse permite transmitir dados para serem efetivamente processados e armazenados junto com dados relacionais, permitindo análise mais profunda e avançada.

+ **Saída de Trabalho:** envie a saída dos trabalhos do Stream Analytics diretamente para o SQL Data Warehouse.

Veja [Integrar com o Stream Analytics do Azure](./sql-data-warehouse-integrate-azure-stream-analytics.md) ou a [documentação do Stream Analytics do Azure](https://azure.microsoft.com/documentation/services/stream-analytics/) para obter mais informações.

<!--Image references-->

<!--Article references-->
[development overview]: sql-data-warehouse-overview-develop/

[Azure Data Factory]: sql-data-warehouse-integrate-azure-data-factory.md
[Azure Machine Learning]: sql-data-warehouse-integrate-azure-machine-learning.md
[Azure Stream Analytics]: sql-data-warehouse-integrate-azure-stream-analytics.md
[Power BI]: sql-data-warehouse-integrate-power-bi.md
[Partners]: sql-data-warehouse-integrate-solution-partners.md

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=AcomDC_0204_2016-->