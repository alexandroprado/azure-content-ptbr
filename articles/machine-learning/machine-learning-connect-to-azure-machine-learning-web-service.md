<properties
	pageTitle="Conectar a um Serviço Web de Aprendizado de Máquina | Microsoft Azure"
	description="Com C# ou Python, conecte a um serviço Web de Aprendizado de Máquina do Azure usando uma chave de autorização."
	services="machine-learning"
	documentationCenter=""
	authors="garyericson"
	manager="paulettm"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/03/2016" 
	ms.author="garye" />


# Conectar a um Serviço Web de Aprendizado de Máquina do Azure
A experiência do desenvolvedor de Aprendizado de Máquina do Azure é uma API de serviço Web para fazer previsões de dados de entrada em tempo real ou em modo de lote. Você pode usar o Estúdio de Aprendizado de Máquina do Azure para criar previsões e implantar um serviço Web do Aprendizado de Máquina do Azure.

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

Para saber como criar e implantar um serviço Web do Aprendizado de Máquina usando o Estúdio de Aprendizado de Máquina:

- [Implantar um serviço Web do Aprendizado de Máquina](machine-learning-publish-a-machine-learning-web-service.md)
- [Introdução ao Estúdio de Aprendizado de Máquina](https://azure.microsoft.com/documentation/videos/getting-started-with-ml-studio/)
- [Visualização de Aprendizado de Máquina do Azure](https://studio.azureml.net/)
- [Centro de Documentação do Aprendizado de Máquina](https://azure.microsoft.com/documentation/services/machine-learning/)

## Serviço Web de Aprendizado de Máquina do Azure ##

Com o serviço Web de Aprendizado de Máquina do Azure, um aplicativo externo se comunica com um modelo de pontuação do fluxo de trabalho de Aprendizado de Máquina em tempo real. Uma chamada do serviço Web de Aprendizado de Máquina retorna resultados de previsão para um aplicativo externo. Para fazer uma chamada de serviço Web de Aprendizado de Máquina, transmita uma chave de API que é criada quando você implanta uma previsão. O serviço Web de Aprendizado de Máquina baseia-se em REST, uma opção popular de arquitetura para projetos de programação da Web.

O Aprendizado de Máquina do Azure tem dois tipos de serviços:

- Serviço de Solicitação-Resposta (RRS) – Um serviço de baixa latência e altamente escalonável que fornece uma interface para os modelos sem monitoração de estado criados e implantados no Estúdio de Aprendizado de Máquina.
- Serviço de Execução de Lote (BES) – Um serviço assíncrono que pontua um lote de registros de dados.

Para saber mais sobre os serviços Web de Aprendizado de Máquina, confira [Implantar um serviço Web do Aprendizado de Máquina](machine-learning-publish-a-machine-learning-web-service.md).

## Obtenha uma chave de autorização de Aprendizado de Máquina do Azure ##
Você pode obter uma chave de API do serviço Web de um serviço Web de Aprendizado de Máquina. Você pode obtê-la no Estúdio de Aprendizado de Máquina no Portal do Azure.
### Estúdio de Aprendizado de Máquina ###
1. No Estúdio de Aprendizado de Máquina, clique em **SERVIÇOS WEB** à esquerda.
2. Clique em um serviço Web. A "chave de API" está na guia **PAINEL**.

### Portal do Azure ###

1. Clique em **APRENDIZADO DE MÁQUINA** à esquerda.
2. Clique em um espaço de trabalho.
3. Clique em **SERVIÇOS WEB**.
4. Clique em um serviço Web.
5. Clique em um ponto de extremidade. A “CHAVE DE API” está mais abaixo na parte inferior direita.

## <a id="connect"></a>Conectar-se a um serviço Web do Aprendizado de Máquina

Você pode se conectar a um serviço Web de Aprendizado de Máquina usando qualquer linguagem de programação que dá suporte à resposta e solicitação HTTP. Você pode exibir exemplos em C#, Python e R de uma página de Ajuda do serviço Web de Aprendizado de Máquina.

### Para exibir uma página de ajuda de API do serviço Web de Aprendizado de Máquina ###
Uma página de ajuda de API do Aprendizado de Máquina é criada quando você implanta um serviço Web. Confira [Passo a passo do Aprendizado de Máquina do Azure – Implantar serviço Web](machine-learning-walkthrough-5-publish-web-service.md).


**Para exibir uma página de Ajuda de API do Aprendizado de Máquina** no Estúdio de Aprendizado de Máquina:

1. Escolha **SERVIÇOS WEB**.
2. Escolha um serviço Web.
3. Escolha a **página de Ajuda da API** - **SOLICITAÇÃO/RESPOSTA** ou **EXECUÇÃO EM LOTES**.


**Página de ajuda de API do Aprendizado de Máquina** A página de ajuda da API do Aprendizado de Máquina contém detalhes sobre um serviço Web de previsão.



### Exemplo de C# ###

Para se conectar a um serviço Web do Aprendizado de Máquina, use um **HttpClient** passando ScoreData. ScoreData contém um FeatureVector, um vetor com n dimensões de recursos numéricos que representa o ScoreData. Autentique no serviço de Aprendizado de Máquina com uma chave de API.

Para se conectar a um serviço Web de Aprendizado de Máquina, o pacote de Nuget **Microsoft.AspNet.WebApi.Client** deve ser instalado.

**Instalar o Nuget Microsoft.AspNet.WebApi.Client no Visual Studio**

1. Publique “Baixe o conjunto de dados de Download de UCI: Serviço Web do conjunto de dados da classe Adulto 2”.
2. Clique em **Ferramentas** > **Gerenciador de Pacotes do NuGet** > **Console do Gerenciador de Pacotes**.
2. Escolha **Install-Package Microsoft.AspNet.WebApi.Client**.

**Para executar o exemplo de código**

1. Publique o experimento “Exemplo 1: Baixe o conjunto de dados de UCI: conjunto de dados da classe Adulto 2”, parte da coleção de exemplos de Aprendizado de Máquina.
2. Atribua apiKey com a chave de um serviço Web. Confira **Obter uma chave de autorização de Aprendizado de Máquina do Azure** acima.
3. Atribua serviceUri com o URI de solicitação.


### Exemplo de Python ###

Para se conectar a um serviço Web de Aprendizado de Máquina, use a biblioteca **urllib2** passando ScoreData. ScoreData contém um FeatureVector, um vetor com n dimensões de recursos numéricos que representa o ScoreData. Autentique no serviço de Aprendizado de Máquina com uma chave de API.


**Para executar o exemplo de código**

1. Publique o experimento “Exemplo 1: Baixe o conjunto de dados de UCI: conjunto de dados da classe Adulto 2”, parte da coleção de exemplos de Aprendizado de Máquina.
2. Atribua apiKey com a chave de um serviço Web. Confira **Obter uma chave de autorização de Aprendizado de Máquina do Azure** acima.
3. Atribua serviceUri com o URI de solicitação. Veja como obter um URI de solicitação.

<!---HONumber=AcomDC_0204_2016-->