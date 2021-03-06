<properties 
   pageTitle="Criar um balanceador de carga interno usando a CLI do Azure no Gerenciador de Recursos | Microsoft Azure"
   description="Saiba como criar um balanceador de carga interno no Gerenciador de Recursos usando a CLI do Azure"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/09/2016"
   ms.author="joaoma" />

# Introdução à criação de um balanceador de carga interno usando a CLI do Azure

[AZURE.INCLUDE [load-balancer-get-started-ilb-arm-selectors-include.md](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]<BR>[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/learn-about-deployment-models-rm-include.md)] [modelo de implantação clássico](load-balancer-get-started-ilb-classic-cli.md).

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## O que é necessário para criar um balanceador de carga interno?

Você precisa criar e configurar os seguintes objetos para implantar um balanceador de carga:

- Configuração do IP de front-end - configura o endereço IP privado para o tráfego de rede de entrada. 

- Pool de endereços de back-end – contém NICs (Interfaces de Rede) para receber o tráfego do balanceador de carga.

- Regras de balanceamento de carga - contém regras de mapeamento de uma porta de entrada do tráfego de rede para uma porta de recebimento do tráfego de rede no pool de back-end.

- Regras NAT de entrada – contém regras de mapeamento de uma porta pública no balanceador de carga para uma porta de uma máquina virtual específica no pool de endereços de back-end.

- Investigações – contém investigações de integridade usadas para verificar a disponibilidade de máquinas virtuais associadas ao pool de endereços de back-end.

É possível obter mais informações sobre componentes do balanceador de carga com o gerenciador de recursos do Azure em [Suporte do Gerenciador de Recursos do Azure para balanceador de carga](load-balancer-arm.md).

## Configurar a CLI para usar o Gerenciador de Recursos

1. Se você nunca usou a CLI do Azure, consulte [Instalar e configurar a CLI do Azure](xplat-cli.md) e siga as instruções até o ponto em que você seleciona sua conta e assinatura do Azure.

2. Execute o comando **azure config mode** para alternar para o modo do Gerenciador de Recursos, como mostrado abaixo.

		azure config mode arm

	Saída esperada:

		info:    New mode is arm

## Passo a passo da criação de um balanceador de carga interno 

As seguintes etapas criarão um balanceador de carga interno com base no cenário acima:

### Etapa 1 

Se você ainda não fez isso, baixe a versão mais recente da [interface de linha de comando do Azure](https://azure.microsoft.com/downloads/).

### Etapa 2 

Depois de instalá-la, autentique sua conta.

	azure login

O processo de autenticação solicitará seu nome de usuário e a senha de sua assinatura do Azure.

### Etapa 3

Altere as ferramentas de comando para o modo do gerenciador de recursos do Azure.

	azure config mode arm

## Criar um grupos de recursos

Todos os recursos do gerenciador de recursos do Azure estão associados a um grupo de recursos. Se você ainda não fez isso, crie um grupo de recursos.

	azure group create <resource group name> <location>


## Criar um conjunto do balanceador de carga interno 


### Etapa 1 

Crie um balanceador de carga interno usando o comando `azure network lb create`. No cenário a seguir, é criado um grupo de recursos chamado nrprg na região Leste dos EUA.
 	
	azure network lb create -n nrprg -l westus

>[AZURE.NOTE] todos os recursos de um balanceador de carga interno, como uma rede virtual e sub-rede da rede virtual, devem estar no mesmo grupo de recursos e na mesma região.


### Etapa 2 

Crie um endereço IP de front-end para o balanceador de carga interno. O endereço IP usado deve estar dentro do intervalo da sub-rede de sua rede virtual.

	
	azure network lb frontend-ip create -g nrprg -l ilbset -n feilb -a 10.0.0.7 -e nrpvnetsubnet -m nrpvnet

Parâmetros usados:

**-g** - grupo de recursos **-l** - nome do conjunto do balanceador de carga interno **-n** - nome do IP de front-end **-a** - endereço IP privado dentro do intervalo da sub-rede. **-e** - nome da sub-rede **-m** - nome da rede virtual

### Etapa 3 

Crie o pool de endereços de back-end.

	azure network lb address-pool create -g nrprg -l ilbset -n beilb

Parâmetros usados:

**-g** - grupo de recursos **-l** - nome do conjunto do balanceador de carga interno **-n** - nome do pool de endereços de back-end

Depois de definir um endereço IP de front-end e um pool de endereços de back-end, você poderá criar regras de balanceador de carga, regras NAT de entrada e personalizar as investigações de integridade.

### Etapa 4


Crie uma regra do balanceador de carga para o balanceador de carga interno. Seguindo o cenário acima, o comando cria uma regra do balanceador de carga que escuta a porta 1433 no pool de front-end e envia o tráfego de rede com carga balanceada ao pool de endereços de back-end também usando a porta 1433.

	azure network lb rule create -g nrprg -l ilbset -n ilbrule -p tcp -f 1433 -b 1433 -t feilb -o beilb

Parâmetros usados:

**-g** - grupo de recursos **-l** - nome do conjunto do balanceador de carga interno **-n** - nome da regra do balanceador de carga **-p** - protocolo usado para a regra **-f** - porta que escuta o tráfego de rede de entrada no front-end do balanceador de carga **-b** - porta que recebe o tráfego de rede no pool de endereços de back-end

### Etapa 5

Crie regras NAT de entrada. Regras NAT de entrada são usadas para criar pontos de extremidade em um balanceador de carga que irão para uma instância de máquina virtual específica. Seguindo o exemplo acima, duas regras NAT foram criadas para o acesso de área de trabalho remota.

	azure network lb inbound-nat-rule create -g nrprg -l ilbset -n NATrule1 -p TCP -f 5432 -b 3389
	
	azure network lb inbound-nat-rule create -g nrprg -l ilbset -n NATrule2 -p TCP -f 5433 -b 3389

Parâmetros usados:

**-g** - grupo de recursos **-l** - nome do conjunto de balanceadores de carga internos **-n** - nome da regra NAT de entrada **-p** - protocolo usado para a regra **-f** - porta que escuta o tráfego de rede de entrada no front-end do balanceador de carga **-b** - porta que recebe o tráfego de rede no pool de endereços de back-end

### Etapa 5 

Crie investigações de integridade para o balanceador de carga. Uma investigação de integridade verifica todas as instâncias da máquina virtual para se certificar de que ela pode enviar o tráfego de rede. A instância de máquina virtual com verificações de investigação com falha é removida do balanceador de carga até ele ficar online novamente e as verificações de investigação se tornarem íntegras.

	azure network lb probe create -g nrprg -l ilbset -n ilbprobe -p tcp -i 300 -c 4

**-g** - grupo de recursos **-l** - nome do conjunto de balanceadores de carga internos **-n** - nome do teste de integridade **-p** - protocolo usado pelo teste de integridade **-i** - intervalo de investigação em segundos **-c** - número de verificações

>[AZURE.NOTE] A plataforma Microsoft Azure usa um endereço IPv4 estático e publicamente roteável para uma variedade de cenários administrativos. O endereço IP é 168.63.129.16. Esse endereço IP não deve ser bloqueado por nenhum firewall porque ele pode causar um comportamento inesperado. Em relação ao Balanceamento de Carga Interno do Azure, esse endereço IP é usado por testes de monitoramento do balanceador de carga para determinar o estado de integridade para máquinas virtuais em um conjunto com balanceamento de carga. Se um grupo de segurança de rede é usado para restringir o tráfego para máquinas virtuais do Azure em um conjunto com balanceamento de carga interno, ou então é aplicado a uma Sub-rede de Rede Virtual, certifique-se de que uma regra de segurança de rede seja adicionada para permitir o tráfego em 168.63.129.16.

## Criar NICs

Você precisa criar NICs (ou modificar as existentes) e associá-las a regras NAT, regras do balanceador de carga e testes.

### Etapa 1 

Crie uma NIC chamada *lb-nic1-be* e a associe à regra NAT *rdp1* e ao pool de endereços de back-end *beilb*.
	
	azure network nic create -g nrprg -n lb-nic1-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1" eastus

Parâmetros:

- **-g** - nome do grupo de recursos
- **-n** - nome do recurso NIC
- **--subnet-name** - nome da sub-rede 
- **--subnet-vnet-name** - nome da rede virtual
- **-d** - ID do recurso de pool back-end - começa com /subscription/{subscriptionID/resourcegroups/<resourcegroup-name>/providers/Microsoft.Network/loadbalancers/<load-balancer-name>/backendaddresspools/<name-of-the-backend-pool> 
- **-e** – ID da regra NAT que será associada ao recurso NIC – começa com /subscriptions/####################################/resourceGroups/<resourcegroup-name>/providers/Microsoft.Network/loadBalancers/<load-balancer-name>/inboundNatRules/<nat-rule-name>


Saída esperada:

	info:    Executing command network nic create
	+ Looking up the network interface "lb-nic1-be"
	+ Looking up the subnet "nrpvnetsubnet"
	+ Creating network interface "lb-nic1-be"
	+ Looking up the network interface "lb-nic1-be"
	data:    Id                              : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
	data:    Name                            : lb-nic1-be
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : eastus
	data:    Provisioning state              : Succeeded
	data:    Enable IP forwarding            : false
	data:    IP configurations:
	data:      Name                          : NIC-config
	data:      Provisioning state            : Succeeded
	data:      Private IP address            : 10.0.0.4
	data:      Private IP Allocation Method  : Dynamic
	data:      Subnet                        : /subscriptions/####################################/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet
	data:      Load balancer backend address pools
	data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool
	data:      Load balancer inbound NAT rules:
	data:        Id                          : /subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1
	data:
	info:    network nic create command OK

### Etapa 2

Crie uma NIC chamada *lb-nic2-be* e a associe à regra NAT *rdp2* e ao pool de endereços de back-end *beilb*.

 	azure network nic create -g nrprg -n lb-nic2-be --subnet-name nrpvnetsubnet --subnet-vnet-name nrpvnet -d "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/beilb" -e "/subscriptions/####################################/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp2" eastus

### Etapa 3 

Crie uma VM (máquina virtual) chamada *DB1* e a associe à NIC chamada *lb-nic1-be*. Uma conta de armazenamento chamada *web1nrp* foi criada antes da execução do comando a seguir.

	azure vm create --resource-group nrprg --name DB1 --location eastus --vnet-name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic1-be --availset-name nrp-avset --storage-account-name web1nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

>[AZURE.IMPORTANT] As VMs em um balanceador de carga precisam estar no mesmo conjunto de disponibilidade. Use `azure availset create` para criar um conjunto de disponibilidade.

### Etapa 4

Crie uma VM (máquina virtual) chamada *DB2* e a associe à NIC chamada *lb-nic2-be*. Uma conta de armazenamento chamada *web1nrp* foi criada antes da execução do comando abaixo.

	azure vm create --resource-group nrprg --name DB2 --location eastus --vnet-	name nrpvnet --vnet-subnet-name nrpvnetsubnet --nic-name lb-nic2-be --availset-name nrp-avset --storage-account-name web2nrp --os-type Windows --image-urn MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:4.0.20150825

## Excluir um balanceador de carga 


Para remover um balanceador de carga, use o comando a seguir

	azure network lb delete -g nrprg -n ilbset 

Em que **nrprg** é o grupo de recursos e **ilbset** o nome do balanceador de carga interno.


## Próximas etapas

[Configurar um modo de distribuição do balanceador de carga usando a afinidade de IP de origem](load-balancer-distribution-mode.md)

[Definir configurações de tempo limite de TCP ocioso para o balanceador de carga](load-balancer-tcp-idle-timeout.md)

<!----HONumber=AcomDC_0218_2016-->