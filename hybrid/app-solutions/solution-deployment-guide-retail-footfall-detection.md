---
title: Implantar solução de detecção de footfall com base em ia no Azure e no Hub de Azure Stack
description: Saiba como implantar uma solução de detecção de footfall com base em ia para analisar o tráfego de visitantes em lojas de varejo usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909787"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Implantar uma solução de detecção de footfall com base em ia usando o Azure e o Hub de Azure Stack

Este artigo descreve como implantar uma solução baseada em ia que gera informações de ações do mundo real usando o Azure, o Hub de Azure Stack e o kit de desenvolvimento de ia Visão Personalizada.

Nesta solução, você aprenderá a:

> [!div class="checklist"]
> - Implantar CNAB (pacotes de aplicativos nativos de nuvem) na borda. 
> - Implante um aplicativo que abranja os limites de nuvem.
> - Use o Visão Personalizada ia dev Kit para inferência na borda.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar a usar este guia de implantação, verifique se você:

- Examine o tópico [padrão de detecção de footfall](pattern-retail-footfall-detection.md) .
- Obtenha acesso de usuário a uma instância de sistema integrada de Kit de Desenvolvimento do Azure Stack (ASDK) ou Azure Stack Hub, com:
  - O [serviço Azure app no provedor de recursos do Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md) instalado. Você precisa de acesso de operador à sua instância de Hub de Azure Stack ou trabalhar com o administrador para instalar o.
  - Uma assinatura para uma oferta que fornece a cota de armazenamento e serviço de aplicativo. Você precisa de acesso de operador para criar uma oferta.
- Obter acesso a uma assinatura do Azure.
  - Se você não tiver uma assinatura do Azure, Inscreva-se para uma [conta de avaliação gratuita](https://azure.microsoft.com/free/) antes de começar.
- Crie duas entidades de serviço em seu diretório:
  - Uma configuração para uso com recursos do Azure, com acesso no escopo de assinatura do Azure.
  - Uma configuração para uso com Azure Stack recursos do Hub, com acesso no escopo de assinatura do Hub Azure Stack.
  - Para saber mais sobre como criar entidades de serviço e autorizar o acesso, consulte [usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md). Se preferir usar CLI do Azure, consulte [criar uma entidade de serviço do Azure com CLI do Azure](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).
- Implante serviços cognitivas do Azure no Azure ou Azure Stack Hub.
  - Primeiro, [saiba mais sobre os serviços cognitivas](https://azure.microsoft.com/services/cognitive-services/).
  - Em seguida, visite [implantar serviços cognitivas do Azure para Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar serviços cognitivas no Hub Azure Stack. Primeiro, você precisa inscrever-se para acessar a versão prévia.
- Clone ou baixe um kit de desenvolvimento de ia Visão Personalizada do Azure não configurado. Para obter detalhes, consulte a [visão de ai devkit](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Inscreva-se para uma conta de Power BI.
- Um serviço cognitiva do Azure API de Detecção Facial a chave de assinatura e a URL do ponto de extremidade. Você pode obter ambos com a avaliação gratuita [experimentar serviços cognitivas](https://azure.microsoft.com/try/cognitive-services/?api=face-api) . Ou siga as instruções em [criar uma conta de serviços cognitivas](/azure/cognitive-services/cognitive-services-apis-create-account).
- Instale os seguintes recursos de desenvolvimento:
  - [CLI do Azure 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Carregador](https://porter.sh/). Você usa o carregador para implantar aplicativos de nuvem usando manifestos de pacote do CNAB que são fornecidos para você.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Ferramentas de IoT do Azure para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Extensão do Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Implantar o aplicativo de nuvem híbrida

Primeiro, você usa a CLI carregador para gerar um conjunto de credenciais e, em seguida, implantar o aplicativo de nuvem.  

1. Clone ou baixe o código de exemplo da solução de https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. O carregador irá gerar um conjunto de credenciais que automatizará a implantação do aplicativo. Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:

    - Uma entidade de serviço para acessar recursos do Azure, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.
    - A ID da assinatura para sua assinatura do Azure.
    - Uma entidade de serviço para acessar Azure Stack recursos do Hub, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.
    - A ID da assinatura para sua assinatura do hub de Azure Stack.
    - Os serviços cognitivas do Azure API de Detecção Facial URL de ponto de extremidade de recurso e chave.

1. Execute o processo de geração de credencial carregador e siga os prompts:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. O carregador também requer um conjunto de parâmetros a serem executados. Crie um arquivo de texto de parâmetro e insira os seguintes pares de nome/valor. Pergunte ao administrador do Hub do Azure Stack se você precisar de assistência com qualquer um dos valores necessários.

   > [!NOTE] 
   > O `resource suffix` valor é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure. Ele deve ser uma cadeia de caracteres exclusiva de letras e números, com até 8 caracteres.

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   Salve o arquivo de texto e anote seu caminho.

1. Agora você está pronto para implantar o aplicativo de nuvem híbrida usando o carregador. Execute o comando de instalação e assista à medida que os recursos são implantados no Azure e no Hub de Azure Stack:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Após a conclusão da implantação, anote os seguintes valores:
    - A cadeia de conexão da câmera.
    - A cadeia de conexão da conta de armazenamento de imagens.
    - Os nomes do grupo de recursos.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Preparar o Visão Personalizada ia DevKit

Em seguida, configure o kit de desenvolvimento do ia Visão Personalizada como mostrado no guia de [início rápido do ia devkit de visão](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). Você também configura e testa sua câmera usando a cadeia de conexão fornecida na etapa anterior.

## <a name="deploy-the-camera-app"></a>Implantar o aplicativo de câmera

Use a CLI do carregador para gerar um conjunto de credenciais e, em seguida, implante o aplicativo de câmera.

1. O carregador irá gerar um conjunto de credenciais que automatizará a implantação do aplicativo. Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:

    - Uma entidade de serviço para acessar recursos do Azure, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.
    - A ID da assinatura para sua assinatura do Azure.
    - A cadeia de conexão da conta de armazenamento de imagem fornecida quando você implantou o aplicativo de nuvem.

1. Execute o processo de geração de credencial carregador e siga os prompts:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. O carregador também requer um conjunto de parâmetros a serem executados. Crie um arquivo de texto de parâmetro e insira o texto a seguir. Pergunte ao administrador do Hub do Azure Stack se você não souber alguns dos valores necessários.

    > [!NOTE]
    > O `deployment suffix` valor é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure. Ele deve ser uma cadeia de caracteres exclusiva de letras e números, com até 8 caracteres.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Salve o arquivo de texto e anote seu caminho.

4. Agora você está pronto para implantar o aplicativo de câmera usando o carregador. Execute o comando install e veja como a implantação do IoT Edge é criada.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Verifique se a implantação da câmera está concluída exibindo o feed de câmera em `https://<camera-ip>:3000/` , em que `<camara-ip>` é o endereço IP da câmera. Esta etapa pode levar até 10 minutos.

## <a name="configure-azure-stream-analytics"></a>Configurar o Azure Stream Analytics

Agora que os dados estão fluindo para Azure Stream Analytics da câmera, precisamos autorizá-lo manualmente para se comunicar com Power BI.

1. No portal do Azure, abra **todos os recursos**e o trabalho * \[ yoursuffix \] de footfall de processo* .

2. Na seção **Topologia do Trabalho** do painel do trabalho do Stream Analytics, selecione a opção **Saídas**.

3. Selecione o coletor de saída de **tráfego de saída** .

4. Selecione **renovar autorização** e entre em sua conta do Power bi.
  
    ![Renovar prompt de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Salve as configurações de saída.

6. Vá para o painel **visão geral** e selecione **Iniciar** para começar a enviar dados para Power bi.

7. Selecione **Agora** para a hora de início da saída do trabalho e selecione **Iniciar**. Você pode exibir o status do trabalho na barra de notificação.

## <a name="create-a-power-bi-dashboard"></a>Criar um painel de Power BI

1. Quando o trabalho for bem sucedido, vá para [Power bi](https://powerbi.com/) e entre com sua conta corporativa ou de estudante. Se a consulta do trabalho de Stream Analytics estiver gerando resultados, o conjunto de *footfall* do conjunto de valores que você criou existirá na guia **DataSets** .

2. No espaço de trabalho Power BI, selecione **+ criar** para criar um novo painel chamado *análise de footfall.*

3. Na parte superior da janela, escolha **Adicionar bloco**. Em seguida, escolha **Fluxo de Dados Personalizado** e **Avançar**. Escolha o **footfall-DataSet** em **seus conjuntos de seus DataSets**. Selecione **cartão** na lista suspensa **tipo de visualização** e adicione **idade** a **campos**. Escolha **Avançar** para inserir um nome para o bloco e escolha **Aplicar** para criar o bloco.

4. Você pode adicionar outros campos e cartões conforme desejado.

## <a name="test-your-solution"></a>Testar sua solução

Observe como os dados nos cartões criados no Power BI mudam conforme as pessoas diferentes se movimentam na frente da câmera. As inferências podem levar até 20 segundos para aparecer uma vez registradas.

## <a name="remove-your-solution"></a>Remover sua solução

Se você quiser remover sua solução, execute os comandos a seguir usando carregador, usando os mesmos arquivos de parâmetro que você criou para implantação:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre [considerações de design de aplicativo híbrido]. (overview-app-design-considerations.md)
- Revise e proponha melhorias ao [código para este exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
