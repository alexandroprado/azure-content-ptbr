<properties 
	pageTitle="Escala distribuída geograficamente com ambientes de Serviço de Aplicativo" 
	description="Saiba como escalonar horizontalmente os aplicativos que usam a distribuição geográfica com o Gerenciador de Tráfego e os Ambientes de Serviço de Aplicativo." 
	services="app-service" 
	documentationCenter="" 
	authors="stefsch" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="01/09/2016" 
	ms.author="stefsch"/>

# Escala distribuída geograficamente com ambientes de Serviço de Aplicativo

## Visão geral ##
Cenários de aplicativos que exigem escala muito alta podem exceder a capacidade de recursos de computação disponíveis para uma única implantação de um aplicativo. Aplicativos de votação, eventos esportivos e de entretenimento televisionados são exemplos de cenários que exigem escala extremamente alta. Os requisitos de alta escala podem ser atendidos pela escalação horizontal de aplicativos, com várias implantações de aplicativo feitas dentro de uma única região, bem como entre regiões, para lidar com requisitos de carga extrema.

Ambientes de Serviço de Aplicativo são uma plataforma ideal para escalar horizontalmente. Uma vez que uma configuração de Ambiente de Serviço de Aplicativo tiver selecionada possa dar suporte a uma taxa de solicitação conhecida, os desenvolvedores poderão implantar Ambientes de Serviço de Aplicativo adicionais em massa para obter uma capacidade de carga de pico desejada.

Por exemplo, suponha que um aplicativo em execução em uma configuração de Ambiente de Serviço de Aplicativo foi testado para manipular solicitações de 20K por segundo (RPS). Se a capacidade de carga de pico desejada é de 100 mil RPS, cinco (5) Ambientes de Serviço de Aplicativo podem ser criados e configurados para garantir que o aplicativo possa lidar com a carga máxima prevista.

Já que os clientes normalmente acessam aplicativos usando um domínio personalizado, os desenvolvedores precisam ter uma maneira de distribuir solicitações de aplicativo entre todas as instâncias do Ambiente de Serviço de Aplicativo. Uma ótima maneira de fazer isso é resolver o domínio personalizado usando um [perfil do Gerenciador de Tráfego do Azure][AzureTrafficManagerProfile]. O perfil do Gerenciador de Tráfego pode ser configurado para apontar para todos os Ambientes de Serviço de Aplicativo individuais. O Gerenciador de Tráfego manipulará automaticamente a distribuição de clientes em todos os Ambientes de Serviço de Aplicativo baseado nas configurações de balanceamento de carga no perfil do Gerenciador de Tráfego. Essa abordagem funciona independentemente de todos os Ambientes de Serviço de Aplicativo serem implantados em uma única região do Azure ou distribuídos pelo mundo todo em várias regiões do Azure.

Além disso, uma vez que os clientes acessam aplicativos usando o domínio personalizado, eles não estão cientes do número de Ambientes de Serviço de Aplicativo que estão executando um aplicativo. Sendo assim, os desenvolvedores podem adicionar e remover Ambientes de Serviço de Aplicativo rápida e facilmente baseados na carga de tráfego observada.

O diagrama conceitual abaixo mostra um aplicativo escalado horizontalmente em três Ambientes de Serviço de Aplicativo dentro de uma única região.

![Arquitetura conceitual][ConceptualArchitecture]

O restante deste tópico explica as etapas envolvidas na configuração de uma topologia distribuída para o aplicativo de exemplo usando vários Ambientes de Serviço de Aplicativo.

## Planejando a topologia ##
Antes de criar uma superfície de aplicativo distribuído, é bom ter algumas informações prévias.

- **Domínio personalizado para o aplicativo:** qual é o nome de domínio personalizado que os clientes usarão para acessar o aplicativo? Para o aplicativo de exemplo, o nome de domínio personalizado é *www.scalableasedemo.com*
- **Domínio do Gerenciador de Tráfego:** um nome de domínio deve ser escolhido ao criar um [perfil do Gerenciador de Tráfego do Azure][AzureTrafficManagerProfile]. Esse nome será combinado com o sufixo *trafficmanager.net* para registrar uma entrada de domínio gerenciada pelo Gerenciador de Tráfego. Para o aplicativo de exemplo, o nome escolhido é *scalable-ase-demo*. Assim, o nome de domínio completo gerenciado pelo Gerenciador de Tráfego é *scalable-ase-demo.trafficmanager.net*.
- **Estratégia para escalonar a superfície do aplicativo:** a superfície do aplicativo será distribuída em vários Ambientes de Serviço de Aplicativo em uma única região? Várias regiões? Uma combinação de ambas as abordagens? A decisão deve se basear nas expectativas da origem do tráfego do cliente e em como o resto da infraestrutura de back-end de suporte de um aplicativo pode ser escalonado. Por exemplo, com um aplicativo 100% sem monitoração de estado, ele pode ser altamente dimensionado usando uma combinação de vários Ambientes de Serviço de Aplicativo por região do Azure, multiplicado por Ambientes de Serviço de Aplicativo implantado em várias regiões do Azure. Com mais de 15 regiões públicas do Azure disponíveis para escolha, os clientes podem realmente criar uma superfície de aplicativo de hiperescala mundial. Para o aplicativo de exemplo usado neste artigo, três Ambientes de Serviço de Aplicativo foram criados em uma única região do Azure (centro-sul dos EUA).
- **Convenção de nomenclatura para os Ambientes de Serviço de Aplicativo:** cada Ambiente de Serviço de Aplicativo requer um nome exclusivo. Além de um ou dois Ambientes de Serviço de Aplicativo, é útil ter uma convenção de nomenclatura para ajudar a identificar cada Ambiente de Aplicativo de Serviço. Para o aplicativo de exemplo, foi usada uma convenção de nomenclatura simples. Os nomes dos três Ambientes de Serviço de Aplicativo são *fe1ase*, *fe2ase* e *fe3ase*.
- **Convenção de nomenclatura para os aplicativos:** como várias instâncias do aplicativo serão implantadas, é necessário um nome para cada instância do aplicativo implantado. Um recurso pouco conhecido, mas muito conveniente dos Ambientes de Serviço de Aplicativo é que o mesmo nome de aplicativo pode ser usado em vários Ambientes de Serviço de Aplicativo. Como cada Ambiente de Serviço de Aplicativo tem um sufixo de domínio exclusivo, os desenvolvedores podem optar por usar novamente o mesmo nome de aplicativo em cada ambiente. Por exemplo um desenvolvedor poderia ter aplicativos nomeados da seguinte forma: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, etc. No entanto, no caso do aplicativo de exemplo, cada instância do aplicativo também tem um nome exclusivo. Os nomes de instância de aplicativo usados são *webfrontend1*, *webfrontend2* e *webfrontend3*.


## Configurando o perfil do Gerenciador de Tráfego ##
Quando várias instâncias de um aplicativo são implantadas em vários Ambientes de Serviço de Aplicativo, as instâncias de aplicativos individuais podem ser registradas com o Gerenciador de Tráfego. Para o aplicativo de exemplo, um perfil do Gerenciador de Tráfego é necessário para *scalable-ase-demo.trafficmanager.net* que seja capaz de direcionar os clientes para uma das seguintes instâncias de aplicativo implantadas:

- **webfrontend1.fe1ase.p.azurewebsites.net:** uma instância do aplicativo de exemplo implantada no primeiro Ambiente de Serviço de Aplicativo.
- **webfrontend2.fe2ase.p.azurewebsites.net:** uma instância do aplicativo de exemplo implantada no segundo Ambiente de Serviço de Aplicativo.
- **webfrontend3.fe3ase.p.azurewebsites.net:** uma instância do aplicativo de exemplo implantada no terceiro Ambiente de Serviço de Aplicativo.

A maneira mais fácil de registrar vários pontos de extremidade do Serviço de Aplicativo do Azure, todos executados na **mesma** região do Azure, é com a visualização do [suporte de Gerenciador de Recursos do Azure (ARM) do Gerenciador de Tráfego][ARMTrafficManager] do PowerShell.

A primeira etapa é a criação de um perfil do Gerenciador de Tráfego do Azure. O código abaixo mostra como o perfil foi criado para o aplicativo de exemplo:

    $profile = New-AzureTrafficManagerProfile –Name scalableasedemo -ResourceGroupName yourRGNameHere -TrafficRoutingMethod Weighted -RelativeDnsName scalable-ase-demo -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"

Observe como o parâmetro *RelativeDnsName* foi definido como *scalable-ase-demo*. É assim que o nome de domínio *scalable-ase-demo.trafficmanager.net* é criado e associado a um perfil do Gerenciador de Tráfego.

O parâmetro *TrafficRoutingMethod* define a política de balanceamento de carga que o Gerenciador de Tráfego usará para determinar como distribuir a carga cliente entre todos os pontos de extremidade disponíveis. Neste exemplo, o método *Weighted* foi o escolhido. Isso fará com que as solicitações de cliente sejam distribuídas entre todos os pontos de extremidade do aplicativo registrado baseadas nos pesos relativos associados a cada ponto de extremidade.

Com o perfil criado, cada instância do aplicativo é adicionada ao perfil como um *ponto de extremidade externo*. O código a seguir mostra as URLs para cada uma das três instâncias do aplicativo que estão sendo adicionadas ao perfil.

    Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend1 –TrafficManagerProfile $profile –Type ExternalEndpoints –Target webfrontend1.fe1ase.p.azurewebsites.net –EndpointStatus Enabled –Weight 10
    Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend2 –TrafficManagerProfile $profile –Type ExternalEndpoints –Target webfrontend2.fe2ase.p.azurewebsites.net –EndpointStatus Enabled –Weight 10
    Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend3 –TrafficManagerProfile $profile –Type ExternalEndpoints –Target webfrontend3.fe3ase.p.azurewebsites.net –EndpointStatus Enabled –Weight 10
    
    Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile

Observe como não há uma chamada para *Add-AzureTrafficManagerEndpointConfig* em cada instância do aplicativo individual. O parâmetro *Target* em cada comando Powershell aponta para o FQDN (nome de domínio totalmente qualificado) de cada uma das três instâncias do aplicativo implantadas. Os FQDNs diferentes são os valores que serão usados para percorrer a cadeia de CNAME DNS *scalable-ase-demo.trafficmanager.net* e distribuir a carga de tráfego em todos os pontos de extremidade registrados no perfil do Gerenciador de Tráfego.

Todos os três pontos de extremidade usam o mesmo valor (10) para o parâmetro *Weight*. Isso faz com que o Gerenciador de Tráfego distribua as solicitações de cliente entre todas as três instâncias do aplicativo de forma quase igual.

*Observação:* já que o suporte ÃRM do Gerenciador de Tráfego está atualmente no modo visualização, os pontos de extremidade do Serviço de Aplicativo do Azure devem definir o parâmetro *Type* como *ExternalEndpoints*. No futuro, os pontos de extremidade do Serviço de Aplicativo do Azure terão suporte nativo como um tipo de ponto de extremidade pela variante ARM do Gerenciador de Tráfego.

## Apontando o domínio personalizado do aplicativo no domínio do Gerenciador de Tráfego ##
A última etapa necessária é apontar o domínio personalizado do aplicativo para o domínio do Gerenciador de Tráfego. Para o aplicativo de exemplo, isso significa apontar *www.scalableasedemo.com* para *scalable-ase-demo.trafficmanager.net*. Essa etapa deve ser concluída no registrador de domínio que gerencia o domínio personalizado.

Usando as ferramentas de gerenciamento de seu registrador domínio, um registro CNAME precisa ser criado apontando para o domínio personalizado no domínio do Gerenciador de Tráfego. A figura a seguir mostra um exemplo da aparência dessa configuração CNAME:

![CNAME para um domínio personalizado][CNAMEforCustomDomain]

Embora não seja abordado neste tópico, lembre-se de que cada instância de aplicativo individual precisa ter também o domínio personalizado registrado nele. Caso contrário, se uma solicitação chegar a uma instância do aplicativo e o aplicativo não tiver o domínio personalizado registrado nele, a solicitação falhará.

Neste exemplo, o domínio personalizado é *www.scalableasedemo.com* e cada instância do aplicativo tem o domínio personalizado associado a ele.

![Domínio personalizado][CustomDomain]

Para uma recapitulação do registro de um domínio personalizado nos aplicativos de Serviço de aplicativo do Azure, consulte o seguinte artigo sobre [registrar domínios personalizados][RegisterCustomDomain].

## Experimentando a topologia distribuída ##
O resultado final da configuração do Gerenciador de Tráfego e do DNS é que as solicitações de *www.scalableasedemo.com* fluirão através da seguinte sequência:

1. Um navegador ou dispositivo fará uma pesquisa de DNS para *www.scalableasedemo.com*
2. A entrada CNAME no registrador de domínio faz com que a pesquisa de DNS seja redirecionada para o Gerenciador de Tráfego do Azure.
3. Uma pesquisa de DNS é feita para *scalable-ase-demo.trafficmanager.net* em um dos servidores DNS do Gerenciador de Tráfego do Azure.
4. Baseado na política de balanceamento de carga (o parâmetro *TrafficRoutingMethod* usado anteriormente durante a criação do perfil do Gerenciador de Tráfego), o Gerenciador de Tráfego irá selecionar um dos pontos de extremidade configurados e retornar o FQDN do ponto de extremidade para o navegador ou o dispositivo.
5.  Já que o FQDN do ponto de extremidade é a URL de uma instância do aplicativo em execução em um Ambiente de Serviço de Aplicativo, o navegador ou dispositivo solicitará que um servidor DNS da Microsoft Azure resolva o FQDN para um endereço IP. 
6. O navegador ou dispositivo enviará a solicitação HTTP/S para o endereço IP.  
7. A solicitação chegará em uma das instâncias do aplicativo em execução em um dos Ambientes de Serviço do Aplicativo.

A imagem do console abaixo mostra uma pesquisa de DNS de domínio personalizado do aplicativo de exemplo resolver com êxito uma instância do aplicativo em execução em dos três Ambientes de Serviço de Aplicativo de exemplo (neste caso, o segundo dos três Ambientes de Serviço de Aplicativo):

![Pesquisa de DNS][DNSLookup]

## Informações e links adicionais ##
Documentação sobre a visualização do [suporte de Gerenciador de Recursos do Azure (ARM) do Gerenciador de Tráfego][ARMTrafficManager] do PowerShell.

[AZURE.INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[AzureTrafficManagerProfile]: https://azure.microsoft.com/documentation/articles/traffic-manager-manage-profiles/
[ARMTrafficManager]: https://azure.microsoft.com/documentation/articles/traffic-manager-powershell-arm/
[RegisterCustomDomain]: https://azure.microsoft.com/pt-BR/documentation/articles/web-sites-custom-domain-name/


<!-- IMAGES -->
[ConceptualArchitecture]: ./media/app-service-app-service-environment-geo-distributed-scale/ConceptualArchitecture-1.png
[CNAMEforCustomDomain]: ./media/app-service-app-service-environment-geo-distributed-scale/CNAMECustomDomain-1.png
[DNSLookup]: ./media/app-service-app-service-environment-geo-distributed-scale/DNSLookup-1.png
[CustomDomain]: ./media/app-service-app-service-environment-geo-distributed-scale/CustomDomain-1.png

<!---HONumber=AcomDC_0128_2016-->