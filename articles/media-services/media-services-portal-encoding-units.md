<properties
	pageTitle="Como escalar o processamento de mídia usando o Portal Clássico do Azure"
	description="Saiba como dimensionar os Serviços de Mídia especificando o número de Unidades Reservadas para Streaming por Demanda e Unidades Reservadas para Codificação com as quais você deseja provisionar sua conta."
	services="media-services"
	documentationCenter=""
	authors="juliako,milangada"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/29/2016"
	ms.author="juliako"/>


# Como escalar o processamento de mídia usando o Portal Clássico do Azure

> [AZURE.SELECTOR]
- [.NET](media-services-dotnet-encoding-units.md)
- [Portal](media-services-portal-encoding-units.md)
- [REST](https://msdn.microsoft.com/library/azure/dn859236.aspx)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

## Visão geral

Uma conta dos Serviços de Mídia é associada a um Tipo de Unidade Reservada que determina a velocidade com que as suas tarefas de processamento de mídia são processadas. Você pode escolher entre os seguintes tipos de unidade reservada: **S1**, **S2** ou **S3**. Por exemplo, o mesmo trabalho de codificação é executado mais rápido quando se usa o tipo de unidade reservada **S2** em comparação ao tipo **S1**. Para saber mais, consulte [Tipos de unidade reservada para codificação](https://azure.microsoft.com/blog/author/milanga/).

Além de especificar o tipo de unidade reservada, você pode especificar para provisionar sua conta com unidades reservadas para codificação. O número de unidades reservadas para codificação provisionada determina o número de tarefas de mídia que podem ser processadas simultaneamente em uma determinada conta. Por exemplo, se sua conta tiver cinco unidades reservadas, as cinco tarefas de mídia serão executadas simultaneamente enquanto houver tarefas para serem processadas. As tarefas restantes irão aguardar na fila e serão selecionadas para processamento sequencialmente assim que uma tarefa em execução seja concluída. Se uma conta não tiver nenhuma unidade reservada provisionada, as tarefas serão selecionadas sequencialmente. Nesse caso, o tempo de espera entre a conclusão de uma tarefa e o início da próxima dependerá da disponibilidade dos recursos do sistema.

>[AZURE.IMPORTANT] As Unidades Reservadas trabalham para paralelizar todo o processamento de mídia, incluindo trabalhos de indexação usando o Indexador de Mídia do Azure. No entanto, ao contrário da codificação, a indexação de trabalhos não processará mais rapidamente com unidades reservadas mais rápidas.

Para alterar o tipo de unidade reservada e o número de unidades reservadas para codificação, faça o seguinte:

1. No [Portal Clássico do Azure](https://manage.windowsazure.com/), clique em **Serviços de Mídia**. Em seguida, clique no nome do serviço de mídia.

2. Selecione a página **CODIFICAÇÃO**.

	Para alterar o **TIPO DE UNIDADE RESERVADA**, pressione S1, S2 ou S3.

	Para alterar o número de unidades reservadas para o tipo de unidade reservada selecionado, use controle deslizante **CODIFICAÇÃO**.


	![Página Processadores](./media/media-services-portal-encoding-units/media-services-encoding-scale.png)


	>[AZURE.NOTE] Os datacenters a seguir não oferecem o tipo de unidade reservada Premium: Singapura, Hong Kong, Osaka, Pequim, Xangai.

3. Pressione o botão SALVAR para salvar as alterações.

	As novas unidades reservadas para codificação são alocadas assim que você pressiona SALVAR.

	>[AZURE.NOTE] O número mais alto de unidades especificadas para o período de 24 horas é usado para calcular o custo.

##Cotas e limitações

Para saber mais sobre as cotas e limitações e sobre como abrir um tíquete de suporte, consulte [Cotas e limitações](media-services-quotas-and-limitations.md).



##Roteiros de aprendizagem dos Serviços de Mídia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornecer comentários

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0204_2016-->