<properties
	pageTitle="Executar o assistente Habilitar o Banco de Dados para o Stretch | Microsoft Azure"
	description="Saiba como configurar um banco de dados para o Stretch Database executando assistente Habilitar Banco de Dados para Stretch."
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/26/2016"
	ms.author="douglasl"/>

# Executar o assistente Habilitar o Banco de Dados para Stretch

Para configurar um banco de dados para Banco de Dados de Stretch, execute o assistente Habilitar Banco de Dados para Stretch. Este tópico descreve as informações que você precisa inserir e as escolhas que você deve fazer no assistente.

Para saber mais sobre o Banco de Dados de Stretch, confira [Banco de Dados de Stretch](sql-server-stretch-database-overview.md).

## Iniciar o assistente

1.  No SQL Server Management Studio, no Pesquisador de Objetos, escolha o banco de dados no qual você deseja habilitar o Stretch.

2.  Clique com botão direito do mouse e escolha **Tarefas** e, em seguida, escolha **Stretch** e **Habilitar** para iniciar o assistente.

## <a name="Intro"></a>Introdução
Examine o objetivo do assistente e os pré-requisitos.

## <a name="Tables"></a>Selecionar tabelas
Selecione as tabelas que você deseja habilitar para o Stretch.

|Coluna|Descrição|
|----------|---------------|
|(sem título)|Marque a caixa de seleção nessa coluna para habilitar a tabela selecionada para Stretch.|
|**Nome**|Especifica o nome da coluna na tabela.|
|(sem título)|Um símbolo nesta coluna normalmente indica que não é possível habilitar a tabela selecionada para o Stretch devido a um problema de bloqueio. Talvez o motivo seja o uso de um tipo de dados sem suporte pela tabela. Passe o cursor do mouse sobre o símbolo para exibir mais informações em uma dica de ferramenta. Para saber mais, confira [Problemas de bloqueio e limitações de área da superfície do Banco de Dados de Stretch](sql-server-stretch-database-limitations.md).|
|**Ampliado**|Indica se a tabela já está habilitada.|
|**Linhas**|Especifica o número de linhas na tabela.|
|**Tamanho (KB)**|Especifica o tamanho da tabela em KB.|
|**Migrar**|No CTP 3.1, por meio do RC0, só é possível migrar uma tabela inteira usando o assistente. Se você quiser especificar um predicado para selecionar as linhas para migração em uma tabela que contém os dados atuais e do histórico, execute a instrução ALTERAR TABELA para especificar um predicado depois de sair do assistente. Para saber mais, confira [Habilitar o Banco de Dados de Stretch para uma tabela](sql-server-stretch-database-enable-table.md) ou [ALTERAR TABELA (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx).|

## <a name="Configure"></a>Configurar a implantação do Azure

1.  Entre no Microsoft Azure com uma conta da Microsoft.

2.  Escolha a assinatura do Azure que você quer usar para o Banco de Dados de Stretch.

3.  Selecione uma região do Azure. Se você criar um novo servidor, ele será criado nessa região.

    Para minimizar a latência, selecione a região do Azure na qual o SQL Server está localizado. Para saber mais sobre regiões, confira [Regiões do Azure](https://azure.microsoft.com/regions/).

4.  Especifique se você quer usar um servidor existente ou criar um novo servidor do Azure.

    Se o Active Directory no SQL Server for federado com o Azure Active Directory, você poderá optar por usar uma conta de serviço federado para o SQL Server para se comunicar com o servidor remoto do Azure. Para saber mais sobre os requisitos dessa opção, confira [Opções de ALTERAR CONJUNTO DO BANCO DE DADOS (Transact-SQL)](https://msdn.microsoft.com/library/bb522682.aspx).

    -   **Servidor existente**

        1.  Escolha ou digite o nome do servidor do Azure existente.

        2.  Selecione o método de autenticação

            -   Se você escolher **Autenticação do SQL Server**, crie um novo logon e senha.

            -   Escolha **Autenticação Integrada do Active Directory** para usar uma conta de serviço federado para o SQL Server se comunicar com o servidor remoto do Azure.

    -   **Criar um novo servidor**

        1.  Crie um logon e senha para o administrador do servidor.

        2.  Como opção, use uma conta de serviço federado para o SQL Server se comunicar com o servidor remoto do Azure.

## <a name="Credentials"></a>Proteger as credenciais
Digite uma senha forte para criar uma chave mestra de banco de dados ou, se já houver uma chave mestra de banco de dados, digite a senha dela.

Você precisa ter uma chave mestra de banco de dados para proteger as credenciais usadas pelo Banco de Dados de Stretch para se conectar ao banco de dados remoto.

Para saber mais sobre a chave mestra de banco de dados, confira [CRIAR CHAVE MESTRA (Transact-SQL)](https://msdn.microsoft.com/library/ms174382.aspx) e [Criar uma Chave Mestra de Banco de Dados](https://msdn.microsoft.com/library/aa337551.aspx). Para saber mais sobre as credenciais criadas pelo assistente, confira [CRIAR CREDENCIAL COM ESCOPO DO BANCO DE DADOS (Transact-SQL)](https://msdn.microsoft.com/library/mt270260.aspx).

## <a name="Network"></a>Selecionar o endereço IP
Use o endereço IP público de seu SQL Server ou digite um intervalo de endereços IP para criar uma regra de firewall no Azure que permita ao SQL Server se comunicar com o servidor remoto do Azure.

## <a name="Summary"></a>Resumo
Examine os valores inseridos e as opções selecionadas no assistente. Em seguida, escolha **Concluir** para habilitar o Stretch.

## <a name="Results"></a>Resultados
Revise os resultados.

Como opção, escolha **Monitorar** para iniciar o monitoramento do status da migração de dados do Monitor do Banco de Dados de Stretch. Para saber mais, confira [Monitorar e solucionar problemas de migração de dados (Banco de Dados de Stretch)](sql-server-stretch-database-monitor.md).

## <a name="KnownIssues"></a>Solução de problemas com o assistente
**O assistente do Banco de Dados de Stretch falhou.** Se o Banco de Dados de Stretch ainda não estiver habilitado no nível do servidor, e você executar o assistente sem as permissões de administrador do sistema para habilitá-lo, o assistente falhará. Peça ao administrador do sistema para habilitar o Banco de Dados de Stretch na instância do servidor local e, em seguida, execute o assistente novamente. Para saber mais, confira [Pré-requisito: permissão para habilitar o Banco de Dados de Stretch no servidor](sql-server-stretch-database-enable-database.md#EnableTSQLServer).

## Próximas etapas
Habilitar outras tabelas para o Banco de Dados de Stretch. Monitorar a migração de dados e gerenciar as tabelas e bancos de dados habilitados para o Stretch.

-   [Habilitar o Banco de Dados de Stretch para uma tabela](sql-server-stretch-database-enable-table.md) para habilitar outras tabelas.

-   [Monitorar o Banco de Dados de Stretch](sql-server-stretch-database-monitor.md) para ver o status da migração dos dados.

-   [Pausar e retomar o Banco de Dados de Stretch](sql-server-stretch-database-pause.md)

-   [Gerenciar e solucionar problemas do Banco de Dados de Stretch](sql-server-stretch-database-manage.md)

-   [Fazer backup e restaurar os bancos de dados habilitados para Stretch](sql-server-stretch-database-backup.md)

## Consulte também
[Habilitar o Banco de Dados de Stretch para um banco de dados](sql-server-stretch-database-enable-database.md) [Habilitar o Banco de Dados de Stretch para uma tabela](sql-server-stretch-database-enable-table.md)

<!---HONumber=AcomDC_0302_2016-->