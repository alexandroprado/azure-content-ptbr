<properties
   pageTitle="Usando o Conector do Office 365 em Aplicativos Lógicos | Serviço de Aplicativo do Microsoft Azure"
   description="Como criar e configurar o Conector do Office 365 ou o aplicativo de API e usá-lo em um aplicativo lógico no Serviço de Aplicativo do Azure"
   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="anuragdalmia"
   manager="erikre"
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="02/10/2016"
   ms.author="sameerch"/>


# Introdução ao Conector do Office 365 e à adição dele a seu Aplicativo Lógico
>[AZURE.NOTE] Esta versão do artigo aplica-se à versão do esquema 2014-12-01-preview de aplicativos lógicos.

Conecte-se à sua conta do Office 365 para enviar e receber emails e gerenciar seu calendário e seus contatos. Você pode executar várias ações, como enviar, receber e obter emails, criar e excluir eventos de seu calendário e criar, atualizar, obter e excluir seus contatos.

Aplicativos lógicos podem ser disparados com base em diversas fontes de dados e oferecem conectores para obter e processar dados como parte do fluxo. Você pode adicionar o Conector do Office 365 a seu fluxo de trabalho de negócios e processar dados como parte desse fluxo de trabalho dentro de um Aplicativo Lógico.

**Ações básicas**

- Novos Emails (Gatilho)
- Enviar Email
- Responder Email
- Enviar Evento
- Adicionar Contato

## Criar o aplicativo de API do conector O365
Um conector pode ser criado em um aplicativo lógico ou diretamente no Azure Marketplace. Para criar um conector no Marketplace:

1. No quadro inicial do Azure, selecione **Marketplace**.
2. Procure "Conector do Office 365", selecione-o e selecione **Criar**.
3.	Configure o Conector do Office 365 fornecendo os detalhes do Plano de Hospedagem e o grupo de recursos e selecionando o nome do aplicativo de API: ![][21]


## Criar um aplicativo lógico
Vamos criar um aplicativo lógico simples que é disparado quando um email é recebido (em sua ID de email de consulta de vendas - digamos sales@contoso.com). Ele cria um evento, adiciona um contato com os detalhes do remetente, envia um email à sua conta pessoal e, por fim, envia uma resposta com um reconhecimento.

1.	Entre no Portal do Azure e clique em ‘Novo -> Web + celular -> Aplicativo Lógico’: ![][1]

2.	Na página ‘Criar aplicativo lógico’, forneça os detalhes necessários, como nome, plano de serviço de aplicativo e local: ![][2]

3.	Clique em ‘Gatilhos e Ações’ e o editor do Aplicativo Lógico será aberto: ![][3]

4.	Selecione o gatilho do Office 365 na seção ‘Aplicativos de API neste grupo de recursos’ na galeria para adicioná-lo ao fluxo: ![][4]

6.	A conexão com o Office 365 exige que você autorize o aplicativo lógico a ser capaz de acessar sua conta. Clique em “Autorizar” para fornecer as credenciais de logon do Office 365: ![][5]

7.	Você será redirecionado à página de entrada do Office 365 e poderá se autenticar com as credenciais da sua conta do Office 365: ![][6] ![][7]

8.	Quando a autorização for concluída, os gatilhos do Office 365 serão exibidos: ![][8]

9.	Selecione o gatilho “Novo Email” e os parâmetros de entrada serão exibidos.


10.	Altere a frequência do gatilho para ‘Minutos’ e clique em ✓: ![][9]

11. O gatilho ‘Novo Email’ do Office 365 é configurado e você pode ver que os parâmetros de saída também são exibidos: ![][10]

12.	Selecione "Conector do Office 365" na seção “Usados Recentemente” da galeria e uma nova ação do "Office 365" é adicionada.

13.	Selecione ‘Evento de Envio’ na lista de ações e os parâmetros de entrada da ação ‘Evento de Envio’ serão exibidos: ![][11]

14.	Especifique os detalhes do evento e clique em ✓: ![][12]

15.	Selecione "Conector do Office 365" na seção “Usados Recentemente” da galeria e uma nova ação do "Office 365" é adicionada.

16.	Selecione ‘Adicionar Contato’ na lista de ações e os parâmetros de entrada da ação ‘Adicionar Contato’ serão exibidos: ![][13]

17.	Clique em '+' ao lado do campo ‘Endereço de Email’ e selecione o valor do campo de saída do gatilho: ![][14]

18. Clique em ✓ e a configuração da ação será concluída: ![][15]

19.	Selecione "Conector do Office 365" na seção “Usados Recentemente” da galeria e uma nova ação do "Office 365" é adicionada.


20.	Selecione ‘Enviar Email’ na lista de ações e os parâmetros de entrada da ação 'Enviar Email' serão exibidos: ![][19]

21.	Forneça os detalhes necessários para enviar o email. Você pode criar uma mensagem digitando algo como o que está abaixo. Uma vez que a ação estiver configurada, clique em ✓:

		Body - @concat('You got a new sales enquiry from',triggers().output.body.From)

	![][20]
22.	Selecione "Conector do Office 365" na seção “Usados Recentemente” da galeria e uma nova ação do "Office 365" é adicionada.


23.	Selecione ‘Responder’ na lista de ações e os parâmetros de entrada da ação ‘Responder’ serão exibidos: ![][16]

24.	Clique em '+' ao lado do campo e selecione o valor da ID da mensagem de saída do gatilho e clique em ✓: ![][17]

25. Clique em OK na tela do editor do aplicativo lógico e clique em “Criar”. Serão necessários cerca de 30 segundos para a conclusão da criação.

26. Envie um email para a conta em que você configurou o gatilho e verá um email na sua conta de email pessoal e um evento de calendário e contato em sua conta de email profissional. Além disso, você deve obter uma resposta confirmando que a consulta de venda será respondida em breve.

## Faça mais com seu Conector
Agora que o conector foi criado, você pode adicioná-lo a um fluxo de trabalho comercial usando um Aplicativo Lógico. Consulte [O que são Aplicativos Lógicos?](app-service-logic-what-are-logic-apps.md).

> [AZURE.NOTE] Se você deseja começar com os Aplicativos Lógicos do Azure antes de se inscrever em uma conta do Azure, acesse [Experimentar os Aplicativos Lógicos](https://tryappservice.azure.com/?appservice=logic), em que você pode criar imediatamente um aplicativo lógico inicial de curta duração no Serviço de Aplicativo. Não é necessário nenhum cartão de crédito; não há compromissos.

Exibir a referência da API REST de Swagger em [Conectores e referência de aplicativos de API](http://go.microsoft.com/fwlink/p/?LinkId=529766).

Você também pode examinar estatísticas de desempenho e controlar a segurança do conector. Consulte [Gerenciar e monitorar Aplicativos de API e conectores internos](app-service-logic-monitor-your-connectors.md).

<!--Image references-->
[1]: ./media/app-service-logic-connector-office365/1_New_Logic_App.png
[2]: ./media/app-service-logic-connector-office365/2_Logic_App_Settings.png
[3]: ./media/app-service-logic-connector-office365/3_Logic_App_Editor.png
[4]: ./media/app-service-logic-connector-office365/4_Select_Office365_Gallery.png
[5]: ./media/app-service-logic-connector-office365/5_Office365_Authorize.png
[6]: ./media/app-service-logic-connector-office365/6_Office365_Login.png
[7]: ./media/app-service-logic-connector-office365/7_Office365_User_Consent.png
[8]: ./media/app-service-logic-connector-office365/8_Office365_Trigger.png
[9]: ./media/app-service-logic-connector-office365/9_Office365_Trigger_Settings.png
[10]: ./media/app-service-logic-connector-office365/10_Office365_Trigger_Configured.png
[11]: ./media/app-service-logic-connector-office365/11_Office365_Actions_List.png
[12]: ./media/app-service-logic-connector-office365/12_Office365_Create_Event_Inputs.png
[13]: ./media/app-service-logic-connector-office365/13_Office365_Add_Contact_Inputs.png
[14]: ./media/app-service-logic-connector-office365/14_Office365_Add_Contact_Email_FromTrigger.png
[15]: ./media/app-service-logic-connector-office365/15_Office365_Add_Contacts_Configured.png
[16]: ./media/app-service-logic-connector-office365/16_Office365_Reply_To_Inputs.png
[17]: ./media/app-service-logic-connector-office365/17_Office365_Reply_To_MessageId.png
[18]: ./media/app-service-logic-connector-office365/18_Office365_Reply_To_Configured.png
[19]: ./media/app-service-logic-connector-office365/19_Office365_Send_Inputs.png
[20]: ./media/app-service-logic-connector-office365/20_Office365_Send_Configured.png
[21]: ./media/app-service-logic-connector-office365/21-create-new-o365-api-app.png

<!---HONumber=AcomDC_0224_2016-->