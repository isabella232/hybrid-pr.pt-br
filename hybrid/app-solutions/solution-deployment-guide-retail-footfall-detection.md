---
title: Implantar solução de detecção de clientela baseada em IA no Azure e no Azure Stack Hub
description: Saiba como implantar uma solução de detecção de clientela baseada em IA para analisar o tráfego de visitantes em lojas de varejo usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901483"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Implantar uma solução de detecção de clientela baseada em IA usando o Azure e o Azure Stack Hub

Este artigo descreve como implantar uma solução baseada em IA que gera insights de ações do mundo real usando o Azure, o Azure Stack Hub e o Kit de desenvolvimento de IA da Visão Personalizada.

Nesta solução, você aprenderá a:

> [!div class="checklist"]
> - Implantar CNABs (Pacotes de aplicativos nativos de nuvem) na borda. 
> - Implantar um aplicativo que abranja os limites da nuvem.
> - Use o Kit de desenvolvimento de IA da Visão Personalizada para inferência na borda.

> [!Tip]  
> ![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar a usar este guia de implantação, você deve:

- Examinar o tópico [Padrão de detecção de clientela](pattern-retail-footfall-detection.md).
- Obter o acesso de usuário a um ASDK (Kit de Desenvolvimento do Azure Stack) ou a uma instância do sistema integrado do Azure Stack Hub com:
  - O [Serviço de Aplicativo do Azure no provedor de recursos do Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) instalado. Você precisa de acesso de operador à sua instância do Azure Stack Hub ou terá de trabalhar com seu administrador para fazer a instalação.
  - Uma assinatura de uma oferta que forneça a cota de Serviço de Aplicativo e Armazenamento. Você precisa ter acesso de operador para criar uma oferta.
- Obter acesso a uma assinatura do Azure.
  - Caso você não tenha uma assinatura do Azure, inscreva-se em uma [conta de avaliação gratuita](https://azure.microsoft.com/free/) antes de começar.
- Criar duas entidades de serviço em seu diretório:
  - Uma configuração para usar com recursos do Azure, com acesso no escopo da assinatura do Azure.
  - Uma configuração para usar com recursos do Azure Stack Hub, com acesso no escopo da assinatura do Azure Stack Hub.
  - Para saber mais sobre como criar entidades de serviço e autorizar o acesso, confira [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md). Caso prefira usar a CLI do Azure, confira [Criar uma entidade de serviço do Azure com a CLI do Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).
- Implantar Serviços Cognitivos do Azure no Azure ou no Azure Stack Hub.
  - Primeiro, [saiba mais sobre os Serviços Cognitivos](https://azure.microsoft.com/services/cognitive-services/).
  - Depois, visite [Implantar Serviços Cognitivos do Azure para Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar Serviços Cognitivos no Azure Stack Hub. Primeiro, você precisa se inscrever para acessar a versão prévia.
- Clonar ou baixar um Kit de desenvolvimento de IA da Visão Personalizada não configurado. Para obter detalhes, confira o [DevKit de IA da Visão](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Criar uma conta do Power BI.
- Uma chave de assinatura e um URL de ponto de extremidade da API de Detecção Facial dos Serviços Cognitivos do Azure. É possível obter ambos com a avaliação gratuita [Experimente os Serviços Cognitivos](https://azure.microsoft.com/try/cognitive-services/?api=face-api). Ou siga as instruções em [Criar uma conta de Serviços Cognitivos](/azure/cognitive-services/cognitive-services-apis-create-account).
- Instalar os seguintes recursos de desenvolvimento:
  - [CLI 2.0 do Azure](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [CE do Docker](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Você usará o Porter para implantar os aplicativos de nuvem usando os manifestos de pacote do CNAB que lhe forem fornecidos.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Extensão do Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Implantar o aplicativo de nuvem híbrida

Primeiro, você usará a CLI do Porter para gerar um conjunto de credenciais e, em seguida, implantar o aplicativo de nuvem.  

1. Clone ou baixe o código de exemplo da solução usando https://github.com/azure-samples/azure-intelligent-edge-patterns. 

1. O Porter gerará um conjunto de credenciais que automatizará a implantação do aplicativo. Antes de executar o comando de geração de credenciais, você deve ter o seguinte disponível:

    - Uma entidade de serviço para acessar recursos do Azure, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.
    - A ID de sua assinatura do Azure.
    - Uma entidade de serviço para acessar recursos do Azure Stack Hub, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.
    - A ID de sua assinatura do Azure Stack Hub.
    - Sua chave e seu URL de ponto de extremidade de recurso da API de Detecção Facial dos Serviços Cognitivos do Azure.

1. Execute o processo de geração de credencial do Porter e siga as solicitações:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. O Porter também requer um conjunto de parâmetros para ser executado. Crie um arquivo de texto de parâmetros e insira os seguintes pares de nome/valor. Entre em contato com o administrador do Azure Stack Hub caso precise de assistência a respeito de algum dos valores necessários.

   > [!NOTE] 
   > O valor `resource suffix` é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure. Ele deve ser uma cadeia de caracteres exclusiva com letras e números e até 8 caracteres.

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

1. Agora você já pode implantar o aplicativo de nuvem híbrida usando o Porter. Execute o comando de instalação e veja os recursos sendo implantados no Azure e no Azure Stack Hub:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Assim que a implantação for concluída, anote os seguintes valores:
    - A cadeia de conexão da câmera.
    - A cadeia de conexão da conta de armazenamento de imagens.
    - Os nomes do grupo de recursos.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Preparar o DevKit de IA da Visão Personalizada

Depois, configure o DevKit de IA da Visão Personalizada conforme mostrado no [Guia de início rápido do DevKit de IA da Visão](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). Configure e teste também sua câmera usando a cadeia de conexão fornecida na etapa anterior.

## <a name="deploy-the-camera-app"></a>Implantar o aplicativo da câmera

Use a CLI do Porter para gerar um conjunto de credenciais, depois implante o aplicativo da câmera.

1. O Porter gerará um conjunto de credenciais que automatizará a implantação do aplicativo. Antes de executar o comando de geração de credenciais, você deve ter o seguinte disponível:

    - Uma entidade de serviço para acessar recursos do Azure, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.
    - A ID de sua assinatura do Azure.
    - A cadeia de conexão da conta de armazenamento de imagens fornecida quando você implantou o aplicativo de nuvem.

1. Execute o processo de geração de credencial do Porter e siga as solicitações:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. O Porter também requer um conjunto de parâmetros para ser executado. Crie um arquivo de texto de parâmetros e insira o texto a seguir. Entre em contato com o administrador do Azure Stack Hub caso desconheça algum dos valores exigidos.

    > [!NOTE]
    > O valor `deployment suffix` é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure. Ele deve ser uma cadeia de caracteres exclusiva com letras e números e até 8 caracteres.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Salve o arquivo de texto e anote seu caminho.

4. Agora você já pode implantar o aplicativo da câmera usando o Porter. Execute o comando install e veja a implantação do IoT Edge ser criada.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Para verificar se a implantação foi concluída, exiba o feed da câmera em `https://<camera-ip>:3000/`, onde `<camara-ip>` é o endereço IP da câmera. Essa etapa pode levar até 10 minutos.

## <a name="configure-azure-stream-analytics"></a>Configurar o Azure Stream Analytics

Agora que os dados estão fluindo para o Azure Stream Analytics a partir da câmera, precisamos autorizá-los manualmente para se comunicarem com o Power BI.

1. No portal do Azure, abra **Todos os Recursos** e o trabalho *process-footfall\[yoursuffix\]* .

2. Na seção **Topologia do Trabalho** do painel do trabalho do Stream Analytics, selecione a opção **Saídas**.

3. Selecione o coletor de saída **traffic-output**.

4. Selecione **Renovar autorização** e entre em sua conta do Power BI.
  
    ![Solicitação de renovação de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Salve as configurações de saída.

6. Acesse o painel **Visão geral** e selecione **Iniciar** para começar a enviar dados ao Power BI.

7. Selecione **Agora** para a hora de início da saída do trabalho e selecione **Iniciar**. Você pode exibir o status do trabalho na barra de notificação.

## <a name="create-a-power-bi-dashboard"></a>Criar um painel do Power BI

1. Depois que o trabalho for concluído com êxito, acesse o [Power BI](https://powerbi.com/) e entre com sua conta corporativa ou de estudante. Se a consulta do trabalho do Stream Analytics estiver gerando resultados, o conjunto de dados *footfall-dataset* criado estará presente na guia **Conjuntos de Dados**.

2. No espaço de trabalho do Power BI, escolha **+ Criar** para criar um painel novo chamado *Análise de Clientela*.

3. Na parte superior da janela, escolha **Adicionar bloco**. Em seguida, escolha **Fluxo de Dados Personalizado** e **Avançar**. Escolha o **footfall-dataset** em **Seus Conjuntos de Dados**. Escolha **Cartão** na lista suspensa **Tipo de visualização** e adicione **período** em **Campos**. Escolha **Avançar** para inserir um nome para o bloco e escolha **Aplicar** para criar o bloco.

4. Você pode adicionar campos e cartões conforme desejado.

## <a name="test-your-solution"></a>Testar sua solução

Observe como os dados nos cartões criados no Power BI mudam conforme pessoas diferentes se movimentam na frente da câmera. Depois de registradas, as inferências podem levar até 20 segundos para aparecer.

## <a name="remove-your-solution"></a>Remover sua solução

Caso queira remover sua solução, execute os comandos a seguir usando o Porter, empregando os mesmos arquivos de parâmetro que você criou para a implantação:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre as [Considerações de design do aplicativo híbrido](overview-app-design-considerations.md)
- Revise e proponha melhorias para [o código desse exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
