<properties 
   pageTitle="Substituir um controlador EBOD StorSimple | Microsoft Azure"
   description="Explica como remover e substituir um ou ambos os controladores EBOD em um dispositivo StorSimple 8600."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carolz"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="12/02/2015"
   ms.author="alkohli" />

# Substituir um controlador EBOD em seu dispositivo StorSimple

## Visão geral

Este tutorial explica como substituir um módulo do controlador EBOD com falha no dispositivo Microsoft Azure StorSimple. Para substituir um módulo do controlador EBOD, você precisa:

- Remover o controlador EBOD com falha
- Instalar um novo controlador EBOD

Antes de começar, considere as seguintes informações:

- Módulos EBOD em branco precisam ser inseridos em todos os slots não utilizados. O compartimento não será resfriado corretamente se um slot for deixado aberto.

- O controlador EBOD é intercambiável e pode ser removido ou substituído. Não remova um módulo com falha até que você tenha uma peça de reposição. Depois de iniciado o processo de substituição, ele precisa ser concluído em até 10 minutos.

>[AZURE.IMPORTANT]Antes de remover e substituir um controlador EBOD, examine as informações de segurança em [Substituição de componentes de hardware do StorSimple](storsimple-hardware-component-replacement.md).

## Remover um controlador EBOD

Antes de substituir o módulo do controlador EBOD com falha em seu dispositivo StorSimple, verifique se o outro módulo do controlador EBOD está ativo e em execução. O procedimento e a tabela a seguir explicam como remover o módulo do controlador EBOD.

#### Para remover um módulo EBOD

1. Abra o Portal Clássico do Azure.

2. Navegue até **Dispositivos** > **Manutenção** > **Status de Hardware** e verifique se o status do LED do módulo do controlador EBOD ativo está verde e o LED do módulo do controlador EBOD com falha está vermelho.

3. Localize o módulo do controlador EBOD na parte posterior do dispositivo.

4. Remova os cabos que conectam o módulo do controlador EBOD ao controlador antes de remover o módulo EBOD do sistema.

5. Anote a porta SAS exata do módulo do controlador EBOD que foi conectado ao controlador. Você precisará restaurar o sistema de acordo com essa configuração depois de substituir o módulo EBOD.

    >[AZURE.NOTE]Normalmente, será a porta A, que é rotulada como **Host in** no diagrama a seguir.

    ![Backplane do controlador EBOD](./media/storsimple-ebod-controller-replacement/IC741049.png)

     **Figura 1** Parte posterior do módulo EBOD

    |Rótulo|Descrição|
    |:----|:----------|
    |1|LED de falha|
    |2|LED de energia|
    |3|Conectores SAS|
    |4|LEDs de SAS|
    |5|Portas seriais apenas para uso em fábrica|
    |6|Porta A (Host in)|
    |7|Porta B (Host out)|
    |8|Porta C (Apenas para uso em fábrica)|

## Instalar um novo controlador EBOD

O procedimento e a tabela a seguir explicam como instalar um módulo do controlador EBOD em seu dispositivo StorSimple.

#### Para instalar um controlador EBOD

1. Verifique o dispositivo EBOD quanto a danos, especialmente no conector de interface. Não instale o novo controlador EBOD se algum dos pinos estiver dobrado.

2. Com as travas na posição aberta, deslize o módulo para dentro do compartimento até que as travas travem.

    ![Instalando o controlador EBOD](./media/storsimple-ebod-controller-replacement/IC741050.png)

    **Figura 2** Instalando o módulo do controlador EBOD

3. Feche a trava. Você deve ouvir um clique ao travar a trava.

    ![Liberando a trava do EBOD](./media/storsimple-ebod-controller-replacement/IC741047.png)

    **Figura 3** Fechando a trava do módulo EBOD

4. Reconecte os cabos. Use exatamente a mesma configuração de antes da substituição. Consulte o diagrama e a tabela a seguir para obter detalhes sobre como conectar os cabos.

    ![Cabeamento do dispositivo 4U para alimentação](./media/storsimple-ebod-controller-replacement/IC770723.png)

    **Figura 4** Reconectando os cabos

    |Rótulo|Descrição|
    |:----|:----------|
    |1|Compartimento principal|
    |2|PCM 0|
    |3|PCM 1|
    |4|Controlador 0|
    |5|Controlador 1|
    |6|Controlador 0 do EBOD|
    |7|Controlador 1 do EBOD|
    |8|Compartimento EBOD|
    |9|PDUs|

## Próximas etapas

Saiba mais sobre [substituição de componentes de hardware do StorSimple](storsimple-hardware-component-replacement.md).

<!---HONumber=AcomDC_1203_2015-->