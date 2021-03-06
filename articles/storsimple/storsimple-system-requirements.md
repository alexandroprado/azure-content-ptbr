<properties 
   pageTitle="Requisitos do sistema do StorSimple | Microsoft Azure" 
   description="Descreve os requisitos e as práticas recomendadas para software, alta disponibilidade e rede para uma solução do Microsoft Azure StorSimple." 
   services="storsimple" 
   documentationCenter="NA" 
   authors="alkohli" 
   manager="carmonm" 
   editor=""/>

<tags
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD" 
   ms.date="02/04/2016"
   ms.author="alkohli"/>

# Software StorSimple, alta disponibilidade e requisitos de rede

## Visão geral

Bem-vindo ao Microsoft Azure StorSimple. Este artigo descreve os requisitos do sistema importantes e as práticas recomendadas para seu dispositivo StorSimple e para os clientes de armazenamento acessarem o dispositivo. Recomendamos que você examine as informações com atenção antes de implantar o sistema StorSimple e consulte-as, quando necessário, durante a implantação e operação subsequente.

Os requisitos do sistema incluem:

- **Requisitos de software para clientes de armazenamento** - descreve os sistemas operacionais com suporte e quaisquer requisitos adicionais para esses sistemas operacionais.
- **Requisitos de rede para o dispositivo StorSimple** - fornece informações sobre as portas que precisam ser abertas no firewall para permitir o tráfego iSCSI, de nuvem ou de gerenciamento.
- **Requisitos de alta disponibilidade para o StorSimple** - descreve os requisitos de alta disponibilidade e as práticas recomendadas para o dispositivo StorSimple e o computador host. 


## Requisitos de software para clientes de armazenamento 

Os requisitos de software a seguir são para os clientes de armazenamento que acessam seu dispositivo StorSimple.

| Sistemas operacionais com suporte | Versão necessária | Requisitos/observações adicionais |
| --------------------------- | ---------------- | ------------- |
| Windows Server | 2008R2 SP1, 2012, 2012R2 |Os volumes iSCSI do StorSimple são permitidos para o uso somente nos seguintes tipos de disco do Windows:<ul><li>Volume simples no disco básico</li><li>Volume simples e espelhado no disco dinâmico</li></ul>O provisionamento dinâmico do Windows Server 2012 e dos recursos ODX serão permitidos se você estiver usando um volume iSCSI do StorSimple.<br><br>O StorSimple pode criar volumes dinâmica ou totalmente provisionados. Não é possível criar volumes parcialmente provisionados.<br><br>Reformatar um volume de provisionamento dinâmico pode levar muito tempo. É recomendável excluir o volume e criar um novo em vez de reformatar. No entanto, se você ainda preferir reformatar um volume:<ul><li>execute o comando a seguir antes de reformatar para evitar atrasos de reclamação de espaço: <br>`fsutil behavior set disabledeletenotify 1`</br></li><li>depois da conclusão da formatação, use o seguinte comando para reativar a reclamação de espaço:<br>`fsutil behavior set disabledeletenotify 0`</br></li><li>aplique o hotfix do Windows Server 2012, conforme descrito em [KB 2878635](https://support.microsoft.com/kb/2870270) em seu computador Windows Server.</li></ul></li></ul></ul> Se você estiver configurando o Gerenciador de Instantâneos do StorSimple ou o Adaptador do StorSimple para SharePoint, vá para [Requisitos de software para os componentes opcionais](#software-requirements-for-optional-components).|
| VMWare ESX | 5\.1 e 5.5 | Compatível com o VMware vSphere como cliente iSCSI. O recurso de bloco VAAI é compatível com o VMware vSphere nos dispositivos StorSimple. 
| Linux RHEL/CentOS | 5 e 6 | Suporte para os clientes Linux iSCSI com o iniciador open-iSCSI versões 5 e 6. |
| Linux | SUSE Linux 11 | |
 > [AZURE.NOTE] O IBM AIX não é suportado atualmente com o StorSimple.

## Requisitos de software para os componentes opcionais

Os seguintes requisitos de software são para os componentes opcionais do StorSimple (Gerenciador de Instantâneos do StorSimple e Adaptador do StorSimple para SharePoint).

| Componente | Plataforma de host | Requisitos/observações adicionais |
| --------------------------- | ---------------- | ------------- |
| Gerenciador de instantâneos do StorSimple | Windows Server 2008R2 SP1, 2012, 2012R2 | O uso do Gerenciador de Instantâneos do StorSimple no Windows Server é necessário para fazer backup/restauração dos discos dinâmicos espelhados e quaisquer backups consistentes com o aplicativo.<br> O Gerenciador de Instantâneos do StorSimple é suportado somente no Windows Server 2008 R2 SP1 (64 bits), Windows 2012 R2 e Windows Server 2012.<ul><li>Se você estiver usando o Windows Server 2012, instale o .NET 3.5 – 4.5 antes de instalar o Gerenciador de Instantâneos do StorSimple.</li><li>Se você estiver usando o Windows Server 2008 R2 SP1, deverá instalar o Windows Management Framework 3.0 antes de instalar o Gerenciador de Instantâneos do StorSimple.</li></ul> |
| Adaptador do StorSimple para SharePoint | Windows Server 2008R2 SP1, 2012, 2012R2 |<ul><li>O Adaptador do StorSimple para SharePoint só tem suporte no SharePoint 2010 e no SharePoint 2013.</li><li>O RBS requer o SQL Server Enterprise Edition, versão 2008 R2 ou 2012.</li></ul>|
 
## Requisitos de rede para seu dispositivo StorSimple

Seu dispositivo StorSimple é um dispositivo bloqueado. No entanto, é preciso abrir portas no firewall para permitir o tráfego de gerenciamento, de nuvem ou iSCSI. A tabela a seguir lista as portas que precisam estar abertas no firewall. Nesta tabela, *entrada* ou *de entrada* refere-se à direção a partir da qual as solicitações de cliente acessam o dispositivo. *Saída* ou *de saída* refere-se à direção na qual seu dispositivo StorSimple envia dados externamente, além da implantação: por exemplo, saída para a Internet.

| Nº da porta<sup>1,2</sup> | Entrada ou saída | Escopo da porta | Obrigatório | Observações |
|------------------------|-----------|------------|----------|-------| 
|TCP 80 (HTTP)<sup>3</sup>| Saída | WAN | Não |<ul><li>A porta de saída é usada para acesso à Internet a fim de obter as atualizações.</li><li>O proxy da Web de saída pode ser configurado pelo usuário.</li><li>Para permitir atualizações do sistema, esta porta também deve estar aberta para os IPs fixos do controlador.</li></ul> |
|TCP 443 (HTTPS)<sup>3</sup>| Saída | WAN | Sim |<ul><li>A porta de saída é usada para acesso aos dados na nuvem.</li><li>O proxy da Web de saída pode ser configurado pelo usuário.</li><li>Para permitir atualizações do sistema, esta porta também deve estar aberta para os IPs fixos do controlador.</li></ul>|
|UDP 53 (DNS) | Saída | WAN | Em alguns casos; consulte as observações. |Esta porta só será necessária se você estiver usando um servidor DNS baseado na Internet. |
| UDP 123 (NTP) | Saída | WAN | Em alguns casos; consulte as observações. |Esta porta é necessária apenas se você estiver usando um servidor NTP baseado na Internet. |
| TCP 9354 | Saída | WAN | Sim |A porta de saída é usada pelo dispositivo StorSimple para se comunicar com o serviço StorSimple Manager. |
| 3260 (iSCSI) | No | LAN | Não | Esta porta é usada para acessar dados em iSCSI.|
| 5985 | No | LAN | Não | A porta de entrada é usada pelo StorSimple Snapshot Manager para se comunicar com o dispositivo do StorSimple.<br>Essa porta também é usada quando você se conecta remotamente ao Windows PowerShell para o StorSimple via HTTP. |
| 5986 | No | LAN | Não | Esta porta é usada quando você se conecta remotamente ao Windows PowerShell para StorSimple via HTTPS. |

<sup>1</sup> Nenhuma porta de entrada precisa estar aberta na Internet pública.

<sup>2</sup> Se várias portas tiverem uma configuração de gateway, a ordem do tráfego de saída roteado será determinada com base na ordem de roteamento da porta descrita em [Roteamento de porta](#routing-metric) abaixo.

<sup>3</sup> Os IPs fixos do controlador em seu dispositivo StorSimple devem ser roteáveis e conseguirem se conectar à Internet. Os endereços IP fixos são usados para fornecer as atualizações ao dispositivo. Se os controladores de dispositivo não puderem se conectar à Internet através de IPs fixa, não será possível atualizar o dispositivo StorSimple.

> [AZURE.IMPORTANT] Verifique se o firewall não modifica nem descriptografa nenhum tráfego SSL entre o dispositivo StorSimple e o Azure.

### Métrica de roteamento

Uma métrica de roteamento é associada às interfaces e ao gateway que encaminha os dados para as redes especificadas. A métrica de roteamento é usada pelo protocolo de roteamento para calcular o melhor caminho para um determinado destino, se ela detecta que existem vários caminhos para o mesmo destino. Quanto menor a métrica de roteamento, maior será a preferência.

No contexto do StorSimple, se vários gateways e interfaces de rede estiverem configurados para encaminhar o tráfego, a métrica de roteamento entrará em ação para determinar a ordem relativa em que as interfaces serão usadas. A métrica de roteamento não pode ser alterada pelo usuário. No entanto, você pode usar o cmdlet `Get-HcsRoutingTable` para imprimir a tabela de roteamento (e as métricas) em seu dispositivo do StorSimple. Mais informações sobre o cmdlet Get-HcsRoutingTable em [Solução de problemas com a implantação do StorSimple](storsimple-troubleshoot-deployment.md).

Os algoritmos de métrica de roteamento são diferentes, dependendo da versão de software em execução no dispositivo do StorSimple.

**Versões anteriores à Atualização 1**

Isso inclui versões de software anteriores à Atualização 1 como a versão GA, 0.1, 0.2 ou 0.3. A ordem com base nas métricas de roteamentos é a seguinte:

   *Interface de rede de 10 GbE configurada por último > Outras interfaces de rede de 10 GbE > Última interface de rede de 1 GbE configurada > Outra interface de rede de 1 GbE*


**Versões a partir da Atualização 1 e anteriores à Atualização 2**

Isso inclui versões de software, como 1, 1.1 e 1.2. A ordem com base nas métricas de roteamentos é decidida da seguinte maneira:

   *DADOS 0 > Interface de rede de 10 GbE configurada por último > Outras interfaces de rede de 10 GbE > Última interface de rede de 1 GbE configurada > Outra interface de rede de 1 GbE*

   Na Atualização 1, a métrica de roteamento de DADOS 0 é a mais baixa; portanto, todo o tráfego de nuvem é roteado por meio de DADOS 0. Anote isso se houver mais de uma interface de rede habilitada para nuvem em seu dispositivo StorSimple.


**Versões a partir da Atualização 2**

A Atualização 2 contém vários aprimoramentos relacionados à rede; além disso, a métrica de roteamento foi alterada. O comportamento pode ser explicado da seguinte maneira.

- Um conjunto de valores predeterminados foi atribuído a interfaces de rede. 	
		
- Considere uma tabela de exemplo mostrada abaixo, com valores atribuídos às várias interfaces de rede quando são habilitadas ou desabilitadas para a nuvem, mas com um gateway configurado. Observe que os valores atribuídos aqui são apenas exemplos.

		
	| Interface de rede | Habilitado para nuvem | Desabilitado para a nuvem com o gateway |
	|-----|---------------|---------------------------|
	| Data 0 | 1 | - |
	| Data 1 | 2 | 20 |
	| Data 2 | 3 | 30 |
	| Data 3 | 4 | 40 |
	| Data 4 | 5 | 50 |
	| Data 5 | 6 | 60 |


- A ordem na qual o tráfego da nuvem será roteado pelas interfaces de rede é:
	 
	*Data 0 > Data 1 > Data 2 > Data 3 > Data 4 > Data 5*

	Isso pode ser explicado pelo exemplo a seguir.

	Considere um dispositivo do StorSimple com duas interfaces de rede habilitadas para a nuvem, Data 0 e Data 5. Data 1 a 4 são desabilitados para a nuvem, mas têm um gateway configurado. A ordem na qual o tráfego será roteado para este dispositivo será:

	*Data 0 (1) > Data 5 (6) > Data 1 (20) > Data 2 (30) > Data 3 (40) > Data 4 (50)*
	
	*em que os números entre parênteses indicam as respectivas métricas de roteamento.*
	
	Se Data 0 falhar, o tráfego de nuvem será roteado por meio de Data 5. Considerando que um gateway é configurado em todas as outras redes, se Data 0 e Data 5 falharem, o tráfego de nuvem passará por Data 1.
 

- Se uma interface de rede habilitada para a nuvem falhar, serão três tentativas com um atraso de 30 segundos para se conectar à interface. Se todas as tentativas falharem, o tráfego será roteado para a próxima interface habilitada para a nuvem disponível, conforme determinado pela tabela de roteamento. Se todas as interfaces de rede habilitadas para a nuvem falharem, o dispositivo falhará no outro controlador (sem reinicialização, neste caso).
	
- Se houver uma falha de VIP para uma interface de rede habilitada para iSCSI, haverá três tentativas com um atraso de 2 segundos. Esse comportamento permanece o mesmo em relação às versões anteriores. Se todas as interfaces de rede iSCSI falharem, ocorrerá um failover de controlador (acompanhado por uma reinicialização).


- Um alerta também será gerado no dispositivo do StorSimple quando houver uma falha de VIP. Para saber mais, acesse a [referência rápida de alerta](storsimple-manage-alerts.md).
	
- Quanto às repetições, o iSCSI terá precedência sobre a nuvem.

	Considere o seguinte exemplo: um dispositivo do StorSimple tem duas interfaces de rede habilitadas, Data 0 e Data 1. Data 0 é habilitado para a nuvem, enquanto Data 1 é habilitado para a nuvem e para iSCSI. Nenhuma outra interface de rede neste dispositivo é habilitada para a nuvem ou para iSCSI.
		
	Se Data 1 falhar, por ser a última interface de rede do iSCSI, isso resultará em um failover do controlador para Data 1 no outro controlador.


### Práticas recomendadas de rede

Além dos requisitos de rede acima, para obter o desempenho ideal de sua solução StorSimple, atenda às seguintes práticas recomendadas:

- Verifique se o dispositivo StorSimple tem uma largura de banda dedicada de 40 Mbps (ou mais) disponível sempre. Essa largura de banda não deve ser compartilhada com outros aplicativos.

- Verifique a conectividade de rede com a Internet está disponível sempre. Conexões de Internet esporádicas ou não confiáveis com os dispositivos, sem incluir qualquer conectividade com a Internet, resultarão em uma configuração sem suporte.

- Isole o tráfego iSCSI e o tráfego de nuvem com interfaces de rede dedicadas em seu dispositivo para acesso iSCSI e à nuvem. Para obter mais informações, veja como [modificar as interfaces de rede](storsimple-modify-device-config.md#modify-network-interfaces) em seu dispositivo StorSimple.

- Não use uma configuração do LACP (Protocolo de Agregação de Link) para as suas interfaces de rede. Essa é uma configuração sem suporte.


## Requisitos de alta disponibilidade para o StorSimple

A plataforma de hardware incluída com a solução StorSimple possui os recursos de disponibilidade e confiabilidade que fornecem uma base para uma infraestrutura de armazenamento altamente disponível e tolerante a falhas em seu datacenter. No entanto, há requisitos e práticas recomendadas que você deve seguir para ajudar a garantir a disponibilidade de sua solução do StorSimple. Antes de implantar o StorSimple, examine cuidadosamente os requisitos e práticas recomendadas a seguir para o dispositivo do StorSimple e os computadores host conectados.

Para obter mais informações sobre como monitorar e manter os componentes de hardware do seu dispositivo do StorSimple, vá para [Usar o serviço do StorSimple Manager para monitorar componentes de hardware e status](storsimple-monitor-hardware-status.md) e [Substituição dos componentes de hardware do StorSimple](storsimple-hardware-component-replacement.md).

### Requisitos de alta disponibilidade e procedimentos para seu dispositivo StorSimple

Examine as seguintes informações com atenção para garantir a alta disponibilidade do seu dispositivo StorSimple.

#### PCMs

Os dispositivos StorSimple incluem módulos redundantes, intercambiáveis e de refrigeração (PCMs). Cada PCM tem capacidade suficiente para atender ao chassi inteiro. Para garantir a alta disponibilidade, os dois PCMs devem estar instalados.

- Conecte seus PCMs a fontes de alimentação diferentes a fim de fornecer disponibilidade de uma fonte de alimentação falhar.
- Se um PCM falhar, solicite uma substituição imediatamente.
- Remova um PCM com falha somente quando tiver a reposição e estiver pronto para instalá-lo.
- Não remova os dois PCMs simultaneamente. O módulo PCM inclui o módulo de bateria de backup. Remover os dois PCMs resultará em um desligamento sem proteção da bateria e o estado do dispositivo não será salvo. Para obter mais informações sobre a bateria, vá para [Manutenção do módulo de bateria de backup](storsimple-battery-replacement.md#maintain-the-backup-battery-module).

#### Módulos do controlador

Os dispositivos StorSimple incluem módulos de controlador redundantes e intercambiáveis. Os módulos do controlador operam de modo ativo/passivo. A qualquer momento, um módulo do controlador está ativo e fornecendo o serviço, enquanto o outro módulo do controlador está passivo. O módulo do controlador passivo está ligado e se tornará operacional se o módulo do controlador ativo falhar ou for removido. Cada módulo do controlador tem capacidade suficiente para atender ao chassi inteiro. Os dois módulos do controlador devem ser instalados para garantir a alta disponibilidade.

- Certifique-se de que os dois módulos do controlador estejam sempre instalados.

- Se um módulo do controlador falhar, solicite uma substituição imediatamente.

- Remova um módulo de controlador com falha somente quando tiver a reposição e estiver pronto para instalá-lo. A remoção de um módulo para períodos prolongados afetará o fluxo de ar e, portanto, o resfriamento do sistema.

- Certifique-se de que as conexões de rede para ambos os módulos de controlador sejam idênticas, e as interfaces da rede conectada tenham uma configuração de rede idêntica.

- Se um módulo de controlador falhar ou precisar de substituição, certifique-se de que o outro módulo de controlador esteja em um estado ativo antes de substituir o módulo de controlador com falha. Para verificar se um controlador está ativo, vá para [Identificar o controlador ativo em seu dispositivo](storsimple-controller-replacement.md#identify-the-active-controller-on-your-device).

- Não remova os dois módulos de controlador ao mesmo tempo. Se estiver ocorrendo um failover do controlador, não desligue o módulo do controlador em espera ou o remova do chassi.

- Após o failover do controlador, aguarde pelo menos cinco minutos antes de remover um dos módulos de controlador.

#### Interfaces de rede

Cada módulo de controlador do dispositivo StorSimple tem quatro interfaces de rede Ethernet de 1 Gigabit e duas de 10 Gigabits.

- Certifique-se de que as conexões de rede para os dois módulos de controlador sejam idênticas, e que as interfaces de rede as quais as interfaces do módulo de controlador estão conectadas tenham uma configuração de rede idêntica.

- Quando possível, implante as conexões de rede entre opções diferentes para garantir a disponibilidade do serviço no caso de falhas do dispositivo de rede.

- Ao desconectar a única, ou a última, interface habilitada para iSCSI (com IPs atribuídos), desabilite primeiro a interface e, em seguida, desconecte os cabos. Se a interface for desconectada primeiro, o controlador ativo passará por failover para o controlador passivo. Se as interfaces correspondentes do controlador passivo também forem desconectadas, os dois controladores serão reiniciados várias vezes antes da definição de um controlador.

- Conecte pelo menos duas interfaces DATA com a rede de cada controlador de módulo.

- Se você habilitou as duas interfaces de 10 GbE, implante-as em opções diferentes.

- Quando possível, use o MPIO em servidores para garantir que os servidores possam tolerar um link, rede ou falha de interface.

Para saber mais sobre como colocar o dispositivo em rede para proporcionar alta disponibilidade e desempenho, acesse [Instalar o dispositivo StorSimple 8100](storsimple-8100-hardware-installation.md#cable-your-storsimple-8100-device) ou [Instalar o dispositivo StorSimple 8600](storsimple-8600-hardware-installation.md#cable-your-storsimple-8600-device).

#### SSDs e HDDs

Dispositivos StorSimple incluem discos de estado sólido (SSDs) e unidades de disco rígido (HDDs) protegidos com espaços espelhados. O uso de espaços espelhados garante que o dispositivo seja capaz de tolerar a falha de um ou mais SSDs ou HDDs.

- Certifique-se de que todos os módulos SSD e HDD estejam instalados.

- Se um SSD ou HDD falhar, solicite uma substituição imediatamente.

- Se um SSD ou HDD falhar ou exigir substituição, remova somente o SSD ou HDD que exige a substituição.

- Não remova mais de um SSD ou HDD do sistema a qualquer momento. Uma falha de dois ou mais discos de determinado tipo (HDD, SSD) ou uma falha consecutiva em um curto período pode resultar em mau funcionamento do sistema e possível perda de dados. Se isso ocorrer, [entre em contato com o Suporte da Microsoft](storsimple-contact-microsoft-support.md) para obter assistência.

- Durante a substituição, monitore o **Status de Hardware** na página **Manutenção** das unidades no SSDs e HDDs. Um status de marca de verificação verde indica que os discos estão íntegros ou OK, enquanto que um ponto de exclamação vermelho indica um SSD ou HDD com falha.

- Recomendamos a configuração de instantâneos em nuvem para todos os volumes que você precisa proteger em caso de falha do sistema.

#### Compartimento EBOD

O modelo do dispositivo StorSimple 8600 inclui um compartimento EBOD (Extended Bunch of Disks) além do compartimento primário. Um EBOD contém controladores EBOD e HDDs (unidades de disco rígido) protegidos por espaços espelhados. O uso de espaços espelhados garante que o dispositivo seja capaz de tolerar a falha de um ou mais HDDs. O compartimento EBOD está conectado ao compartimento primário por meio de cabos SAS redundantes.

- Verifique se ambos os módulos do controlador de compartimento EBOD, os cabos SAS e todos os discos rígidos estão sempre instalados.

- Se um módulo do controlador de compartimento EBOD falhar, solicite uma substituição imediatamente.

- Se um módulo do controlador de compartimento EBOD falhar, certifique-se de que o outro módulo do controlador esteja ativo antes de substituir o módulo com falha. Para verificar se um controlador está ativo, vá para [Identificar o controlador ativo em seu dispositivo](storsimple-controller-replacement.md#identify-the-active-controller-on-your-device).

- Durante uma substituição do módulo do controlador EBOD, monitore continuamente o status do componente no serviço do StorSimple Manager acessando o status **Manutenção** - **Hardware**.

- Se um cabo SAS falha ou exigir substituição (o Suporte da Microsoft deve participar dessa decisão), remova apenas o cabo SAS que exige a substituição.

- Não remova simultaneamente os dois cabos SAS do sistema em nenhum momento.

### Recomendações de alta disponibilidade para os computadores de host

Leia com atenção essas práticas recomendadas para garantir a alta disponibilidade dos hosts conectados ao dispositivo StorSimple.

- Defina o StorSimple com as [configurações de cluster de servidores de arquivos com dois nós][1]. Ao remover os pontos individuais de falha e criando redundância no lado do host, a solução inteira se torna altamente disponível.

- Use compartilhamentos CA (disponíveis continuamente) disponíveis no Windows Server 2012 (SMB 3.0) para alta disponibilidade durante o failover dos controladores de armazenamento. Para obter informações adicionais sobre a configuração de clusters de servidor de arquivos e compartilhamentos Disponíveis Continuamente com o Windows Server 2012, consulte esta [demonstração em vídeo](http://channel9.msdn.com/Events/IT-Camps/IT-Camps-On-Demand-Windows-Server-2012/DEMO-Continuously-Available-File-Shares).

## Próximas etapas

- [Saiba mais sobre os limites do sistema StorSimple](storsimple-limits.md).
- [Saiba como implantar sua solução StorSimple](storsimple-deployment-walkthrough-u2.md).
 
<!--Reference links-->
[1]: https://technet.microsoft.com/library/cc731844(v=WS.10).aspx

<!---HONumber=AcomDC_0211_2016-->