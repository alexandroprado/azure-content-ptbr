<properties 
   pageTitle="Configurar uma conexão VPN entre duas redes virtuais | Microsoft Azure" 
   description="Saiba como configurar conexões VPN e resolução de nome de domínio entre duas redes virtuais do Azure e como configurar a replicação geográfica do HBase." 
   services="hdinsight,virtual-network" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="12/02/2015"
   ms.author="jgao"/>

# Configurar uma conexão VPN entre duas redes virtuais do Azure  

> [AZURE.SELECTOR]
- [Configure VPN connectivity](../hdinsight-hbase-geo-replication-configure-VNETs.md)
- [Configure DNS](hdinsight-hbase-geo-replication-configure-DNS.md)
- [Configure HBase replication](hdinsight-hbase-geo-replication.md) 

A conectividade de site a site de rede virtual do Azure usa um gateway VPN para fornecer um túnel seguro usando Ipsec/IKE. Os VNets podem estar em diferentes regiões e diferentes assinaturas. Você pode até combinar a comunicação VNet à VNet com configurações multissite. Há vários motivos para a conectividade VNet para VNet:

- Redundância geográfica entre regiões e presença geográfica 
- Aplicativos multicamadas regionais com limite de isolamento forte 
- Comunicação entre organizações e assinaturas no Azure

Para obter mais informações, consulte [Configurar uma conexão VNet com VNet](../virtual-network/virtual-networks-configure-vnet-to-vnet-connection.md).

Para ver em vídeo:

> [AZURE.VIDEO configure-the-vpn-connectivity-between-two-azure-virtual-networks]

Este tutorial faz parte da [série][hdinsight-hbase-replication] sobre a criação de replicação geográfica do HBase.

- Configurar uma conectividade VPN entre duas redes virtuais (este tutorial)
- [Configurar o DNS para as redes virtuais][hdinsight-hbase-geo-replication-dns]
- [Configurar a replicação geográfica HBase][hdinsight-hbase-geo-replication]

O diagrama a seguir ilustra as duas redes virtuais que você criará neste tutorial:

![Diagrama de rede virtual de replicação de HBase do HDInsight][img-vnet-diagram]
 

##Pré-requisitos
Antes de começar este tutorial, você deve ter o seguinte:

- **Uma assinatura do Azure**. Consulte [Obter avaliação gratuita do Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

- **Uma estação de trabalho com o PowerShell do Azure**. Consulte [Instalar e usar o PowerShell do Azure](https://azure.microsoft.com/documentation/videos/install-and-use-azure-powershell/).

	Antes de executar scripts do PowerShell, verifique se você está conectado à sua assinatura do Azure usando o seguinte cmdlet:

		Add-AzureAccount

	Se você tiver várias assinaturas do Azure, use o seguinte cmdlet para definir a assinatura atual:

		Select-AzureSubscription <AzureSubscriptionName>


>[AZURE.NOTE] Os nomes de serviço do Azure e a máquina virtual devem ser exclusivos. O nome usado neste tutorial é Contoso-[Azure Service/VM name]-[EU/US]. Por exemplo, Contoso-VNet-EU é a rede virtual do Azure no datacenter do Norte da Europa; Contoso-DNS-US é a VM do servidor DNS no datacenter no Leste dos EUA. Você deve criar seus próprios nomes.
 

##Crie dois VNets do Azure



**Para criar uma rede virtual chamada Contoso-VNet-EU no Norte da Europa**

1.	Entre no [Portal Clássico do Azure][azure-portal].
2.	Clique em **NOVO**, **SERVIÇOS DE REDE**, **REDE VIRTUAL**, **CRIAÇÃO PERSONALIZADA**.
3.	Digite:

	- **NOME**: Contoso-VNet-EU
	- **LOCALIZAÇÃO**: Norte da Europa

		Este tutorial usa data centers do Norte da Europa e do Leste dos EUA. Você pode escolher seus próprios data centers.
4.	Digite:

	- **SERVIDOR DNS**: (deixe em branco) 
	
		Você precisará de seu próprio servidor DNS para resolução de nomes em redes virtuais. Para obter mais informações sobre a resolução do nome fornecidas pelo Azure e quando usar seu próprio servidor DNS, consulte [Resolução do nome (DNS)](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Para obter instruções para configurar a resolução de nomes entre VNets, consulte [Configurar DNS entre duas redes virtuais do Azure][hdinsight-hbase-dns].
  
	- **Configurar uma VPN ponto a site**: (desmarcado)

		Ponto a site não se aplica a esse cenário.

 	- **Configurar uma VPN site a site**: (desmarcado)
 	
		Você irá configurar a conexão de VPN site a site na rede virtual do Azure no data center Leste dos EUA.
5.	Digite:

	- 	**IP INICIAL DO ESPAÇO DE ENDEREÇO**: 10.1.0.0
	- 	**CIDR DO ESPAÇO DE ENDEREÇO**: /16
	- 	**Sub-rede-1 STARTING IP**: 10.1.0.0
	- 	**Sub-rede-1 CIDR**: /24

	O espaço de endereço não pode se sobrepor à rede virtual dos EUA.

**Para criar uma rede virtual chamada Contoso-VNet-EU no Oeste da Europa**

- Repetir o último procedimento com os seguintes valores:

	- **NOME**: Contoso-VNet-US
	- **LOCAL**: Leste dos EUA
	 
	- **SERVIDOR DNS**: (deixe em branco)
	- **Configurar uma VPN ponto a site**: (desmarcado)
	- **Configurar uma VPN site a site**: (desmarcado)
	 
	- **IP INICIAL DO ESPAÇO DE ENDEREÇO**: 10.2.0.0
	- **CIDR DO ESPAÇO DE ENDEREÇO**: /16
	- **Sub-rede-1 STARTING IP**: 10.2.0.0
	- **Sub-rede-1 CIDR**: /24

















##Configurar uma conexão VPN entre os dois VNets

###Criar redes locais

Ao criar uma configuração de VNet a VNet, você precisa configurar cada VNet para que elas identifiquem umas às outras como um site de rede local. Nesta seção, você configurará cada VNet como uma rede local. As redes locais compartilham os mesmos espaços de endereço IP com o VNet correspondente.

![Definir a configuração de site a site VPN do Azure - redes locais do Azure][img-vnet-lnet-diagram]


**Para criar uma rede local chamada Contoso-LNet-EU correspondente ao espaço de endereço de rede da Contoso-VNet-EU**

1. No Portal Clássico do Azure, clique em **NOVO**, **SERVIÇOS DE REDE**, **REDE VIRTUAL**, **ADICIONAR REDE LOCAL**.
3. Digite:

	- **NOME**: Contoso-LNet-EU
	- **ENDEREÇO IP DO DISPOSITIVO VPN**: 192.168.0.1 (esse endereço será atualizado posteriormente)

		Normalmente, você usaria o endereço IP externo real para um dispositivo VPN. Para configurações de VNet a VNet, você usará o endereço IP do Gateway de VPN. Considerando que você ainda não criou os gateways de VPN para as duas VNets, insira um endereço IP arbitrário e volte para corrigi-lo.
4.	Digite:

	- **IP INICIAL DO ESPAÇO DE ENDEREÇO:** 10.1.0.0
	- **CIDR DO ESPAÇO DE ENDEREÇO:** /16
	
	Isso deve corresponder exatamente ao intervalo que você especificou anteriormente para Contoso-VNet-EU.

**Para criar uma rede local chamada Contoso-LNet-US correspondendo ao espaço de endereço de rede da Contoso-VNet-US**

- Repita o último procedimento com os seguintes valores:

	- **NOME**: Contoso-LNet-US
	- **ENDEREÇO IP DO DISPOSITIVO VPN**: 192.168.0.1 (esse endereço será atualizado posteriormente)
	 
	- **IP INICIAL DO ESPAÇO DE ENDEREÇO**: 10.2.0.0
	- **CIDR DO ESPAÇO DE ENDEREÇO**: /16


###Criar gateways VPN

Há duas partes nessa configuração. Primeiro, configure uma conexão de site a site VNet com uma rede local e, em seguida, você pode criar um VPN de roteamento dinâmico. VNet a VNet requer gateways de VPN do Azure com VPNs de roteamento dinâmico. Não há suporte para as VPNs do Azure com roteamento estático.

**Para configurar a conexão de site a site Contoso-VNet-EU para Contoso-LNet-US**

1.	No Portal Clássico do Azure, clique em **REDES** no painel esquerdo.
2.	Clique em **Contoso-VNet-EU**.
3.	Clique na guia **CONFIGURAR**.
4.	Marque **Conectar à rede local**.
5.	Em **REDE LOCAL**, selecione **Contoso-LNet-US**.
6.	Clique em **Adicionar sub-rede de gateway** na seção de espaços de endereço de rede virtual.
7.	Clique em **SALVAR**.
8.	Clique em **OK** para confirmar.


**Para criar um gateway VPN para Contoso-VNet-EU**

1.	No Portal Clássico do Azure, clique na guia **PAINEL**.
4.	Clique em **CRIAR GATEWAY** na parte inferior da página e clique em **Roteamento dinâmico**.
5.	Clique em **Sim** para confirmar. Observe que o gráfico do gateway na página muda para a cor amarelo e diz Criando Gateway. Geralmente, leva cerca de 15 minutos para que o gateway seja criado.

	Quando o status do gateway mudar para Conectando, o endereço IP de cada Gateway estará visível no Painel. Anote o endereço IP que corresponde a cada VNet, tomando cuidado para não os misturar. Esses são os endereços IP que serão usados quando você editar seus endereços IP de espaço reservado para o Dispositivo VPN em Redes Locais.

6.	Faça uma cópia do **ENDEREÇO IP DO GATEWAY**. Você o usará para configurar o endereço IP do gateway VPN para Contoso-VNet-EU na próxima seção.

**Para criar um gateway VPN para Contoso-VNet-EU**

- Repita os dois últimos procedimentos para configurar a conectividade site a site do Contoso-VNet-US para o Contoso-LNet-EU e a criar um gateway VPN para o Contoso-Vnet-US. Quando terminar, você terá o endereço IP do gateway VPN para o Contoso-VNet-US.


### Defina os endereços IP do dispositivo VPN para redes locais
Na última seção, você pode criar um gateway VPN para cada um dos VNets. Você tem os endereços IP dos gateways de VPN. Agora, você pode voltar para configurar endereços IP de dispositivos VPN da rede local.

**Para configurar o endereço IP do dispositivo VPN para o Contoso-LNet-EU**

1.	No Portal Clássico do Azure, clique em **REDES** no painel esquerdo.
2.	Clique em **REDES LOCAIS** na parte superior.
3.	Clique em **Contoso-LNet-EU** e, em seguida, clique em **EDITAR** na parte inferior.
4.	Atualizar **ENDEREÇO IP DO DISPOSITIVO DE VPN**. Este é o endereço obtido na guia PAINEL do Contoso-VNET-EU.
5.	Clique no botão direito.
6.	Clique no botão de seleção.

**Para configurar o endereço IP do dispositivo VPN para o Contoso-LNet-US**

- Repita o último procedimento para configurar o endereço IP do dispositivo VPN para Contoso-LNet-US.

###Definir chaves de gateway de VNet

Os gateways de Vnet usam uma chave compartilhada para autenticar conexões entre as redes virtuais. A chave não pode ser configurada no Portal Clássico do Azure. Você deve usar o PowerShell ou o SDK do .NET.

**Para definir as chaves**

1. Na estação de trabalho, abra o **ISE do Windows PowerShell** ou o console do Windows PowerShell.
2. Atualize os parâmetros neste script de acompanhamento e execute-o:

		Add-AuzreAccount
		Select-AzureSubscription -[AzureSubscriptionName]
		Set-AzureVNetGatewayKey -VNetName ContosoVNet-EU -LocalNetworkSiteName Contoso-LNet-US -SharedKey A1b2C3D4
		Set-AzureVNetGatewayKey -VNetName ContosoVNet-US -LocalNetworkSiteName Contoso-LNet-EU -SharedKey A1b2C3D4 


##Verifique a conexão VPN 

Sem nenhuma VM implantada nas VNets, você pode usar o diagrama visual da rede virtual na página do Painel da VNet no Portal Clássico do Azure para verificar o status da conexão:

![Status da conexão VPN da rede virtual de replicação de HBase do HDInsight][img-vpn-status]
  



##Próximas etapas

Neste tutorial, você aprendeu como configurar uma conexão VPN entre duas redes virtuais do Azure. Os outros dois artigos da série abrangem:

- [Configurar o DNS entre duas redes virtuais do Azure][hdinsight-hbase-geo-replication-dns]
- [Configurar a replicação geográfica HBase][hdinsight-hbase-geo-replication]



[hdinsight-hbase-geo-replication-dns]: hdinsight-hbase-geo-replication-configure-dns.md
[hdinsight-hbase-geo-replication]: hdinsight-hbase-geo-replication.md


[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[azure-portal]: https://portal.azure.com

[powershell-install]: ../install-configure-powershell



[hdinsight-hbase-replication]: ../hdinsight-hbase-geo-replication/
[hdinsight-hbase-dns]: ../hdinsight-hbase-geo-replication-configure-DNS/


[img-vnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.diagram.png
[img-vnet-lnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.LNet.diagram.png
[img-vpn-status]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.status.png

<!---HONumber=AcomDC_0128_2016-->