<properties
	pageTitle="Visualização do Active Directory B2C do Azure: estrutura extensível de política Microsoft Azure"
	description="Um tópico sobre a estrutura de política extensível do Active Directory B2C do Azure e como criar vários tipos de política"
	services="active-directory-b2c"
	documentationCenter=""
	authors="swkrish"
	manager="msmbaldwin"
	editor="bryanla"/>

<tags
	ms.service="active-directory-b2c"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/28/2016"
	ms.author="swkrish"/>

# Visualização do Azure Active Directory B2C: estrutura de política extensível

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-preview-note.md)]

## Noções básicas

A estrutura de política extensível do Azure AD (Azure Active Directory) B2C é o principal ponto forte do serviço. As políticas descrevem totalmente as experiências de identidade do consumidor, como inscrição, credenciais ou edição de perfil. Por exemplo, uma política de inscrição permite controlar comportamentos definindo as seguintes configurações:

- Tipos de conta (contas sociais, como o Facebook, ou contas locais como um endereço de email) que os consumidores podem usar para se inscrever no aplicativo.
- Atributos (como nome, CEP e tamanho do calçado, por exemplo) que serão coletados junto ao consumidor durante a inscrição.
- Uso da Multi-Factor Authentication.
- A aparência e funcionalidade de todas as páginas de inscrição.
- Informações (que se manifestam como declarações em um token) que o aplicativo recebe quando a execução da política é concluída.

Você pode criar várias políticas de tipos diferentes em seu locatário e usá-las em seus aplicativos, conforme necessário. As políticas podem ser reutilizadas entre os aplicativos. Isso permite aos desenvolvedores definir e modificar experiências de identidade do consumidor com pouca ou nenhuma alteração no código.

As políticas estão disponíveis para uso por meio de uma interface simples do desenvolvedor. O aplicativo dispara uma política usando uma solicitação de autenticação HTTP padrão (passando um parâmetro de política na solicitação) e recebe um token personalizado como resposta. Por exemplo, a única diferença entre solicitações que invocam uma política de inscrição e aquelas que invocam uma política de credenciais é o nome da política usado no parâmetro de cadeia de caracteres de consulta "p":

```

https://login.microsoftonline.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siup                                       // Your sign-up policy

```

```

https://login.microsoftonline.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siin                                       // Your sign-in policy

```

Para obter mais detalhes sobre a estrutura de políticas, consulte esta [postagem do blog](http://blogs.technet.com/b/ad/archive/2015/11/02/a-look-inside-azuread-b2c-with-kim-cameron.aspx).

## Criar uma política de inscrição

Para habilitar a inscrição no seu aplicativo, você precisará criar uma política de inscrição. Essa política descreve as experiências pelas quais os consumidores passarão durante a inscrição e o conteúdo dos tokens que o aplicativo receberá de inscrições bem-sucedidas.

1. [Siga estas etapas para navegar até a folha de recursos do B2C no Portal do Azure](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade).
2. Clique em **Políticas de inscrição**.
3. Clique em **+Adicionar**, na parte superior da folha.
4. O **Nome** determina o nome da política de inscrição usado pelo seu aplicativo. Por exemplo, insira "SiUp".
5. Clique em **Provedores de identidade** e selecione "Endereço de email". Opcionalmente, você também pode selecionar provedores de identidade social, se já configurado. Clique em **OK**.
6. Clique em **Atributos de inscrição**. Escolha aqui os atributos do consumidor que você deseja coletar durante a inscrição. Por exemplo, selecione "Cidade/região", "Nome de exibição" e "CEP". Clique em **OK**.
7. Clique em **Declarações do aplicativo**. Escolha aqui as declarações que você deseja que sejam retornadas nos tokens enviados ao aplicativo após uma experiência de inscrição bem-sucedida. Por exemplo, selecione "Nome de exibição", "Provedor de identidade", "CEP", "Novo usuário" e "ID de objeto do usuário".
8. Clique em **Criar**. Observe que a política recém-criada aparece como "**B2C\_1\_SiUp**" (o fragmento **B2C\_1\_** é adicionado automaticamente) na folha **Políticas de inscrição**.
9. Abra a política clicando em "**B2C\_1\_SiUp**".
10. Selecione "aplicativo Contoso B2C” na lista suspensa **Aplicativos** e `https://localhost:44321/` na lista suspensa **URL de Resposta/URI de redirecionamento**.
11. Clique em **Executar agora**. Uma nova guia do navegador se abre e você poderá percorrer a experiência do consumidor de inscrição no aplicativo.

    > [AZURE.NOTE]
    Leva até um minuto para a criação de políticas e as atualizações entrem em vigor.

## Usar uma política de entrada

Para habilitar a entrada no aplicativo, você precisará criar uma política de entrada. Essa política descreve as experiências pelas quais os consumidores passarão durante a entrada e o conteúdo de tokens que o aplicativo receberá de entradas bem-sucedidas.

1. [Siga estas etapas para navegar até a folha de recursos do B2C no Portal do Azure](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade).
2. Clique em **Políticas de entrada**.
3. Clique em **+Adicionar**, na parte superior da folha.
4. O **Nome** determina o nome da política de entrada usado pelo seu aplicativo. Por exemplo, insira "SiIn".
5. Clique em **Provedores de identidade** e selecione "Endereço de email". Opcionalmente, você também pode selecionar provedores de identidade social, se já configurado. Clique em **OK**.
6. Clique em **Declarações do aplicativo**. Escolha aqui as declarações que você deseja que sejam retornadas dos tokens enviados ao aplicativo após uma experiência de entrada bem-sucedida. Por exemplo, selecione "Nome de exibição", "Provedor de identidade", "CEP" e "ID de objeto do usuário". Clique em **OK**.
7. Clique em **Criar**. Observe que a política recém-criada aparece como "**B2C\_1\_SiIn**" (o fragmento **B2C\_1\_** torna-se automaticamente pré-pendente) na folha **Políticas de entrada**.
8. Abra a política clicando em "**B2C\_1\_SiIn**".
9. Selecione "aplicativo Contoso B2C” na lista suspensa **Aplicativos** e `https://localhost:44321/` na lista suspensa **URL de Resposta/URI de redirecionamento**.
10. Clique em **Executar agora**. Uma nova guia do navegador se abre e você poderá percorrer a experiência do consumidor de entrada no aplicativo.

    > [AZURE.NOTE]
    Leva até um minuto para a criação de políticas e as atualizações entrem em vigor.

## Criar uma política de edição de perfil

Para habilitar a edição de perfil no aplicativo, você precisará criar uma política de edição de perfil. Essa política descreve as experiências pelas quais os consumidores passarão durante a edição do perfil e o conteúdo de tokens que o aplicativo receberá na conclusão bem-sucedida.

1. [Siga estas etapas para navegar até a folha de recursos do B2C no Portal do Azure](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade).
2. Clique em **Políticas de edição de perfil**.
3. Clique em **+Adicionar**, na parte superior da folha.
4. O **Nome** determina o nome da política de edição de perfil usado pelo seu aplicativo. Por exemplo, insira "SiPe".
5. Clique em **Provedores de identidade** e selecione "Endereço de email". Opcionalmente, você também pode selecionar provedores de identidade social, se já configurado. Clique em **OK**.
6. Clique em **Atributos do perfil**. Aqui você escolhe os atributos que o consumidor pode exibir e editar. Por exemplo, selecione "País/Região", "Nome de Exibição" e "CEP". Clique em **OK**.
7. Clique em **Declarações do aplicativo**. Aqui você escolhe as declarações que quer retornar dos tokens de volta ao aplicativo após uma experiência de edição de perfil bem-sucedida. Por exemplo, selecione "Nome de exibição" e "CEP".
8. Clique em **Criar**. Observe que a política recém-criada aparece como "**B2C\_1\_SiPe**" (o fragmento **B2C\_1\_** é adicionado automaticamente) na folha **Políticas de edição de perfil**.
9. Abra a política clicando em "**B2C\_1\_SiPe**".
10. Selecione "aplicativo Contoso B2C” na lista suspensa **Aplicativos** e `https://localhost:44321/` na lista suspensa **URL de Resposta/URI de redirecionamento**.
11. Clique em **Executar agora**. Uma nova guia do navegador se abre e você pode percorrer a experiência do consumidor de edição de perfil no aplicativo.

    > [AZURE.NOTE]
    Leva até um minuto para a criação de políticas e as atualizações entrem em vigor.

<!---HONumber=AcomDC_0224_2016-->