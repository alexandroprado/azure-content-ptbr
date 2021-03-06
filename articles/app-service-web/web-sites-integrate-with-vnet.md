<properties 
	pageTitle="Integrar um aplicativo a uma Rede Virtual do Azure" 
	description="Mostra como conectar um aplicativo do Azure no Serviço de Aplicativo do Azure a uma rede virtual do Azure nova ou existente" 
	services="app-service" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor="cephalin"/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/22/2016" 
	ms.author="ccompy"/>

# Integrar seu aplicativo Web a uma Rede Virtual do Azure #

Este documento descreve o recurso de visualização de integração de rede virtual do Serviço de Aplicativo do Azure e mostra como configurá-lo com os aplicativos no [Serviço de Aplicativo do Azure](http://go.microsoft.com/fwlink/?LinkId=529714). Se você estiver familiarizado com VNETs (Redes Virtuais do Azure), isso é um recurso que permite colocar muitos dos recursos do Azure em uma rede que não pode ser roteada pela Internet, cujo acesso você pode controlar. Essas redes podem ser conectadas às redes locais usando uma variedade de tecnologias de VPN. Para saber mais sobre redes virtuais do Azure, confira [Visão geral da Rede Virtual do Azure][VNETOverview].

O Serviço de Aplicativo do Azure tem duas formas. A primeira forma, e a mais comum, é os sistemas de multilocatários que dão suporte à gama completa de planos de preço. A segunda é o recurso premium do ASE (Ambiente do Serviço de Aplicativo), que implanta em uma VNET de clientes. Com um Ambiente de Serviço de Aplicativo, normalmente não é necessário usar a integração VNET, já o sistema já está em sua VNET e tem acesso a todos os recursos nela. O único motivo para usar o recurso de integração VNET com um ASE seria se você desejasse acessar recursos em outra VNET que não esteja conectada à VNET que hospeda seu ASE.

A integração VNET concede ao seu aplicativo Web acesso a recursos da sua rede virtual, mas não concede acesso privado ao seu aplicativo Web por meio da rede virtual. Um cenário comum onde você usaria esse recurso seria habilitar o acesso de seu aplicativo Web a um banco de dados ou serviços Web que são executados em uma máquina virtual na sua rede virtual do Azure. Com a integração VNET, você não precisa expor um ponto de extremidade público para aplicativos em sua VM, mas pode usar, em vez disso, endereços roteáveis privados fora da Internet.

O recurso de integração VNET:

- requer um plano de preços Standard ou Premium 
- atualmente só funciona com V1 ou VNETs clássicas 
- dá suporte a TCP e UDP
- funciona com aplicativos Web, móveis e de API
- permite que um aplicativo se conecte somente a uma VNET por vez
- permite a integração de até cinco VNETs com um Plano de Serviço de Aplicativo 
- permite que a mesma VNET seja usada por vários aplicativos em um Plano de Serviço de Aplicativo
- dá suporte a um SLA de 99,9% devido a uma dependência no Gateway de VNET

Há algumas coisas a que a integração VNET não dá suporte, incluindo:

- montagem de unidade
- integração ao AD 
- NetBios
- acesso a site privado

### Introdução ###
Veja aqui algumas coisas para se ter em mente antes de conectar seu aplicativo Web a uma rede virtual:

- A integração VNET só funciona com aplicativos em um plano de preços **Standard** ou **Premium**. Se você habilitar o recurso e dimensionar seu Plano de Serviço de Aplicativo para um plano de preços sem suporte, seus aplicativos perderão suas conexões com as VNETs que estão usando.  
- Se a sua rede virtual de destino já existe, ela deve ter a VPN ponto a site habilitada com um gateway de roteamento dinâmico antes que possa ser conectada a um aplicativo. Você não pode habilitar a opção ponto a site da VPN (rede virtual privada) com se o gateway está configurado com roteamento estático.
- A VNET deve estar na mesma assinatura que o ASP (Plano de Serviço de Aplicativo).  
- Os aplicativos que se integram a uma VNET usarão o DNS específico daquela VNET.
- Por padrão, os aplicativos integrados apenas rotearão o tráfego em sua VNET baseados nas rotas que são definidas em sua VNET.  


## Habilitando a integração VNET ##

Você tem a opção de se conectar a uma rede virtual nova ou existente. Se criar uma nova rede, além de criar a VNET, um gateway de roteamento dinâmico será pré-configurado para você e a VPN ponto a Site será habilitada.

>[AZURE.NOTE] A configuração de uma nova integração da rede virtual pode levar vários minutos.

Para habilitar a integração VNET, abra as Configurações do aplicativo configurações e selecione Rede. A interface do usuário oferece três opções de rede. Este guia só aborda a integração VNET, mas as conexões híbridas e ambientes de serviço de aplicativo são discutidos neste documento.

Se seu aplicativo não estiver no plano de preços correto, a interface do usuário permitirá que você escalone seu plano para o plano de preços mais alto que queira escolher.


![][1]
 
###Habilitando a integração VNET com uma VNET já existente###
A interface de usuário de integração de rede virtual permite que você selecione em uma lista de suas VNET V1. Na imagem abaixo, você pode ver que somente uma VNET pode ser selecionada. Há vários motivos para uma VNET ficar esmaecida, incluindo:

- a rede virtual está em outra assinatura à qual sua conta não tem acesso
- a VNET não tem um ponto a Site habilitado
- a VNET não tem um gateway de roteamento dinâmico

Também vale a pena observar que, uma vez que ainda não damos suporte à integração com VNETs V2, elas não são listadas.

![][2]

Para habilitar a integração, simplesmente clique na VNET que deseja integrar. Depois de selecionar a VNET, o aplicativo será reiniciado automaticamente para que as alterações entrem em vigor.

Se sua VNET não tem um gateway ou ponto a site, você precisa configurar isso primeiro. Para fazer isso, vá para o Portal do Azure e exiba a lista de redes virtuais (clássico). Em seguida, clique na rede com que você deseja integrar e clique na caixa grande em Essentials chamada Conexões VPN. A partir daqui, você pode criar sua VPN ponto a site e até mesmo fazê-la criar um gateway. Depois de passar pelo ponto a site com a experiência de criação do gateway, demora cerca de 30 minutos até que ele fique pronto.

![][8]

### Criando uma rede virtual e fazendo a integração com ela ###
Se quiser criar uma nova rede virtual, lembre-se de que atualmente você só pode criar uma V1 ou VNET clássica. Para criar uma rede virtual V1 por meio de interface do usuário de integração de rede virtual, simplesmente selecione Criar nova rede virtual e forneça o nome desejado para a rede virtual e seu espaço de endereço.

Lembre-se de que se você quiser que essa rede virtual se conecte a qualquer outra rede, deverá tentar evitar escolher um espaço de endereço IP que se sobreponha a essas redes.

>[AZURE.NOTE] Pode levar até 30 minutos para uma nova rede virtual ser fornecida com gateways operacionais. A interface do usuário será atualizada quando ela for concluída.

![][3]

As redes virtuais do Azure normalmente são criadas dentro de endereços de rede privada. Por padrão, o recurso Integração de rede virtual roteará o tráfego destinado aos intervalos de endereços IP em sua rede virtual. Os intervalos de endereços IP privados são:

- 10\.0.0.0/8 - esse é o mesmo que 10.0.0.0 - 10.255.255.255
- 172\.16.0.0/12 - é o mesmo que 172.16.0.0 - 172.31.255.255 
- 192\.168.0.0/16 - é o mesmo que 192.168.0.0 - 192.168.255.255
 
O espaço de endereço de rede virtual precisa ser especificado na notação CIDR. Se você não estiver familiarizado com notação CIDR, é um método para especificar blocos de endereço usando um endereço IP e um número inteiro que representa a máscara de rede. Como uma referência rápida, considere que 10.1.0.0/24 seria 256 endereços e 10.1.0.0/25 seria 128 endereços. Um endereço IPv4 com um /32 seria apenas 1 endereço.

Se você definir as informações do servidor DNS aqui, elas serão definidas para sua rede virtual. Após a criação da rede virtual, você poderá editar essas informações nas experiências de usuário da rede virtual.

Quando você cria uma VNET V1 usando a interface de usuário de integração de rede virtual, ela criará uma rede virtual no mesmo grupo de recursos do seu aplicativo.

## Como o sistema funciona ##
De modo oculto este recurso se baseia na tecnologia VPN Ponto a site para se conectar ao aplicativo à sua VNET. Aplicativos no Serviço de Aplicativo do Azure têm uma arquitetura de sistema multilocatário, que impede o provisionamento de um aplicativo diretamente em uma VNET, como ocorre com máquinas virtuais. Ao desenvolver uma tecnologia ponto a site, limitamos o acesso à rede apenas à máquina virtual que está hospedando o aplicativo. O acesso à rede é ainda mais limitado nestes hosts do aplicativo, de modo que seus aplicativos poderão acessar apenas as redes que você configurar para eles acessarem.

![][4]
 
Se você ainda não configurou um servidor DNS com sua rede virtual será necessário usar endereços IP. Ao usar endereços IP, lembre-se de que a grande vantagem desse recurso é que ele permite que você use os endereços privados em sua rede privada. Se você configurar seu aplicativo para usar endereços IP públicos em uma das suas VMs, não estará usando o recurso Integração de rede virtual e estará se comunicando pela internet.



##Gerenciando as integrações VNET##

A capacidade de se conectar e desconectar de uma rede virtual está no nível dos aplicativos. As operações que podem afetar a integração da rede virtual entre vários aplicativos estão no nível do ASP. Você pode obter detalhes sobre sua rede virtual na interface do usuário que é mostrada no nível do aplicativo. A maioria dessas informações também é mostrada no nível do ASP.

![][5]

Na página Status do Recurso de Rede, você pode ver se seu aplicativo está conectado à sua rede virtual. Se o gateway da rede virtual está inativo por qualquer motivo, ele aparece como não conectado.

As informações que você tem no aplicativo na interface do usuário da Integração VNET no nível do aplicativo são as mesmas detalhadas originadas no ASP. Estes são os itens:

- Nome da VNET – esse link abre a interface do usuário da rede
- Local – reflete o local da sua rede virtual. É possível fazer a integração a uma rede virtual em outro local.
- Status do certificado – há certificados usados para proteger a conexão VPN entre o aplicativo e a rede virtual. Isso é um teste para garantir que eles estejam em sincronia.
- Status do gateway – se os gateways ficarem inativos por algum motivo, seu aplicativo não poderá acessar recursos na rede virtual.  
- Espaço de endereço da VNET – é o espaço de endereço IP para sua rede virtual.  
- Espaço de endereço ponto a site – é o espaço de endereço IP ponto a site para sua rede virtual. Seu aplicativo mostrará a comunicação como proveniente de um dos IPs nesse espaço de endereço.  
- Espaço de endereço site a site – você pode usar as VPNs site a site para conectar sua rede virtual a seus recursos locais ou a outras redes virtuais. Se estiver configurado, os intervalos de IP definidos com aquela conexão VPN serão exibidos aqui.
- Servidores DNS – se você tiver servidores DNS configurados com sua rede virtual, eles serão listados aqui.
- IPs roteados para a rede virtual – há uma lista de endereços IP para os quais sua rede virtual definiu o roteamento. Esses endereços serão exibidos aqui.  

A única operação que você pode executar no modo de exibição de aplicativo da integração de rede virtual é desconectar-se da rede virtual à qual seu aplicativo está atualmente conectado. Para fazer isso, simplesmente clique em Desconectar na parte superior. Essa ação não altera sua rede virtual. A VNET e sua configuração, incluindo os gateways, permanece inalterada. Se você quiser excluir sua rede virtual, precisa primeiro excluir os recursos nela, incluindo os gateways.

O modo de exibição do Plano de Serviço de Aplicativo tem algumas operações adicionais. Ele também acesso diferente do aplicativo. Para acessar a interface do usuário da rede do ASP basta abrir sua interface do usuário do ASP e rolar para baixo. Há um elemento de interface do usuário chamado Status do Recurso de Rede. Ele fornecerá alguns pequenos detalhes sobre a integração da rede virtual. Clicar nessa interface do usuário abre a interface do usuário Status de Recurso de Rede. Se você clicar em "Clique aqui para gerenciar", abrirá a interface do usuário que lista as integrações de rede virtual nesse ASP.

![][6]

É bom lembrar do local do ASP ao examinar os locais das redes virtuais com que você estiver integrando. Quando a rede virtual está em outro local, a probabilidade de perceber os problemas de latência é muito menor.

As redes virtuais integradas são um lembrete de quantas redes virtuais estão integradas com seus aplicativos no ASP e quantas você pode ter.

Para ver detalhes adicionados em cada rede virtual, basta clicar na rede virtual que lhe interessa. Além dos detalhes que foram observados anteriormente, você também verá uma lista dos aplicativos nesse ASP que estão usando aquela rede virtual.

Em relação a ações, existem duas ações primárias. A primeira é a capacidade de adicionar rotas que direcionam o tráfego que sai de seu aplicativo para sua rede virtual. A segunda ação é a capacidade de sincronizar certificados e informações de rede.

![][7]

**Roteamento** Conforme observado anteriormente, as rotas que são definidas em sua rede virtual são usadas para direcionar o tráfego para sua rede virtual vindo de seu aplicativo. No entanto, há alguns usos, como quando os clientes querem enviar o tráfego de saída adicional de um aplicativo para a rede virtual, e esse recurso é fornecido para eles. O que acontece com o tráfego depois depende de como o cliente configura sua rede virtual.

**Certificados** O status do certificado reflete uma verificação de que está sendo executada pelo Serviço de Aplicativo para validar se os certificados que estamos usando para a conexão VPN ainda estão válidos. Ao habilitar a integração de rede virtual, se essa for a primeira integração com a rede virtual de qualquer aplicativos nesse ASP, há uma troca de certificados necessária para garantir a segurança da conexão. Juntamente com os certificados, obtemos a configuração do DNS, as rotas e outros itens semelhantes que descrevem a rede. Se esses certificados ou informações de rede forem alterados, você precisará clicar em "Sincronizar Rede". **Observação**: quando você clica em "Sincronização Rede", você pode causar uma breve interrupção na conectividade entre o aplicativo e a rede virtual. Embora seu aplicativo não seja reiniciado, a perda de conectividade pode fazer com que seu site não funcione corretamente.

##Acessar recursos locais##

Um dos benefícios do recurso Integração de rede virtual é que, se a rede virtual estiver conectado à rede local com uma VPN Site a Site, seus aplicativos podem ter acesso aos recursos locais usando seu aplicativo. Para isso funcionar, talvez seja necessário atualizar o gateway de VPN local com as rotas para o intervalo de IP ponto a site. Quando o VPN site a site é configurado pela primeira vez, os scripts usados para configurá-lo devem configurar as rotas, incluindo a VPN ponto a site. Se você adicionar a VPN ponto a site depois de criar a VPN site a site, precisará atualizar as rotas manualmente. Os detalhes sobre como fazer isso variam de acordo com o gateway e não são descritos aqui.

>[AZURE.NOTE] Embora o recurso Integração de rede virtual funcione com uma VPN Site a Site para acessar recursos locais, ele atualmente não funciona com uma VPN de Rota Expressa na mesma situação.

##Detalhes de preço##
Existem algumas nuances de preço que você deve levar em conta ao usar o recurso Integração VNET. Há três cobranças relacionadas ao uso desse recurso:

- Requisitos de tipo de preço do ASP
- Custos de transferência de dados
- Custos de gateway de VPN.

Para que seus aplicativos possam usar o recurso, eles precisam estar em um Plano de Serviço de Aplicativo Standard ou Premium. Você pode ver mais detalhes sobre esses custos aqui: [Preço do Serviço de Aplicativo][ASPricing].

Devido à maneira como as VPNs ponto a site são tratadas, você sempre será cobrado por dados de saída pela conexão da Integração VNET, mesmo que ela não esteja no mesmo data center. Para ver quais são essas cobranças, dê uma olhada aqui: [Detalhes de preço da Transferência de Dados][DataPricing].

O último item é o custo dos gateways de rede virtual. Se você não precisa dos gateways para algo como VPNs Site a Site, está pagando para que eles deem suporte ao recurso Integração de rede virtual. Há detalhes sobre esses custos aqui: [Preço de Gateway de VPN][VNETPricing].

##Solucionar problemas##

Embora o recurso seja fácil de configurar, isso não significa que sua experiência estará livre de problemas. Se você encontrar problemas ao acessar o ponto de extremidade desejado, há utilitários que você pode usar para testar a conectividade do console do aplicativo. Há duas experiências de console que você pode usar. Uma é o console Kudu e a outra é o console que você pode acessar no Portal do Azure. Para acessar o console do Kudu em seu aplicativo, vá para Ferramentas -> Kudu. Isso é o mesmo que ir para [nomedosite].scm.azurewebsites.net. Depois de aberto, vá para a guia Console de depuração. Para obter a console hospedado no portal do Azure, em seu aplicativo, vá para Ferramentas -> Console.


####Ferramentas####

As ferramentas ping, nslookup e tracert não funcionarão no console devido a restrições de segurança. Para compensar essa ausência, duas ferramentas separadas foram adicionadas. Para testar a funcionalidade do DNS, adicionamos uma ferramenta chamada nameresolver.exe. A sintaxe do é:

    nameresolver.exe hostname [optional: DNS Server]

Você pode usar a nameresolver para verificar os nomes de host de que seu aplicativo depende. Dessa forma, você pode testar se não há nada mal configurado no DNS ou se não tem acesso ao seu servidor DNS.

A próxima ferramenta permite testar a conectividade do TCP de uma combinação de host e porta. Essa ferramenta é chamada tcpping.exe e a sintaxe é:

    tcpping.exe hostname [optional: port]

Essa ferramenta informará se você pode acessar um host e uma porta específicos, mas não executará a mesma tarefa realizada pelo utilitário ping baseado em ICMP. O utilitário ping ICMP informará se o host estiver em execução. Com tcpping, você descobre se pode acessar uma porta específica em um host.


####Depurando o acesso a recursos hospedados por VNET####

Há várias coisas que podem impedir que seu aplicativo alcance um host e uma porta específicos. Para dividir o problema, comece com coisas simples, como:

- O Gateway é mostrado como estando em execução no Portal?
- Os certificados são mostrados como sincronizados?
- Alguém alterou a configuração de rede sem fazer uma “Sincronização de Rede” nos ASPs afetados?

Se o gateway estiver inativo, ative-o. Se os certificados estiverem fora de sincronia, vá para a exibição ASP da integração da rede virtual e pressione "Sincronizar Rede". Se você suspeitar de que houve uma alteração feita em sua configuração de rede virtual e ela não foi sincronizada com seus ASPs, vá para o modo de exibição ASP da Integração VNET e pressione “Sincronizar Rede”. Lembrete: isso paralisará momentaneamente a conexão com sua VNET e com seus aplicativos.

Se tudo estiver bem, você precisará se aprofundar um pouco mais:

- Há outros aplicativos usando essa rede virtual com êxito para acessar um recurso remoto? 
- Você consegue ir ao console do aplicativo e usar tcpping para alcançar recursos na sua VNET?  

Se alguma das opções acima é verdadeira, a integração de rede virtual está boa e o problema está em outro lugar. Você também pode simplesmente não ter uma resposta de dois itens acima porque não tem algo na sua rede virtual para acessar. Essa hipótese é mais complicada, já que não há uma maneira simples de ver por que você não consegue acessar o host:porta. Algumas das causas incluem:

- o host de destino está inoperante
- seu aplicativo está inoperante
- você tinha o IP ou nome de host incorreto
- seu aplicativo está escutando em uma porta diferente da que você esperava. Você pode verificar isso indo até o host e usando "netstat - aon" no prompt de comando. Isso mostrará qual ID de processo está escutando em qual porta.  
- você tem um firewall no seu host que impede o acesso à porta do aplicativo usando o intervalo de IP ponto a site
- os grupos de segurança de rede estão configurados de modo a impedir o acesso ao host do aplicativo e à porta do intervalo de IP ponto a site.

Lembre-se de que você não sabe quais IPs no intervalo de IP ponto a site seu aplicativo usará; portanto, é necessário permitir o acesso a todo o intervalo.

As etapas de depuração adicionais incluem:

- fazer logon em outra VM na rede virtual e tentar acessar o host:porta do recurso a partir daí. Existem alguns utilitários de ping do TCP que você pode usar para essa finalidade, ou até mesmo usar o telnet, se for o caso. O objetivo aqui é somente determinar se há conectividade de outra máquina virtual. 
- colocar um aplicativo em outra VM e testar o acesso àquele host e porta usando o console do aplicativo  

####Recursos locais####
Se não puder acessar os recursos locais, a primeira coisa a fazer será verificar se pode acessar um recurso na VNET. Se isso estiver funcionando, as próximas etapas são muito fáceis. De uma VM em sua rede virtual, você precisa tentar acessar o aplicativo local. Você pode usar o telnet ou um utilitário de ping de TCP. Se sua VM não puder acessar seu recurso local primeiro verifique se a conexão de VPN Site a Site está funcionando. Se ela estiver funcionando, verifique as mesmas coisas observadas anteriormente, bem como a configuração e o status do gateway local.

Agora, se sua VM hospedada na Rede Virtual pode alcançar seu sistema local, mas seu aplicativo não pode, provavelmente, isso ocorre devido a um dos seguintes motivos:
- suas rotas não estão configuradas com os intervalos de IP ponto a site em seu gateway local
- seus grupos de segurança de rede estão bloqueando o acesso para o intervalo de IP Ponto a Site
- os firewalls locais estão bloqueando o tráfego do intervalo de IP Ponto a Site
- você tem uma UDR (Rota Definida pelo Usuário) em sua Rede Virtual que impede que o tráfego baseado em Ponto a Site alcance sua rede local

## Conexões híbridas e ambientes de serviço de aplicativo##
Há três recursos que habilitam o acesso a recursos hospedados em rede virtual. Eles são:

- Integração VNET
- Conexões Híbridas
- Ambientes de Serviço de Aplicativo

As conexões híbridas exigem que você instale um agente de retransmissão chamado HCM (gerente de conexões híbridas) na sua rede. O HCM precisa ser capaz de se conectar ao Azure e também a seu aplicativo. Essa solução é especialmente boa usando uma rede remota, como sua rede local ou outra rede hospedada em nuvem, já que não exige um ponto de extremidade com acesso a Internet. O HCM só é executado no Windows e você pode ter até cinco instâncias em execução para fornecer alta disponibilidade. No entanto, as conexões híbridas só dão suporte a TCP e cada ponto de extremidade de HC tem que corresponder a uma combinação de host:porta específica.

O recurso Ambiente de Serviço de Aplicativo permite a execução de uma instância do Serviço de Aplicativo do Azure em sua rede virtual. Isso permite que seus aplicativos acessem os recursos de sua rede virtual sem etapas adicionais. Alguns dos outros benefícios de um Ambiente de Serviço de Aplicativo é que você pode usar oito trabalhadores de núcleo dedicados com 14 GB de RAM. Outra vantagem é que você pode escalonar o sistema para atender às suas necessidades. Ao contrário de ambientes multilocatários, em que o ASP tem tamanho limitado, em um ASE, você controlar quantos recursos deseja dar ao sistema. Em relação ao foco de rede deste documento, uma das coisas que você obtém com um ASE que não obtém da Integração VNET é poder funcionar com VPN Rota Expressa.

Embora haja sobreposição de caso de uso, nenhum desses recursos pode substituir o outro. Saber qual recurso usar está vinculado às suas necessidades e como você deseja usá-lo. Por exemplo:

- Se você for um desenvolvedor e quiser simplesmente executar um site no Azure que acesse o banco de dados na estação de trabalho na sua mesa, a coisa mais fácil de usar é o Conexões Híbridas.  
- Se você tem uma grande organização que deseja colocar um grande número de propriedades da Web na nuvem pública e gerenciá-las em sua própria rede, a melhor opção é o Ambiente de Serviço de Aplicativo.  
- Se você tiver uma quantidade de aplicativos hospedados no Serviço de Aplicativo e simplesmente quiser acessar recursos na VNET, a melhor opção será a Integração VNET.  

Além dos casos de uso, há alguns aspectos relativos à simplicidade. Se sua rede virtual já está conectada à sua rede local, o uso da Integração de rede virtual ou de um Ambiente de Serviço de Aplicativo é uma maneira fácil de consumir recursos locais. Por outro lado, se sua rede virtual não estiver conectada à sua rede local, é muito mais trabalhoso configurar uma VPN site a site com a VNET do que instalar o HCM.

Além das diferenças funcionais, há também diferenças de preço. O recurso Ambiente de Serviço de Aplicativo é uma oferta do serviço Premium, mas oferece a maioria das possibilidades de configuração de rede, além de outros recursos incríveis. A Integração de rede virtual pode ser usada com ASPs Standard ou Premium e é perfeita para consumir com segurança recursos em sua rede virtual do Serviço de Aplicativo multilocatário. O Conexões Híbridas atualmente depende de um conta BizTalk, com preços que variam de gratuito a mais caros baseado na quantidade necessária. Quando se trata de trabalhar em várias redes, no entanto, não há nenhum outro recurso como Conexões Híbridas, que pode permitir que você acesse recursos em mais de cem redes separadas.


<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-upgradeplan.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-existingvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-createvnet.png
[4]: ./media/web-sites-integrate-with-vnet/vnetint-howitworks.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-appmanage.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-aspmanage.png
[7]: ./media/web-sites-integrate-with-vnet/vnetint-aspmanagedetail.png
[8]: ./media/web-sites-integrate-with-vnet/vnetint-vnetp2s.png

<!--Links-->
[VNETOverview]: http://azure.microsoft.com/documentation/articles/virtual-networks-overview/
[ASPricing]: http://azure.microsoft.com/pricing/details/app-service/
[VNETPricing]: http://azure.microsoft.com/pricing/details/vpn-gateway/
[DataPricing]: http://azure.microsoft.com/pricing/details/data-transfers/

<!---HONumber=AcomDC_0302_2016-->