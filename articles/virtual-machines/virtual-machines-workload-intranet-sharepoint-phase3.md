<properties
	pageTitle="Farm do SharePoint Server 2013 Fase 3 | Microsoft Azure"
	description="Crie os computadores e o cluster do SQL Server e habilite grupos de disponibilidade na Fase 3 do farm do SharePoint Server 2013 no Azure."
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="Windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/11/2015"
	ms.author="josephd"/>

# Fase 3 da carga de trabalho do farm da intranet do SharePoint: Configurar a infraestrutura do SQL Server

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]modelo de implantação clássico.

Nesta fase da implantação de um farm do SharePoint 2013 somente intranet com Grupos de Disponibilidade AlwaysOn do SQL Server nos serviços de infraestrutura do Azure, você deve configurar os dois computadores do SQL Server e o computador do nó principal do cluster e combiná-los em um cluster do Windows Server.

Conclua esta fase antes de passar para a [Fase 4](virtual-machines-workload-intranet-sharepoint-phase4.md). Consulte [Implantando o SharePoint com Grupos de Disponibilidade AlwaysOn do SQL Server no Azure](virtual-machines-workload-intranet-sharepoint-overview.md) para conhecer todas as fases.

> [AZURE.NOTE]Essas instruções usam uma imagem do SQL Server na galeria de imagens do Azure e você receberá cobranças contínuas pelo uso da licença do SQL Server. Também é possível criar máquinas virtuais no Azure e instalar suas próprias licenças do SQL Server, mas você deverá ter o Software Assurance e a Licença de Mobilidade para usar sua licença do SQL Server em uma máquina virtual, incluindo uma máquina virtual do Azure. Para saber mais sobre como instalar o SQL Server em uma máquina virtual, consulte [Instalação do SQL Server](https://msdn.microsoft.com/library/bb500469.aspx).

## Criar as máquinas virtuais do cluster do SQL Server no Azure

Há duas máquinas virtuais do SQL Server. Um SQL Server contém a réplica do banco de dados primário de um grupo de disponibilidade. O segundo SQL Server contém a réplica de backup secundária. O backup é fornecido para garantir a alta disponibilidade. Uma máquina virtual adicional destina-se ao nó principal do cluster.

Use o seguinte bloco de comandos do PowerShell para criar as máquinas virtuais para os três servidores. Especifique os valores para as variáveis, removendo os caracteres < and >. Observe que esse conjunto de comandos do PowerShell usa os valores das seguintes tabelas:

- Tabela M para as máquinas virtuais
- Tabela V para as configurações da rede virtual
- Tabela S para a sub-rede
- Tabela ST para suas contas de armazenamento
- Tabela A para os conjuntos de disponibilidade

Lembre-se de que você definiu a Tabela M na [Fase 2: Configurar controladores de domínio](virtual-machines-workload-intranet-sharepoint-phase2.md) e as tabelas V, S, ST e A na [Fase 1: Configurar o Azure](virtual-machines-workload-intranet-sharepoint-phase1.md).

> [AZURE.NOTE]O comando a seguir define o uso do Azure PowerShell 1.0 e posterior. Para obter mais informações, consulte [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/).

Quando você tiver fornecido a todos os valores adequados, execute o bloco resultante no prompt de comando do Azure PowerShell.


	# Set up key variables
	$rgName="<your resource group name>"
	$locName="<Azure location of your resource group>"
	$saName="<Table ST – Item 1 – Storage account name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	$avName="<Table A – Item 2 – Availability set name column>"
	
	# Create the first SQL Server virtual machine
	$vmName="<Table M – Item 3 - Virtual machine name column>"
	$vmSize="<Table M – Item 3 - Minimum size column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$avSet=Get-AzureRMAvailabilitySet –Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for SQL data in GB>
	$diskLabel="<the label on the disk>"
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-SQLDataDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first SQL Server computer." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSQLServer -Offer SQL2014-WS2012R2 -Skus Enterprise -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second SQL Server virtual machine
	$vmName="<Table M – Item 4 - Virtual machine name column>"
	$vmSize="<Table M – Item 4 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for SQL data in GB>
	$diskLabel="<the label on the disk>"
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-ADDSDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second SQL Server computer." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSQLServer -Offer SQL2014-WS2012R2 -Skus Enterprise -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the cluster majority node server
	$vmName="<Table M – Item 5 - Virtual machine name column>"
	$vmSize="<Table M – Item 5 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the cluster majority node server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

> [AZURE.NOTE]Como essas máquinas virtuais são para um aplicativo de intranet, elas não recebem um endereço IP público ou um rótulo de nome de domínio DNS e não são expostas na Internet. No entanto, isso também significa que você não pode se conectar a eles no portal do Azure. O botão **Conectar** não ficará disponível quando você exibir as propriedades da máquina virtual. Use o acessório Conexão de Área de Trabalho Remota ou outra ferramenta da Área de Trabalho Remota para se conectar à máquina virtual usando o endereço IP privado ou o nome DNS da intranet.

## Configurar o computador do SQL Server

Para cada máquina virtual com o SQL Server em execução, use o cliente de área de trabalho remota de sua preferência e crie uma conexão de área de trabalho remota. Use seu nome DNS ou do computador da intranet e as credenciais da conta de administrador local.

Para cada máquina virtual que executa o SQL Server, associá-las ao domínio do AD DS apropriado com esses comandos no prompt do Windows PowerShell.

	$domName="<AD DS domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

Observe que você deve fornecer as credenciais de conta de domínio depois de inserir o comando Add-Computer.

Depois de reiniciar, reconecte-os usando uma conta que tenha privilégios de administrador local.

Para cada SQL Server, siga os seguintes procedimentos:

1. Use o [procedimento para fazer logon em uma máquina virtual com uma conexão de Área de Trabalho Remota](virtual-machines-workload-intranet-sharepoint-phase2.md#logon) para efetuar logon usando as credenciais da conta de administrador local.

2. Use o [procedimento para inicializar um disco vazio](virtual-machines-workload-intranet-sharepoint-phase2.md#datadisk) duas vezes, uma vez para cada SQL Server, para adicionar o disco de dados adicional.

3. Execute os seguintes comandos em um prompt de comando do Windows PowerShell.

		md f:\Data
		md f:\Log
		md f:\Backup

4. Use o [procedimento para testar a conectividade](virtual-machines-workload-intranet-sharepoint-phase2.md#testconn) para testar a conectividade para os locais na rede da sua organização. Esse procedimento garante que a resolução de nome DNS esteja funcionando corretamente (que a máquina virtual esteja configurada corretamente com os servidores DNS na rede virtual) e que pacotes possam ser enviados para e da rede virtual entre locais.

Use o procedimento a seguir duas vezes, uma vez para cada SQL Server, para configurar o SQL Server para usar a unidade F: para novos bancos de dados e para contas e permissões.

1.	Na tela inicial, digite **SQL Studio** e, em seguida, clique em **SQL Server 2014 Management Studio**.
2.	Em **Conectar ao Servidor**, clique em **Conectar**.
3.	No painel esquerdo, clique com botão direito no nó superior (a instância padrão com o mesmo nome da máquina) e, em seguida, clique em **Propriedades**.
4.	Em **Propriedades do Servidor**, clique em **Configurações de Banco de Dados**.
5.	Em **Locais padrão do banco de dados**, defina os seguintes valores:
- Para **Dados**, defina o caminho como **f:\\Data**.
- Para **Log**, defina o caminho como **f:\\Log**.
- Para **Backup**, defina o caminho como **f:\\Backup**.
- Somente novos bancos de dados usam esses locais.
6.	Clique em **OK** para fechar a janela.
7.	No painel esquerdo, expanda a **pasta Segurança**.
8.	Clique com o botão direito em **Logons** e clique em **Novo logon**.
9.	Em **Nome de logon**, digite *domínio*\\sp\_farm\_db em (no qual *domínio* é o nome do domínio no qual a conta sp\_farm\_db foi criada).
10.	Em **Selecionar uma página**, clique em **Funções de Servidor**, em **sysadmin** e em **OK**.
11.	Feche o SQL Server 2014 Management Studio.

Use o procedimento a seguir duas vezes, uma vez para cada SQL Server, para permitir conexões de área de trabalho remota usando a conta sp\_farm\_db.

1.	Na tela inicial, clique **Este PC** e em **Propriedades**.
2.	Na janela **Sistema**, clique em **Configurações Remotas**.
3.	Na seção **Área de Trabalho Remota**, clique em **Selecionar Usuários** e, em seguida, clique em **Adicionar**.
4.	Em **Digite os nomes de objetos a serem selecionados**, digite [domínio]**\\sp\_farm\_db** e clique em **OK** três vezes.

O SQL Server exige uma porta que os clientes usem para acessar o servidor de banco de dados. Ele também precisa de portas para se conectar com o SQL Server Management Studio e gerenciar o grupo de alta disponibilidade. Em seguida, execute o seguinte comando em um prompt de comando de nível de administrador do Windows PowerShell duas vezes, uma vez para cada SQL Server, para adicionar uma regra de firewall que permita o tráfego de entrada para o SQL Server.

	New-NetFirewallRule -DisplayName "SQL Server ports 1433, 1434, and 5022" -Direction Inbound –Protocol TCP –LocalPort 1433,1434,5022 -Action Allow

Saia como administrador local de cada uma das máquinas virtuais do SQL Server.

Para obter informações sobre como otimizar o desempenho do SQL Server no Azure, consulte [Práticas recomendadas de desempenho do SQL Server em máquinas virtuais do Azure](virtual-machines-sql-server-performance-best-practices.md). Você também desabilitar o armazenamento com redundância geográfica (GRS) para a conta de armazenamento do farm do SharePoint e usar espaços de armazenamento para otimizar o IOPs.

## Configurar o servidor do nó principal do cluster

Use o [procedimento para fazer logon em uma máquina virtual com uma conexão de Área de Trabalho Remota](virtual-machines-workload-intranet-sharepoint-phase2.md#logon) para que o nó principal do cluster efetue logon usando as credenciais de uma conta do domínio.

No nó principal do cluster, use o [procedimento para testar a conectividade](virtual-machines-workload-intranet-sharepoint-phase2.md#testconn) para testar a conectividade para os locais na rede da sua organização.

## Criar o cluster do Windows Server

Os Grupos de Disponibilidade AlwaysOn do SQL Server dependem do recurso de Clustering de Failover do Windows Server (WSFC). Esse recurso permite que várias máquinas participem como um grupo em um cluster. Quando uma máquina falhar, um segundo computador estará pronto para assumir seu lugar. Portanto, a primeira tarefa é habilitar o recurso de Clustering de Failover em todos os computadores participantes, que incluem:

- O SQL Server primário
- O SQL Server secundário
- O nó principal do cluster

O cluster de failover requer pelo menos três VMs. Duas máquinas hospedam o SQL Server. A segunda VM do SQL Server é uma réplica secundária síncrona, que garante que nenhum dado seja perdido se a máquina primária falhar. A terceira máquina não precisa hospedar o SQL Server. O nó principal do cluster fornecem um quorum no WSFC. Como o cluster do WSFC depende de um quorum para monitorar a integridade, o nó principal é necessário para garantir que o cluster do WSFC esteja sempre online. Se apenas duas máquinas estiverem em um cluster e uma falhar, não haverá quorum. Para saber mais, consulte [Configuração de modos de quorum WSFC e votação (SQL Server)](http://msdn.microsoft.com/library/hh270280.aspx).

Para os computadores do SQL Server e o nó principal do cluster, execute o seguinte comando em um prompt de comando de nível de administrador do Windows PowerShell.

	Install-WindowsFeature Failover-Clustering -IncludeManagementTools

Devido ao comportamento atual não compatível com RFC do DHCP no Azure, a criação de um Cluster de Failover do Windows Server (WSFC) pode falhar. Para obter detalhes, procure por "Comportamento do cluster do WSFC no sistema de rede do Azure" em Alta disponibilidade e recuperação de desastres para o SQL Server em máquinas virtuais do Azure. No entanto, há uma solução alternativa. Use as etapas a seguir para criar o cluster:

1.	Faça logon na máquina virtual primária do SQL Server com a conta **sp\_install**.
2.	Na tela inicial, digite **Failover** e clique em **Gerenciador de Cluster de Failover**.
3.	No painel esquerdo, clique com botão direito **Gerenciador de Cluster de Failover** e, em seguida, clique em **Criar Cluster**.
4.	Na página Antes de Começar, clique em **Avançar**.
5.	Na página Selecionar Servidores, digite o nome da máquina primária do SQL Server, clique em **Adicionar** e, em seguida, clique em **Avançar**.
6.	Na página Aviso de Validação, clique em **Não. Eu não preciso de suporte da Microsoft para este cluster e, portanto, não desejo executar os testes de validação. Ao clicar em Avançar, continue a criação do cluster** e, em seguida, clique em **Avançar**.
7.	Na página Ponto de Acesso para Administrar o Cluster, na caixa de texto **Nome do Cluster**, digite o nome do cluster e clique em **Avançar**.
8.	Na página Confirmação, clique em **Avançar** para começar a criação do cluster.
9.	Na página Resumo, clique em **Concluir**.
10.	No painel esquerdo, clique no novo cluster. Na seção **Recursos Principais de Cluster** do painel de conteúdo, abra o nome do cluster do servidor. O recurso **Endereço IP** aparece com o estado **Falha**. O recurso de endereço IP não pode ficar online porque o cluster recebeu o mesmo endereço IP que a própria máquina. O resultado é um endereço duplicado.
11.	Clique com o botão direito no recurso **Endereço IP** com falha e clique em **Propriedades**.
12.	Na caixa de diálogo **Propriedades do Endereço IP**, clique em **Endereço IP Estático**.
13.	Digite um IP não usado no intervalo de endereço correspondente à sub-rede na qual o SQL Server está localizado e, em seguida, clique em **OK**.
14.	Clique com o botão direito no recurso Endereço IP com falha e clique em **Colocar Online**. Aguarde até que ambos os recursos estejam online. Quando o recurso de nome do cluster fica online, ele atualiza o controlador de domínio com uma nova conta de computador do Active Directory (AD). Essa conta do AD é usada posteriormente para executar o serviço de cluster do grupo de disponibilidade.
15.	Agora que a conta do AD foi criada, coloque o nome do cluster offline. Clique no nome do cluster em **Recursos Principais de Cluster** e, em seguida, clique em **Colocar Offline**.
16.	Para remover o endereço IP do cluster, clique com botão direito em **Endereço IP**, clique em **Remover** e, em seguida, clique em **Sim** quando solicitado. O recurso de cluster não poderá mais ficar online porque ele depende do recurso de endereço IP. No entanto, um grupo de disponibilidade não depende do nome do cluster ou do endereço IP para funcionar corretamente. Dessa forma, o nome do cluster pode ficar offline.
17.	Para adicionar os nós restantes ao cluster, clique com o botão direito do mouse no nome do cluster no painel esquerdo e clique em **Adicionar Nó**.
18.	Na página Antes de Começar, clique em **Avançar**.
19.	Na página Selecionar Servidores, digite o nome e, em seguida, clique em **Adicionar** para adicionar o SQL Server secundário e o nó principal do cluster ao cluster. Depois de adicionar os dois computadores, clique em **Avançar**. Se não for possível adicionar uma máquina e a mensagem de erro exibida for “Serviço de Registro Remoto não está em execução”, faça o seguinte: Faça logon na máquina, abra o snap-in Serviços (services.msc) e habilite o Registro Remoto. Para saber mais , consulte [Não é possível se conectar ao Serviço de Registro Remoto](http://technet.microsoft.com/library/bb266998.aspx).
20.	Na página Aviso de Validação, clique em **Não. Eu não preciso de suporte da Microsoft para este cluster e, portanto, não desejo executar os testes de validação. Ao clicar em Avançar, continue a criação do cluster** e, em seguida, clique em **Avançar**.
21.	Na página Confirmação, clique em **Avançar**.
22.	Na página Resumo, clique em **Concluir**.
23.	No painel esquerdo, clique em **Nós**. Você deve ver os três computadores listados.

## Habilitar Grupos de Disponibilidade AlwaysOn

A próxima etapa é habilitar os Grupos de Disponibilidade AlwaysOn usando o SQL Server Configuration Manager. Observe que um grupo de disponibilidade no SQL Server é diferente de um conjunto de disponibilidade do Azure. Um grupo de disponibilidade contém bancos de dados que são altamente disponíveis e recuperáveis. Um conjunto de disponibilidade do Azure aloca máquinas virtuais de diferentes domínios de falha. Para saber mais sobre domínios de falha, consulte [Gerenciamento da disponibilidade das máquinas virtuais](virtual-machines-manage-availability.md).

Use essas etapas para habilitar Grupos de Disponibilidade AlwaysOn no SQL Server.

1.	Faça logon no SQL Server primário usando a conta **sp\_farm\_db** ou outra conta que tenha a função de servidor sysadmin no SQL Server.
2.	Na tela inicial, digite **Configuração do SQL Server** e, em seguida, clique em **SQL Server Configuration Manager**.
3.	No painel esquerdo, clique em **Serviços do SQL Server**.
4.	No painel de conteúdo, clique duas vezes em **SQL Server (MSSQLSERVER)**.
5.	Em **Propriedades do SQL Server (MSSQLSERVER)**, clique na guia **Alta Disponibilidade AlwaysOn**, selecione **Habilitar Grupos de Disponibilidade AlwaysOn**, clique em **Aplicar** e, em seguida, clique em **OK** quando solicitado. Não feche a janela de propriedades ainda.
6.	Clique na guia virtual-machines-manage-availability e digite [Domínio]**\\sqlservice** em **Nome da Conta**. Digite a senha da conta sqlservice em **Senha** e em **Confirmar senha** e clique em **OK**.
7.	Na janela de mensagem, clique em **Sim** para reiniciar o serviço do SQL Server.
8.	Faça logon no SQL Server secundário e repita esse processo.

O próximo diagrama mostra a configuração resultante da conclusão bem-sucedida desta fase, com nomes de computador de espaço reservado.

![](./media/virtual-machines-workload-intranet-sharepoint-phase3/workload-spsqlao_03.png)

## Próxima etapa

- Use a [Fase 4](virtual-machines-workload-intranet-sharepoint-phase4.md) para continuar com a configuração desta carga de trabalho.

<!---HONumber=AcomDC_1217_2015-->