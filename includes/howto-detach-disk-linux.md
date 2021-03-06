<properties writer="kathydav" editor="tysonn" manager="timlt" />


Quando não precisar mais de um disco de dados conectado a uma máquina virtual, você poderá desanexá-lo facilmente. Essa ação remove o disco da máquina virtual, mas não o remove do armazenamento. Se desejar usar os dados existentes no disco novamente, você pode reanexá-lo à mesma máquina virtual ou anexá-lo a uma outra máquina virtual.

> [AZURE.NOTE] Uma máquina virtual no Azure usa diferentes tipos de discos -- um disco de sistema operacional, um disco temporário local e discos de dados opcionais. Discos de dados são a forma recomendada de armazenar dados para uma máquina virtual. Para obter detalhes, consulte [Sobre discos e VHDs para Máquinas Virtuais](virtual-machines-disks-vhds.md). Não é possível desanexar um disco do sistema operacional, a menos que você também exclua a máquina virtual.

## Localize o disco

Antes de poder desanexar um disco de uma máquina virtual, você precisa conhecer o número LUN, que é um identificador para o disco a ser desanexado. Para fazer isso, siga estas etapas:

1. 	Abra a CLI do Azure para Mac, Linux, e Windows e conecte-se à sua assinatura do Azure. Consulte [Conectar ao Azure da CLI do Azure](../articles/xplat-cli-connect.md) para obter mais informações.

2.  Verifique se você está no modo de gerenciamento de serviços do Azure, que é o padrão digitando `azure config
 	mode asm`.

3. 	Descubra quais discos estão anexados à sua máquina virtual usando `azure vm disk list
	<virtual-machine-name>` da seguinte maneira:

		$azure vm disk list ubuntuVMasm
		info:    Executing command vm disk list
		+ Fetching disk images
		+ Getting virtual machines
		+ Getting VM disks
		data:    Lun  Size(GB)  Blob-Name                         OS
		data:    ---  --------  --------------------------------  -----
		data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
		data:    1    10        test.VHD
		data:    0    30        ubuntuVMasm-76f7ee1ef0f6dddc.vhd
		info:    vm disk list command OK

4. 	Observe o LUN ou o **número de unidade lógica** para o disco que você deseja desanexar.


## Desanexar o disco

Depois de localizar o número LUN do disco, você estará pronto para desanexá-lo:

1. 	Desanexe o disco selecionado da máquina virtual executando o comando `azure vm disk detach
 	<virtual-machine-name> <LUN>` da seguinte forma:

		$azure vm disk detach ubuntuVMasm 0
		info:    Executing command vm disk detach
		+ Getting virtual machines
		+ Removing Data-Disk
		info:    vm disk detach command OK

2. 	É possível verificar se o disco foi desanexado executando este comando:

		$azure vm disk list ubuntuVMasm
		info:    Executing command vm disk list
		+ Fetching disk images
		+ Getting virtual machines
		+ Getting VM disks
		data:    Lun  Size(GB)  Blob-Name                         OS
		data:    ---  --------  --------------------------------  -----
		data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
		data:    1    10        test.VHD
		info:    vm disk list command OK

O disco permanece desanexado no armazenamento mas já não está conectado a uma máquina virtual.

<!---HONumber=AcomDC_0204_2016-->