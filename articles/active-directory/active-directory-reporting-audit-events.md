<properties
   pageTitle="Eventos do Relatório de Auditoria do Active Directory do Azure | Microsoft Azure"
   description="Eventos de auditoria que estão disponíveis para exibição e download no Active Directory do Azure"
   services="active-directory"
   documentationCenter=""
   authors="kenhoff"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="12/07/2015"
   ms.author="kenhoff"/>

# Eventos de relatório de auditoria do Active Directory do Azure

*Esta documentação é parte do [Guia de Relatórios do Active Directory do Azure](active-directory-reporting-guide.md).*

O relatório de auditoria do Active Directory do Azure ajuda os clientes a identificar ações privilegiadas que ocorreram nos seus respectivos Active Directory do Azure. Ações com privilégios incluem alterações de elevação (por exemplo, criação de funções ou redefinições de senha), alteração de configurações de política (por exemplo, políticas de senha) ou alterações na configuração de diretório (por exemplo, as alterações às configurações de federação de domínio). Os relatórios fornecem o registro de auditoria para o nome do evento, o ator que executou a ação, o recurso de destino afetado pela alteração e a data e hora (em UTC). Os clientes são capazes de recuperar a lista de eventos de auditoria para o Active Directory do Azure por meio do [Portal de Gerenciamento do Azure](https://manage.windowsazure.com/), conforme descrito em [Veja seus relatórios de acesso e de uso](active-directory-view-access-usage-reports.md).


## Lista de eventos de relatório de auditoria
<!--- audit event descriptions should be in the past tense --->

Eventos | Descrição do evento
------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Eventos de usuário** |
Adicionar usuário | Um usuário foi adicionado ao diretório.
Excluir usuário | Um usuário foi excluído do diretório.
Definir propriedades de licença | Defina as propriedades de licença para um usuário no diretório.
Redefinir senha de usuário | Redefina a senha de um usuário no diretório.
Alterar senha de usuário | A senha de um usuário no diretório foi alterada.
Alterar licença de usuário | A licença atribuída a um usuário no diretório foi alterada. Para ver quais licenças foram atualizadas, observe o evento "Atualizar usuário" imediatamente após ou antes esse evento.
Atualizar usuário | Um usuário no diretório foi atualizado. [Veja abaixo](#quotupdate-userquot-attributes) os atributos que podem ser atualizados.
Definir alteração forçada de senha de usuário | Defina a propriedade que força o usuário a alterar sua senha no logon.
**Eventos de grupo** |
Adicionar grupo | Criou um grupo no diretório.
Atualizar grupo | Atualizou um grupo no diretório.
Excluir grupo | Excluiu um grupo no diretório.
Adicionar membro ao grupo | Adicionou um membro a um grupo no diretório.
Remover membro do grupo | Removeu um membro de um grupo no diretório.
**Eventos de aplicativo** |
Adicionar entidade de serviço | Uma entidade de serviço foi adicionada ao diretório.
Remover entidade de serviço | Uma entidade de serviço foi removida do diretório.
Adicionar credenciais de entidade de serviço | Credenciais adicionadas a uma entidade de serviço.
Remover credenciais de entidade de serviço | Credenciais removidas de uma entidade de serviço.
Adicionar entrada de delegação | [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity) criado no diretório.
Definir entrada de delegação | [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity) atualizado no diretório.
Remover entrada de delegação | [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity) excluído do diretório.
**Eventos de função** |
Adicionar um membro da função à Função | Um usuário foi adicionado a uma função de diretório.
Remover membro de função da Função | Um usuário foi removido de uma função de diretório.
Definir informações de contato da empresa | Defina preferências de contato do nível da empresa. Isso inclui endereços de email para marketing, bem como notificações técnicas sobre os Serviços Online da Microsoft.
**Eventos B2B** |
Convites do Lote carregados. | Um administrador carregou um arquivo que contém os convites a serem enviados para os usuários do parceiro.
Convites do Lote processados. | Um arquivo que contém os convites para os usuários do parceiro foi processado.
Convide um usuário externo. | Um usuário externo foi convidado para o diretório.
Resgate um convite para usuário externo. | Um usuário externo resgatou seu convite para o diretório.
Adicione o usuário externo ao grupo. | Um usuário externo recebeu associação a um grupo no diretório.
Atribua um usuário externo ao aplicativo. | Um usuário externo recebeu acesso direto a um aplicativo.
Criação de locatário viral. | Um novo locatário foi criado no AD do Azure pelo resgate do convite.
Criação de usuário viral. | Um usuário foi criado em um locatário existente no AD do Azure pelo resgate do convite.
**Eventos de diretório** |
Adicionar um parceiro à empresa | Um parceiro foi adicionado ao diretório.
Remover o parceiro da empresa | Um parceiro foi removido do diretório.
Adicionar domínio a empresa | Um domínio foi adicionado ao diretório.
Remover o domínio da empresa | Um domínio foi removido do diretório.
Domínio de atualização | Um domínio no diretório foi atualizado.
Definir a autenticação de domínio | A configuração de domínio padrão para a empresa foi alterada.
Definir configurações de federação no domínio | As configurações de federação para um domínio foram atualizadas.
Verificar domínio | Um domínio no diretório foi verificado.
Verificar domínio por email | Um domínio no diretório foi verificado usando verificação por email.
Definir o sinalizador DirSyncEnabled na empresa | Defina a propriedade que habilita um diretório para o Azure AD Sync.
Definir política de senha | Defina as restrições de comprimento e de caractere para senhas de usuário.
Definir informações da empresa | Atualizadas as informações de nível de empresa. Consulte o cmdlet [Get-MsolCompanyInformation](https://msdn.microsoft.com/library/azure/dn194126.aspx) do PowerShell para obter mais detalhes.

<!---

List of events that still need descriptions:

Restore Application
Set String Auth Policy
Promote tenant to partner

--->

## Retenção do relatório de auditoria
Os eventos no relatório de auditoria do Azure AD são mantidos por 180 dias. Para saber mais sobre retenção de relatórios, confira [Políticas de retenção de relatório do Active Directory do Azure](active-directory-reporting-retention.md).

Para clientes interessados no armazenamento de seus eventos de auditoria por períodos mais longos, a API de Relatório pode ser usada regularmente receber eventos de auditoria em um armazenamento de dados separado. Confira [Introdução à API de Relatório](active-directory-reporting-api-getting-started.md) para obter detalhes.

## Propriedades incluídas com cada evento de auditoria

Propriedade | Descrição
------------- | --------------------------------------------------------------
Data e hora | A data e hora em que o evento de auditoria ocorreu
Ator | O usuário ou a entidade de serviço que executou a ação
Ação | A ação que foi executada
Destino | O usuário ou a entidade de serviço em que a ação foi executada


## Atributos de “Atualizar Usuário”
O evento de auditoria "Atualizar usuário" inclui informações adicionais sobre quais atributos de usuário foram atualizados. Para cada atributo, o valor anterior e o novo valor são ambos incluídos.

Atributo | Descrição
------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------
AccountEnabled | O usuário tem permissão para entrar.
AssignedLicense | Todas as licenças que foram atribuídas ao usuário.
AssignedPlan | Planos de serviço resultante das licenças atribuídas ao usuário.
LicenseAssignmentDetail | Detalhes sobre licenças atribuídas ao usuário. Por exemplo, se o licenciamento baseado em grupo estava envolvido, isso inclui o grupo que concedeu a licença.
Móvel | O celular do usuário.
OtherMail | O endereço de email alternativo do usuário.
OtherMobile | O celular alternativo do usuário.
StrongAuthenticationMethod | Uma lista de métodos de verificação configurados pelo usuário para Multi-Factor Authentication, como o código de chamada de voz, SMS ou verificação de um aplicativo móvel.
StrongAuthenticationRequirement | Se a Multi-Factor Authentication é imposta, habilitada ou desabilitada para este usuário.
StrongAuthenticationUserDetails | Número de telefone do usuário, número de telefone alternativo e endereço de email usado para verificação de Multi-Factor Authentication e redefinição de senha.
TelephoneNumber | O número de telefone do usuário.

Registros de auditoria são um controle necessário para muitas regulamentações de conformidade. Para clientes que usam o Relatório de Auditoria do Active Directory do Azure para atender aos seus regulamentos de conformidade, recomenda-se que o cliente envie uma cópia deste tópico de ajuda com a cópia do relatório de auditoria exportado do cliente para ajudar a explicar os detalhes do relatório. Se o auditor gostaria de entender os regulamentos de conformidade que o Azure atende atualmente, indique a [Página de conformidade](https://azure.microsoft.com/support/trust-center/compliance/) da Central de confiabilidade do Microsoft Azure ao auditor.

<!---HONumber=AcomDC_0128_2016-->