<properties
	pageTitle="Adicione a API do Microsoft Translator ao PowerApps Enterprise | Microsoft Azure"
	description="Crie ou configure uma nova API do Microsoft Translator no ambiente de serviço de aplicativo da sua organização"
	services=""
    suite="powerapps"
	documentationCenter="" 
	authors="linhtranms"
	manager="dwrede"
	editor=""/>

<tags
   ms.service="powerapps"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na" 
   ms.date="11/25/2015"
   ms.author="litran"/>

# Crie uma nova API do Microsoft Translator no ambiente de serviço de aplicativo da sua organização

## Crie a API no portal do Azure

1. No [Portal do Azure](https://portal.azure.com/), entre com sua conta de trabalho. Por exemplo, entre com *Seunomedeusuário*@* SuaEmpresa*.com. Quando fizer isso, entrará automaticamente na assinatura de sua empresa.
 
2. Selecione **Procurar** na barra de tarefas: ![][7]

3. Na lista, você pode usar a barra de rolagem para localizar o PowerApps ou digitar *powerapps*: ![][8]

4. Em **PowerApps**, selecione **Gerenciar APIs**: ![Navegue até as APIs registradas][1]

5. Em **Gerenciar APIs**, selecione **Adicionar** para adicionar a nova API: ![Adicionar API][2]

6. Insira um **nome** descritivo para sua API.
	
7. Em **Fonte**, selecione as **APIs disponíveis** para selecionar as APIs criadas previamente e selecione **Microsoft Translator**: ![selecione a api do Microsoft Translator][3]

8. Selecione **Configurações - Definir as configurações necessárias**: ![definir configurações de API do Microsoft Translator][4]

9. Insira o *Id do Cliente* e *Segredo do Cliente* do seu aplicativo Microsoft Translator. Se não tiver um, consulte a seção "Registrar um aplicativo do Microsoft Translator para uso com o PowerApps" neste tópico para criar a ID e os valores secretos necessários.

9. Selecione **OK** para concluir as etapas.

Quando terminar, uma nova API do Microsoft Translator é adicionada ao seu ambiente de serviço de aplicativo.


## Opcional: registre um aplicativo do Microsoft Translator para uso com PowerApps

Se não tiver um aplicativo do Microsoft Translator existente com os valores do segredo e ID, então use as seguintes etapas para criar o aplicativo e obter os valores necessários.


1. Vá para [Página do desenvolvedor do Azure Data Market][5] e entre com sua Conta da Microsoft. 

2. Selecione **Registrar**.

3. Em **Registrar seu aplicativo**:

	a) Insira um valor para **Id do Cliente**. b) Digite o **nome** do seu aplicativo. c) Digite um valor fictício para a **url de redirecionamento**. Por exemplo, insira **https://contosoredirecturl*. d) Insira uma **descrição**. e) Selecione **Criar**.

	![Registre seu aplicativo][6]

É criado um novo aplicativo Microsoft Translator. Você pode usar esse aplicativo em sua configuração da API do Microsoft Translator no portal do Azure.


## Resumo e próximas etapas
Neste tópico, você adicionou a API do Microsoft Translator para o seu PowersApps Enterprise. Em seguida, disponibilize aos usuários acesso à API para que ela possa ser adicionada aos seus aplicativos:

[Adicione uma conexão e disponibilize acesso aos usuários](powerapps-manage-api-connection-user-access.md)


<!--References-->
[1]: ./media/powerapps-create-api-microsofttranslator/browse-to-registered-apis.PNG
[2]: ./media/powerapps-create-api-microsofttranslator/add-api.PNG
[3]: ./media/powerapps-create-api-microsofttranslator/select-microsofttranslator-api.PNG
[4]: ./media/powerapps-create-api-microsofttranslator/configure-microsofttranslator-api.PNG
[5]: https://datamarket.azure.com/developer/applications/
[6]: ./media/powerapps-create-api-microsofttranslator/register-your-application.PNG
[7]: ./media/powerapps-create-api-microsofttranslator/browseall.png
[8]: ./media/powerapps-create-api-microsofttranslator/allresources.png

<!---HONumber=AcomDC_1203_2015-->