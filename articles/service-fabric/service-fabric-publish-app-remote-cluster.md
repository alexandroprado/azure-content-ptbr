<properties
    pageTitle="Publicar um aplicativo em um cluster remoto com o Visual Studio | Microsoft Azure"
    description="Saiba mais sobre como publicar um aplicativo em um cluster do Service Fabric remoto usando o Visual Studio."
    services="service-fabric"
    documentationCenter="na"
    authors="cawams"
    manager="timlt"
    editor="" />

<tags
    ms.service="multiple"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="multiple"
    ms.date="12/06/2015"
    ms.author="cawa" />

# Publicar um aplicativo em um cluster remoto usando o Visual Studio

A extensão do Service Fabric do Azure para Visual Studio permite facilmente repetir e executar por script a publicação de um aplicativo em um cluster do Service Fabric.

## Os artefatos necessários para publicação

### Deploy-FabricApplication.ps1

Este é um script do PowerShell que usa um caminho de perfil de publicação como um parâmetro para a publicação de aplicativos do Service Fabric. Como esse script faz parte do seu aplicativo, fique à vontade para modificá-lo como achar melhor para ele.

### Perfis de publicação

Uma pasta no projeto de aplicativo do Service Fabric chamada **PublishProfiles** contém arquivos XML que armazenam informações essenciais à publicação de aplicativos, como:

- Parâmetros de conexão de cluster do Service Fabric
- Caminho para um arquivo de parâmetros de aplicativo
- Configurações de atualização

Por padrão, o aplicativo incluirá dois perfis de publicação: Local.xml e Cloud.xml. Você pode adicionar mais perfis copiando e colando um dos arquivos padrão.

### Arquivos de parâmetros de aplicativo

Uma pasta no projeto de aplicativo do Service Fabric chamada **ApplicationParameters** contém arquivos XML com valores de parâmetro do manifesto do aplicativo especificados pelo usuário. Os arquivos de manifesto do aplicativo podem ser parametrizados, de modo que você possa usar valores diferentes para configurações de implantação. Para saber mais sobre a parametrização de aplicativo, confira [Gerenciar vários ambientes no Service Fabric](service-fabric-manage-multiple-environment-app-configuration.md).

>[AZURE.NOTE]Para serviços de ator, você deve compilar o projeto antes de tentar editar o arquivo em um editor ou por meio da caixa de diálogo de publicação. Isso ocorre porque parte dos arquivos de manifesto serão gerados durante a compilação.

## Publicar um aplicativo usando a caixa de diálogo Publicar Aplicativo do Service Fabric

As etapas a seguir demonstram como publicar um aplicativo usando a caixa de diálogo **Publicar Aplicativo do Service Fabric** fornecida pelas Ferramentas do Service Fabric do Visual Studio.

1. No menu de atalho do projeto de Aplicativo do Service Fabric, escolha **Publicar...** para exibir a caixa de diálogo **Publicar Aplicativo do Service Fabric**.

    ![A caixa de diálogo **Publicar Aplicativo do Service Fabric**][0]

    O arquivo selecionado na lista suspensa **Perfil de destino** é onde todas as configurações, exceto as **Versões do manifesto**, são salvas. Você pode reutilizar um perfil existente ou criar um novo escolhendo **<Gerenciar Perfis... >** na lista suspensa **Perfil de destino**. Quando você escolhe um perfil de publicação, seu conteúdo aparece nos campos correspondentes da caixa de diálogo. Para salvar suas alterações a qualquer momento, escolha o link **Salvar Perfil**.

2. Na seção **Ponto de extremidade da conexão**, especifique um ponto de extremidade de publicação local ou remoto de cluster do Service Fabric. Para adicionar ou alterar o ponto de extremidade de conexão, escolha o botão **Selecionar...**. A caixa de diálogo **Selecionar Cluster do Service Fabric** mostra os pontos de extremidade de conexão do cluster do Service Fabric disponíveis nos quais você pode publicar baseada em suas assinaturas do Azure. Você deve se conectar ao Visual Studio se já não estiver.

    Use a caixa de diálogo de seleção de cluster permite que você escolha o conjunto de assinaturas e clusters disponíveis.

    ![A caixa de diálogo **Selecionar Cluster do Service Fabric**][1]

    >[AZURE.NOTE]Se você deseja publicar em um ponto de extremidade arbitrário (como um cluster de terceiros), confira a seção **Publicando em um ponto de extremidade de cluster arbitrário** abaixo.

    Depois de escolher um ponto de extremidade, o Visual Studio valida a conexão com o cluster do Service Fabric selecionado. Se o cluster não for seguro, o Visual Studio poderá conectar-se a ele imediatamente. No entanto, se o cluster estiver protegido, será necessário instalar um certificado em seu computador local antes de continuar. Confira [Como configurar conexões seguras](service-fabric-visualstudio-configure-secure-connections.md) para saber mais. Ao terminar, escolha o botão **OK**. O cluster selecionado aparece na caixa de diálogo **Publicar Aplicativo do Service Fabric**.

3. Na caixa de listagem suspensa **Arquivo de Parâmetros do Aplicativo**, navegue até um arquivo de parâmetros do aplicativo. Um arquivo de parâmetros do aplicativo contém valores especificados pelo usuário para os parâmetros no arquivo de manifesto do aplicativo. Para adicionar ou alterar um parâmetro, escolha o botão **Editar**. Insira ou altere o valor do parâmetro na grade **Parâmetros**. Quando tiver terminado, escolha o botão **Salvar**.

    ![A caixa de diálogo **Editar Parâmetros**][2]

4. Use a caixa de seleção **Atualizar o Aplicativo** para especificar se essa ação de publicação é ou não uma atualização. Ações de publicação de atualização são diferentes das ações de publicação normais. Confira [Atualização de Aplicativo do Service Fabric](service-fabric-application-upgrade.md) para obter uma lista das diferenças. Para definir as configurações de atualização, escolha o link **Definir Configurações de Atualização**. O editor de parâmetros de atualização é exibido. Confira [Configurar a atualização de um aplicativo do Service Fabric](service-fabric-visualstudio-configure-upgrade.md) para saber mais sobre os parâmetros de atualização.

5. Escolha o botão **Versões do Manifesto...** para exibir a caixa de diálogo **Editar Versões**. Você precisa atualizar as versões do aplicativo e do serviço para que uma atualização ocorra. Veja [Tutorial de atualização de aplicativo do Service Fabric](service-fabric-application-upgrade-tutorial.md) para saber como as versões do aplicativo e do manifesto do serviço afetam um processo de atualização.

    ![A caixa de diálogo **Editar Versões**][3]

    Se o aplicativo e as versões de serviço usarem o controle de versão semântico, por exemplo, 1.0.0, ou valores numéricos no formato 1.0.0.0, selecione a opção **Atualizar automaticamente as versões do aplicativo e do serviço**. Quando você escolhe essa opção, os números de versão do serviço e do aplicativo são atualizados automaticamente sempre que um código, uma configuração ou a versão do pacote de dados é atualizado. Se você preferir editar as versões manualmente, desmarque a caixa de seleção para desativar esse recurso.

    >[AZURE.NOTE]Para que todas as entradas do pacote apareçam em um projeto de ator, primeiro compile o projeto a fim de gerar as entradas nos arquivos de Manifesto do Serviço.

6. Quando você terminar de especificar todas as configurações necessárias, escolha o botão **Publicar** para publicar seu aplicativo no cluster do Service Fabric selecionado. As configurações especificadas por você são aplicadas ao processo de publicação.

## Publicar em um ponto de extremidade de cluster arbitrário (incluindo clusters de terceiros)

A experiência de publicação do Visual Studio é otimizada para publicação em clusters remotos associados a uma de suas assinaturas do Azure. No entanto, é possível publicar em pontos de extremidade arbitrários (como clusters de parte do Service Fabric) editando diretamente o XML do perfil de publicação. Como descrito acima, dois perfis de publicação são fornecidos por padrão, **Local.xml** e **Cloud.xml**, mas você pode criar perfis adicionais para ambientes diferentes. Por exemplo, você pode querer criar um perfil de publicação em clusters de terceiros, talvez chamado **Party.xml**.

Se você está se conectando a um cluster não seguro, tudo de que precisa é o ponto de extremidade de conexão do cluster, como `partycluster1.eastus.cloudapp.azure.com:19000`. Nesse caso, o ponto de extremidade de conexão no perfil de publicação se assemelharia a este:

```XML
<ClusterConnectionParameters ConnectionEndpoint="partycluster1.eastus.cloudapp.azure.com:19000" />
```

  Se você estiver se conectando a um cluster protegido, precisará também fornecer os detalhes do certificado de cliente do repositório local para ser usado na autenticação. Para obter mais detalhes, veja [Configurando conexões seguras para um cluster do Service Fabric](service-fabric-visualstudio-configure-secure-connections.md).

  Depois de configurar o seu perfil de publicação, você pode referenciá-lo na caixa de diálogo de publicação, conforme mostrado abaixo.

  ![Novo perfil de publicação na caixa de diálogo de publicação][4]

  Observe que, nesse caso, o novo perfil de publicação aponta para um dos arquivos de parâmetro do aplicativo padrão. Isso é ideal se você deseja publicar a mesma configuração de aplicativo em vários ambientes. Por outro lado, quando você deseja ter configurações diferentes para cada ambiente que quiser publicar, faz sentido criar um arquivo de parâmetros de aplicativo correspondente.

## Próximas etapas

Para saber como automatizar o processo de publicação em um ambiente de Integração Contínua, confira [Configurar a integração contínua no Service Fabric](service-fabric-set-up-continuous-integration.md).


[0]: ./media/service-fabric-publish-app-remote-cluster/PublishDialog.png
[1]: ./media/service-fabric-publish-app-remote-cluster/SelectCluster.png
[2]: ./media/service-fabric-publish-app-remote-cluster/EditParams.png
[3]: ./media/service-fabric-publish-app-remote-cluster/EditVersions.png
[4]: ./media/service-fabric-publish-app-remote-cluster/publish-to-party-cluster.png

<!---HONumber=AcomDC_0114_2016-->