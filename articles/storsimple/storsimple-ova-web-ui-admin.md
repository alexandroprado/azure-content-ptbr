<properties 
   pageTitle="Administração de interface do usuário da Web da StorSimple Virtual Array | Microsoft Azure"
   description="Descreve como executar tarefas de administração básicas do dispositivo por meio da interface do usuário da Web do StorSimple Virtual Array."
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="03/01/2016"
   ms.author="alkohli" />

# Usar a interface do usuário da Web para administrar o StorSimple Virtual Array

![fluxo do processo de instalação](./media/storsimple-ova-web-ui-admin/manage4.png)

## Visão geral

Os tutoriais descritos neste artigo se aplicam ao Microsoft Azure StorSimple Virtual Array (também conhecido como o dispositivo virtual local StorSimple) que executa a versão GA (disponibilidade geral) de março de 2016. Este artigo descreve alguns dos fluxos de trabalho complexos e tarefas de gerenciamento que podem ser executadas na StorSimple Virtual Array. É possível gerenciar o StorSimple Virtual Array usando a interface do usuário do serviço StorSimple Manager (conhecida como interface do usuário do portal) e a interface de usuário da Web local para o dispositivo. Este artigo se concentra nas tarefas que você pode executar usando a interface do usuário da Web.

Este artigo inclui os seguintes tutoriais:

- Obter a chave de criptografia de dados do serviço
- Solucionar problemas de erros de instalação de interface do usuário da Web
- Gerar um pacote de log
- Desligar ou reiniciar seu dispositivo

## Obter a chave de criptografia de dados do serviço

Uma chave de criptografia de dados de serviço é gerada quando você registra o primeiro dispositivo com o serviço StorSimple Manager. Essa chave é então necessária com a chave de registro do serviço para registrar dispositivos adicionais no serviço StorSimple Manager.

Se você esqueceu sua chave de criptografia de dados de serviço e precisa recuperá-la, execute as seguintes etapas na interface do usuário da Web local do dispositivo registrado com o serviço.

#### Para obter a chave de criptografia de dados do serviço

1. Conecte-se à interface do usuário da Web local. Vá para **Configuração** > **Configurações de Nuvem**.

    ![obter chave de criptografia de dados do serviço 1](./media/storsimple-ova-web-ui-admin/image27.png)

2. Na parte inferior da página, clique em **Obter chave de criptografia de dados de serviço**. Uma chave será exibida. Copie e salve essa chave.

    ![obter chave de criptografia de dados do serviço 2](./media/storsimple-ova-web-ui-admin/image28.png)


## Solucionar problemas de erros de instalação de interface do usuário da Web

Em alguns casos em que você configura o dispositivo por meio da interface do usuário da Web local, você pode encontrar erros. Para diagnosticar e solucionar esses erros, você pode executar os testes de diagnóstico.

#### Para executar os testes de diagnóstico

1. Na interface do usuário da Web local, vá para **Solução de problemas** > **Testes de diagnóstico**.

    ![executar dignóstico 1](./media/storsimple-ova-web-ui-admin/image29.png)

2. Na parte inferior da página, clique em **Executar Testes de Diagnóstico**. Isso iniciará testes para diagnosticar quaisquer possíveis problemas com suas configurações de rede, dispositivo, proxy da Web, de hora ou de nuvem. Você será notificado de que o dispositivo está executando testes.

3. Depois de os testes terem sido concluídos, os resultados serão exibidos. O exemplo a seguir mostra os resultados dos testes de diagnóstico. Observe que as configurações de proxy da Web não foram definidas neste dispositivo e, portanto, o teste de proxy da Web não foi executado. Todos os outros testes para as configurações de rede, servidor DNS e hora foram bem-sucedidas.

    ![executar dignóstico 2](./media/storsimple-ova-web-ui-admin/image30.png)

## Gerar um pacote de log

Um pacote de log é composto de todos os logs relevantes que podem ajudar o Suporte da Microsoft na solução de quaisquer problemas do dispositivo. Nesta versão, um pacote de log pode ser gerado por meio da interface do usuário da Web local.

#### Para gerar o pacote de log

1. Na interface do usuário da Web local, vá para **Solução de problemas** > **Logs do sistema**.

    ![gerar pacote de log 1](./media/storsimple-ova-web-ui-admin/image31.png)

2. Na parte inferior da página, clique em **Criar pacote de log**. Será criado um pacote dos logs do sistema. Isso levará alguns minutos.

    ![gerar pacote de log 2](./media/storsimple-ova-web-ui-admin/image32.png)

    Você será notificado após o pacote ser criado com êxito e a página será atualizada para indicar a hora e data em que o pacote foi criado.

    ![gerar pacote de log 3](./media/storsimple-ova-web-ui-admin/image33.png)

3. Clique em **Baixar pacote de log**. Um pacote compactado será baixado em seu sistema.

    ![gerar pacote de log 4](./media/storsimple-ova-web-ui-admin/image34.png)

4. Você pode descompactar o pacote de log baixado e exibir os arquivos de log do sistema.

## Desligue e reinicie seu dispositivo

Você pode desligar ou reiniciar seu dispositivo virtual usando a interface do usuário da Web local. Nós recomendamos que antes de reiniciar, você execute os volumes ou compartilhamentos offline no host e, em seguida, no dispositivo. Isso minimizará a possibilidade de dados corrompidos.

#### Para desligar o dispositivo virtual

1. Na interface do usuário da Web local, vá para **Manutenção** > **Configurações de energia**.

2. Na parte inferior da página, clique em **Desligar**.

    ![desligamento de dispositivo 1](./media/storsimple-ova-web-ui-admin/image36.png)

3. Um aviso será exibido informando que um desligamento do dispositivo interromperá quaisquer E/Ss em andamento, resultando em um tempo de inatividade. Clique no ícone de verificação ![ícone de verificação](./media/storsimple-ova-web-ui-admin/image3.png).

    ![aviso de desligamento de dispositivo](./media/storsimple-ova-web-ui-admin/image37.png)

    Você será notificado de que o desligamento foi iniciado.

    ![o desligamento de dispositivo foi iniciado](./media/storsimple-ova-web-ui-admin/image38.png)

    O dispositivo será agora desligado. Se você deseja iniciar o dispositivo, você precisa fazer isso por meio do Gerenciador do Hyper-V.

#### Para reiniciar o dispositivo virtual

1. Na interface do usuário da Web local, vá para **Manutenção** > **Configurações de energia**.

2. Na parte inferior da página, clique em **Reiniciar**.

    ![reinicialização do dispositivo](./media/storsimple-ova-web-ui-admin/image36.png)

3. Um aviso será exibido informando que reiniciar o dispositivo interromperá quaisquer E/Ss em andamento, resultando em um tempo de inatividade. Clique no ícone de verificação ![ícone de verificação](./media/storsimple-ova-web-ui-admin/image3.png).

    ![aviso de reinicialização](./media/storsimple-ova-web-ui-admin/image37.png)

    Você será notificado de que a reinicialização foi iniciada.

    ![reinicialização iniciada](./media/storsimple-ova-web-ui-admin/image39.png)

    Enquanto a reinicialização estiver em andamento, você perderá a conexão à interface do usuário. Você pode monitorar a reinicialização atualizando a interface do usuário periodicamente. Como alternativa, você pode monitorar o status de reinicialização do dispositivo por meio do Gerenciador do Hyper-V.

## Próximas etapas

Saiba como [usar o serviço StorSimple Manager para gerenciar seu dispositivo](storsimple-manager-service-administration.md).

<!---HONumber=AcomDC_0302_2016-->