<properties pageTitle="Usar PowerShell para habilitar o Diagnóstico do Azure em uma máquina virtual que executa o Windows | Microsoft Azure" services="virtual-machines" documentationCenter="" description="Saiba como usar o PowerShell para habilitar o Diagnóstico do Azure em uma máquina virtual que executa o Windows" authors="sbtron" manager="" editor="""/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/15/2015"
	ms.author="saurabh"/>


# Usar o PowerShell para habilitar o Diagnóstico do Azure em uma máquina virtual que executa o Windows

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

O Diagnóstico do Azure é a capacidade do Azure que habilita a coleta de dados de diagnóstico em um aplicativo implantado. Você pode usar a extensão de diagnóstico para coletar dados de diagnóstico como logs de aplicativo ou contadores de desempenho de uma máquina virtual (VM) do Azure que executa o Windows. Este artigo descreve como usar o Windows PowerShell para habilitar a extensão de diagnóstico para uma VM. Consulte [Como instalar e configurar o Azure PowerShell](powershell-install-configure.md) para os pré-requisitos necessários para este artigo.

## Habilitar a extensão de diagnóstico se você usar o modelo de implantação do Gerenciador de Recursos

Você pode habilitar a extensão de diagnóstico enquanto cria uma VM do Windows por meio do modelo de implantação do Gerenciador de Recursos do Azure adicionando a configuração da extensão ao modelo do Gerenciador de Recursos. Consulte [Criar uma máquina virtual do Windows com monitoramento e diagnóstico usando o modelo do Gerenciador de Recursos do Azure](virtual-machines-extensions-diagnostics-windows-template.md).

Para habilitar a extensão de diagnóstico em uma VM existente criada por meio do modelo de implantação do Gerenciador de Recursos, é possível usar o cmdlet do PowerShell [Set-AzureRMVMDiagnosticsExtension](https://msdn.microsoft.com/library/mt603499.aspx), conforme mostrado abaixo.


	$vm_resourcegroup = "myvmresourcegroup"
	$vm_name = "myvm"
	$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"

	Set-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path


*$diagnosticsconfig\_path* é o caminho para o arquivo que contém a configuração de diagnóstico em XML, conforme descrito no [exemplo](#sample-diagnostics-configuration) abaixo.

Se o arquivo de configuração de diagnóstico especificar um elemento **StorageAccount** com um nome de conta de armazenamento, o script *Set-AzureRMVMDiagnosticsExtension* definirá automaticamente a extensão de diagnóstico para enviar dados de diagnóstico para essa conta de armazenamento. Para isso funcionar, a conta de armazenamento precisa estar na mesma assinatura que a VM.

Se nenhuma **StorageAccount** tiver sido especificada na configuração de diagnóstico, você precisa passar o parâmetro *StorageAccountName* para o cmdlet. Se o parâmetro *StorageAccountName* for especificado, o cmdlet sempre usará a conta de armazenamento que está especificada no parâmetro, não aquela que está especificada no arquivo de configuração de diagnóstico.

Se a conta de armazenamento de diagnóstico estiver em uma assinatura diferente que a VM, você precisará transmitir explicitamente os parâmetros *StorageAccountName* e *StorageAccountKey* para o cmdlet. O parâmetro *StorageAccountKey* não é necessário quando a conta de armazenamento de diagnóstico está na mesma assinatura, uma vez que o cmdlet pode consultar e definir automaticamente o valor de chave ao habilitar a extensão de diagnóstico. No entanto, se a conta de armazenamento de diagnóstico estiver em uma assinatura diferente, o cmdlet talvez não possa obter a chave automaticamente e você precisará explicitamente especificar a chave por meio do parâmetro *StorageAccountKey*.

	Set-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key

Depois que a extensão de diagnóstico estiver habilitada em uma VM, você poderá obter as configurações atuais usando o cmdlet [Get-AzureRMVmDiagnosticsExtension](https://msdn.microsoft.com/library/mt603678.aspx).

	Get-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name

O cmdlet retorna *PublicSettings*, que contém a configuração XML em um formato codificado em Base64. Para ler o XML, você precisa decodificá-lo.

	$publicsettings = (Get-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name).PublicSettings
	$encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
	$xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
	Write-Host $xmlconfig

O cmdlet [Remove-AzureRMVmDiagnosticsExtension](https://msdn.microsoft.com/library/mt603782.aspx) pode ser usado para remover a extensão de diagnóstico da VM.

## Habilitar a extensão de diagnóstico se você usar o modelo de implantação clássico

É possível usar o cmdlet [Set-AzureVMDiagnosticsExtension](https://msdn.microsoft.com/library/mt589189.aspx) pode ser usado para habilitar a extensão de diagnóstico em uma VM criada usando o modelo de implantação clássico. O exemplo a seguir mostra como criar uma nova VM por meio do modelo de implantação clássico com a extensão de diagnóstico habilitada.

	$VM = New-AzureVMConfig -Name $VM -InstanceSize Small -ImageName $VMImage
	$VM = Add-AzureProvisioningConfig -VM $VM -AdminUsername $Username -Password $Password -Windows
	$VM = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
	New-AzureVM -Location $Location -ServiceName $Service_Name -VM $VM

Para habilitar a extensão de diagnóstico em uma VM existente que foi criada por meio do modelo de implantação clássico, primeiramente use o cmdlet [Get-AzureVM](https://msdn.microsoft.com/library/mt589152.aspx) para obter a configuração da VM. Em seguida, atualize a configuração de VM para incluir a extensão de diagnóstico usando o cmdlet [Set-AzureVMDiagnosticsExtension](https://msdn.microsoft.com/library/mt589189.aspx). Por fim, aplique a configuração atualizada à VM usando [Update-AzureVM](https://msdn.microsoft.com/library/mt589121.aspx).

	$VM = Get-AzureVM -ServiceName $Service_Name -Name $VM_Name
	$VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
	Update-AzureVM -ServiceName $Service_Name -Name $VM_Name -VM $VM_Update.VM

## Exemplo de configuração de diagnóstico

O XML a seguir pode ser usado para a configuração de diagnóstico público com os scripts acima. Este exemplo de configuração transferirá vários contadores de desempenho para a conta de armazenamento de diagnóstico junto com erros de aplicativo, segurança e canais do sistema nos logs de eventos do Windows e quaisquer erros dos logs de infraestrutura do diagnóstico.

A configuração precisa ser atualizada para incluir o seguinte:

- O atributo *resourceID* do elemento **Métricas** precisa ser atualizado com a ID de recurso para a VM.
	- A ID do recurso pode ser criada usando o seguinte padrão: "/subscriptions/{*ID da assinatura para a assinatura com a VM*}/resourceGroups/{*o nome do grupo de recursos para VM*}/providers/Microsoft.Compute/virtualMachines/{*o nome da VM*}".
	- Por exemplo, se a ID de assinatura para a assinatura em que a VM está em execução for **11111111-1111-1111-1111-111111111111**, o nome do grupo de recursos para o grupo de recursos será **MyResourceGroup** e o nome da VM será **MyWindowsVM**, então o valor de *resourceID* será:

		```
		<Metrics resourceId="/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyWindowsVM" >
		```

	- Para obter mais informações sobre como as métricas são geradas com base na configuração de contadores e métricas de desempenho, consulte a [tabela de métricas de Diagnóstico do Azure no armazenamento](virtual-machines-extensions-diagnostics-windows-template.md#wadmetrics-tables-in-storage).

- O elemento **StorageAccount** precisa ser atualizado com o nome da conta de armazenamento de diagnóstico.

	```
	<?xml version="1.0" encoding="utf-8"?>
	<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <WadCfg>
	      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">
	        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error"/>
	        <PerformanceCounters scheduledTransferPeriod="PT1M">
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU utilization" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Privileged Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU privileged time" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% User Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU user time" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor Information(_Total)\Processor Frequency" sampleRate="PT15S" unit="Count">
	        <annotation displayName="CPU frequency" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\System\Processes" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Processes" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Thread Count" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Threads" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Handle Count" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Handles" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes In Use" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Memory usage" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory available" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory committed" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Commit Limit" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory commit limit" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Paged Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory paged pool" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Nonpaged Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory non-paged pool" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active time" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Read Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active read time" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Write Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active write time" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Transfers/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk operations" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Reads/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk read operations" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Writes/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk write operations" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk speed" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Read Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk read speed" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Write Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk write speed" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average queue length" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Read Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average read queue length" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Write Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average write queue length" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\% Free Space" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk free space (percentage)" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\Free Megabytes" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk free space (MB)" locale="pt-BR"/>
	      </PerformanceCounterConfiguration>
	    </PerformanceCounters>
		<Metrics resourceId="(Update with resource ID for the VM)" >
	        <MetricAggregation scheduledTransferPeriod="PT1H"/>
	        <MetricAggregation scheduledTransferPeriod="PT1M"/>
	    </Metrics>
	    <WindowsEventLog scheduledTransferPeriod="PT1M">
	      <DataSource name="Application!*[System[(Level = 1 or Level = 2)]]"/>
	      <DataSource name="Security!*[System[(Level = 1 or Level = 2)]"/>
	      <DataSource name="System!*[System[(Level = 1 or Level = 2)]]"/>
	    </WindowsEventLog>
	      </DiagnosticMonitorConfiguration>
	    </WadCfg>
	    <StorageAccount>(Update with diagnostics storage account name)</StorageAccount>
	</PublicConfig>
	```

## Próximas etapas
- Para obter diretrizes adicionais sobre como usar a capacidade de Diagnóstico do Azure e outras técnicas para solucionar problemas, consulte [Habilitar o diagnóstico nos serviços de nuvem do Azure e máquinas virtuais](cloud-services-dotnet-diagnostics.md).
- O [esquema de configuração de diagnóstico](https://msdn.microsoft.com/library/azure/mt634524.aspx) explica as várias opções de configurações de XML para a extensão de diagnóstico.

<!---HONumber=AcomDC_0121_2016-->