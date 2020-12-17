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
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="56bfd-103">Implantar uma solução de detecção de clientela baseada em IA usando o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="56bfd-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="56bfd-104">Este artigo descreve como implantar uma solução baseada em IA que gera insights de ações do mundo real usando o Azure, o Azure Stack Hub e o Kit de desenvolvimento de IA da Visão Personalizada.</span><span class="sxs-lookup"><span data-stu-id="56bfd-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="56bfd-105">Nesta solução, você aprenderá a:</span><span class="sxs-lookup"><span data-stu-id="56bfd-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="56bfd-106">Implantar CNABs (Pacotes de aplicativos nativos de nuvem) na borda.</span><span class="sxs-lookup"><span data-stu-id="56bfd-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="56bfd-107">Implantar um aplicativo que abranja os limites da nuvem.</span><span class="sxs-lookup"><span data-stu-id="56bfd-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="56bfd-108">Use o Kit de desenvolvimento de IA da Visão Personalizada para inferência na borda.</span><span class="sxs-lookup"><span data-stu-id="56bfd-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="56bfd-109">![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="56bfd-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="56bfd-110">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="56bfd-111">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="56bfd-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="56bfd-112">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="56bfd-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="56bfd-113">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="56bfd-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="56bfd-114">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="56bfd-114">Prerequisites</span></span>

<span data-ttu-id="56bfd-115">Antes de começar a usar este guia de implantação, você deve:</span><span class="sxs-lookup"><span data-stu-id="56bfd-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="56bfd-116">Examinar o tópico [Padrão de detecção de clientela](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="56bfd-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="56bfd-117">Obter o acesso de usuário a um ASDK (Kit de Desenvolvimento do Azure Stack) ou a uma instância do sistema integrado do Azure Stack Hub com:</span><span class="sxs-lookup"><span data-stu-id="56bfd-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="56bfd-118">O [Serviço de Aplicativo do Azure no provedor de recursos do Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) instalado.</span><span class="sxs-lookup"><span data-stu-id="56bfd-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="56bfd-119">Você precisa de acesso de operador à sua instância do Azure Stack Hub ou terá de trabalhar com seu administrador para fazer a instalação.</span><span class="sxs-lookup"><span data-stu-id="56bfd-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="56bfd-120">Uma assinatura de uma oferta que forneça a cota de Serviço de Aplicativo e Armazenamento.</span><span class="sxs-lookup"><span data-stu-id="56bfd-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="56bfd-121">Você precisa ter acesso de operador para criar uma oferta.</span><span class="sxs-lookup"><span data-stu-id="56bfd-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="56bfd-122">Obter acesso a uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="56bfd-123">Caso você não tenha uma assinatura do Azure, inscreva-se em uma [conta de avaliação gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="56bfd-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="56bfd-124">Criar duas entidades de serviço em seu diretório:</span><span class="sxs-lookup"><span data-stu-id="56bfd-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="56bfd-125">Uma configuração para usar com recursos do Azure, com acesso no escopo da assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="56bfd-126">Uma configuração para usar com recursos do Azure Stack Hub, com acesso no escopo da assinatura do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="56bfd-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="56bfd-127">Para saber mais sobre como criar entidades de serviço e autorizar o acesso, confira [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="56bfd-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="56bfd-128">Caso prefira usar a CLI do Azure, confira [Criar uma entidade de serviço do Azure com a CLI do Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="56bfd-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="56bfd-129">Implantar Serviços Cognitivos do Azure no Azure ou no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="56bfd-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="56bfd-130">Primeiro, [saiba mais sobre os Serviços Cognitivos](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="56bfd-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="56bfd-131">Depois, visite [Implantar Serviços Cognitivos do Azure para Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar Serviços Cognitivos no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="56bfd-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="56bfd-132">Primeiro, você precisa se inscrever para acessar a versão prévia.</span><span class="sxs-lookup"><span data-stu-id="56bfd-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="56bfd-133">Clonar ou baixar um Kit de desenvolvimento de IA da Visão Personalizada não configurado.</span><span class="sxs-lookup"><span data-stu-id="56bfd-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="56bfd-134">Para obter detalhes, confira o [DevKit de IA da Visão](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="56bfd-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="56bfd-135">Criar uma conta do Power BI.</span><span class="sxs-lookup"><span data-stu-id="56bfd-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="56bfd-136">Uma chave de assinatura e um URL de ponto de extremidade da API de Detecção Facial dos Serviços Cognitivos do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="56bfd-137">É possível obter ambos com a avaliação gratuita [Experimente os Serviços Cognitivos](https://azure.microsoft.com/try/cognitive-services/?api=face-api).</span><span class="sxs-lookup"><span data-stu-id="56bfd-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="56bfd-138">Ou siga as instruções em [Criar uma conta de Serviços Cognitivos](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="56bfd-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="56bfd-139">Instalar os seguintes recursos de desenvolvimento:</span><span class="sxs-lookup"><span data-stu-id="56bfd-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="56bfd-140">CLI 2.0 do Azure</span><span class="sxs-lookup"><span data-stu-id="56bfd-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="56bfd-141">CE do Docker</span><span class="sxs-lookup"><span data-stu-id="56bfd-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="56bfd-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="56bfd-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="56bfd-143">Você usará o Porter para implantar os aplicativos de nuvem usando os manifestos de pacote do CNAB que lhe forem fornecidos.</span><span class="sxs-lookup"><span data-stu-id="56bfd-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="56bfd-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="56bfd-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="56bfd-145">Azure IoT Tools para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="56bfd-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="56bfd-146">Extensão do Python para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="56bfd-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="56bfd-147">Python</span><span class="sxs-lookup"><span data-stu-id="56bfd-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="56bfd-148">Implantar o aplicativo de nuvem híbrida</span><span class="sxs-lookup"><span data-stu-id="56bfd-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="56bfd-149">Primeiro, você usará a CLI do Porter para gerar um conjunto de credenciais e, em seguida, implantar o aplicativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="56bfd-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="56bfd-150">Clone ou baixe o código de exemplo da solução usando https://github.com/azure-samples/azure-intelligent-edge-patterns.</span><span class="sxs-lookup"><span data-stu-id="56bfd-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="56bfd-151">O Porter gerará um conjunto de credenciais que automatizará a implantação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="56bfd-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="56bfd-152">Antes de executar o comando de geração de credenciais, você deve ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="56bfd-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="56bfd-153">Uma entidade de serviço para acessar recursos do Azure, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="56bfd-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="56bfd-154">A ID de sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="56bfd-155">Uma entidade de serviço para acessar recursos do Azure Stack Hub, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="56bfd-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="56bfd-156">A ID de sua assinatura do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="56bfd-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="56bfd-157">Sua chave e seu URL de ponto de extremidade de recurso da API de Detecção Facial dos Serviços Cognitivos do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="56bfd-158">Execute o processo de geração de credencial do Porter e siga as solicitações:</span><span class="sxs-lookup"><span data-stu-id="56bfd-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="56bfd-159">O Porter também requer um conjunto de parâmetros para ser executado.</span><span class="sxs-lookup"><span data-stu-id="56bfd-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="56bfd-160">Crie um arquivo de texto de parâmetros e insira os seguintes pares de nome/valor.</span><span class="sxs-lookup"><span data-stu-id="56bfd-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="56bfd-161">Entre em contato com o administrador do Azure Stack Hub caso precise de assistência a respeito de algum dos valores necessários.</span><span class="sxs-lookup"><span data-stu-id="56bfd-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="56bfd-162">O valor `resource suffix` é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="56bfd-163">Ele deve ser uma cadeia de caracteres exclusiva com letras e números e até 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="56bfd-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="56bfd-164">Salve o arquivo de texto e anote seu caminho.</span><span class="sxs-lookup"><span data-stu-id="56bfd-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="56bfd-165">Agora você já pode implantar o aplicativo de nuvem híbrida usando o Porter.</span><span class="sxs-lookup"><span data-stu-id="56bfd-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="56bfd-166">Execute o comando de instalação e veja os recursos sendo implantados no Azure e no Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="56bfd-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="56bfd-167">Assim que a implantação for concluída, anote os seguintes valores:</span><span class="sxs-lookup"><span data-stu-id="56bfd-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="56bfd-168">A cadeia de conexão da câmera.</span><span class="sxs-lookup"><span data-stu-id="56bfd-168">The camera's connection string.</span></span>
    - <span data-ttu-id="56bfd-169">A cadeia de conexão da conta de armazenamento de imagens.</span><span class="sxs-lookup"><span data-stu-id="56bfd-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="56bfd-170">Os nomes do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="56bfd-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="56bfd-171">Preparar o DevKit de IA da Visão Personalizada</span><span class="sxs-lookup"><span data-stu-id="56bfd-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="56bfd-172">Depois, configure o DevKit de IA da Visão Personalizada conforme mostrado no [Guia de início rápido do DevKit de IA da Visão](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="56bfd-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="56bfd-173">Configure e teste também sua câmera usando a cadeia de conexão fornecida na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="56bfd-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="56bfd-174">Implantar o aplicativo da câmera</span><span class="sxs-lookup"><span data-stu-id="56bfd-174">Deploy the camera app</span></span>

<span data-ttu-id="56bfd-175">Use a CLI do Porter para gerar um conjunto de credenciais, depois implante o aplicativo da câmera.</span><span class="sxs-lookup"><span data-stu-id="56bfd-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="56bfd-176">O Porter gerará um conjunto de credenciais que automatizará a implantação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="56bfd-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="56bfd-177">Antes de executar o comando de geração de credenciais, você deve ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="56bfd-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="56bfd-178">Uma entidade de serviço para acessar recursos do Azure, inclusive a ID da entidade de serviço, a chave, e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="56bfd-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="56bfd-179">A ID de sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="56bfd-180">A cadeia de conexão da conta de armazenamento de imagens fornecida quando você implantou o aplicativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="56bfd-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="56bfd-181">Execute o processo de geração de credencial do Porter e siga as solicitações:</span><span class="sxs-lookup"><span data-stu-id="56bfd-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="56bfd-182">O Porter também requer um conjunto de parâmetros para ser executado.</span><span class="sxs-lookup"><span data-stu-id="56bfd-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="56bfd-183">Crie um arquivo de texto de parâmetros e insira o texto a seguir.</span><span class="sxs-lookup"><span data-stu-id="56bfd-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="56bfd-184">Entre em contato com o administrador do Azure Stack Hub caso desconheça algum dos valores exigidos.</span><span class="sxs-lookup"><span data-stu-id="56bfd-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="56bfd-185">O valor `deployment suffix` é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure.</span><span class="sxs-lookup"><span data-stu-id="56bfd-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="56bfd-186">Ele deve ser uma cadeia de caracteres exclusiva com letras e números e até 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="56bfd-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="56bfd-187">Salve o arquivo de texto e anote seu caminho.</span><span class="sxs-lookup"><span data-stu-id="56bfd-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="56bfd-188">Agora você já pode implantar o aplicativo da câmera usando o Porter.</span><span class="sxs-lookup"><span data-stu-id="56bfd-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="56bfd-189">Execute o comando install e veja a implantação do IoT Edge ser criada.</span><span class="sxs-lookup"><span data-stu-id="56bfd-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="56bfd-190">Para verificar se a implantação foi concluída, exiba o feed da câmera em `https://<camera-ip>:3000/`, onde `<camara-ip>` é o endereço IP da câmera.</span><span class="sxs-lookup"><span data-stu-id="56bfd-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="56bfd-191">Essa etapa pode levar até 10 minutos.</span><span class="sxs-lookup"><span data-stu-id="56bfd-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="56bfd-192">Configurar o Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="56bfd-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="56bfd-193">Agora que os dados estão fluindo para o Azure Stream Analytics a partir da câmera, precisamos autorizá-los manualmente para se comunicarem com o Power BI.</span><span class="sxs-lookup"><span data-stu-id="56bfd-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="56bfd-194">No portal do Azure, abra **Todos os Recursos** e o trabalho *process-footfall\[yoursuffix\]* .</span><span class="sxs-lookup"><span data-stu-id="56bfd-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="56bfd-195">Na seção **Topologia do Trabalho** do painel do trabalho do Stream Analytics, selecione a opção **Saídas**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="56bfd-196">Selecione o coletor de saída **traffic-output**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="56bfd-197">Selecione **Renovar autorização** e entre em sua conta do Power BI.</span><span class="sxs-lookup"><span data-stu-id="56bfd-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Solicitação de renovação de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="56bfd-199">Salve as configurações de saída.</span><span class="sxs-lookup"><span data-stu-id="56bfd-199">Save the output settings.</span></span>

6. <span data-ttu-id="56bfd-200">Acesse o painel **Visão geral** e selecione **Iniciar** para começar a enviar dados ao Power BI.</span><span class="sxs-lookup"><span data-stu-id="56bfd-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="56bfd-201">Selecione **Agora** para a hora de início da saída do trabalho e selecione **Iniciar**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="56bfd-202">Você pode exibir o status do trabalho na barra de notificação.</span><span class="sxs-lookup"><span data-stu-id="56bfd-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="56bfd-203">Criar um painel do Power BI</span><span class="sxs-lookup"><span data-stu-id="56bfd-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="56bfd-204">Depois que o trabalho for concluído com êxito, acesse o [Power BI](https://powerbi.com/) e entre com sua conta corporativa ou de estudante.</span><span class="sxs-lookup"><span data-stu-id="56bfd-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="56bfd-205">Se a consulta do trabalho do Stream Analytics estiver gerando resultados, o conjunto de dados *footfall-dataset* criado estará presente na guia **Conjuntos de Dados**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="56bfd-206">No espaço de trabalho do Power BI, escolha **+ Criar** para criar um painel novo chamado *Análise de Clientela*.</span><span class="sxs-lookup"><span data-stu-id="56bfd-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="56bfd-207">Na parte superior da janela, escolha **Adicionar bloco**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="56bfd-208">Em seguida, escolha **Fluxo de Dados Personalizado** e **Avançar**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="56bfd-209">Escolha o **footfall-dataset** em **Seus Conjuntos de Dados**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="56bfd-210">Escolha **Cartão** na lista suspensa **Tipo de visualização** e adicione **período** em **Campos**.</span><span class="sxs-lookup"><span data-stu-id="56bfd-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="56bfd-211">Escolha **Avançar** para inserir um nome para o bloco e escolha **Aplicar** para criar o bloco.</span><span class="sxs-lookup"><span data-stu-id="56bfd-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="56bfd-212">Você pode adicionar campos e cartões conforme desejado.</span><span class="sxs-lookup"><span data-stu-id="56bfd-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="56bfd-213">Testar sua solução</span><span class="sxs-lookup"><span data-stu-id="56bfd-213">Test Your Solution</span></span>

<span data-ttu-id="56bfd-214">Observe como os dados nos cartões criados no Power BI mudam conforme pessoas diferentes se movimentam na frente da câmera.</span><span class="sxs-lookup"><span data-stu-id="56bfd-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="56bfd-215">Depois de registradas, as inferências podem levar até 20 segundos para aparecer.</span><span class="sxs-lookup"><span data-stu-id="56bfd-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="56bfd-216">Remover sua solução</span><span class="sxs-lookup"><span data-stu-id="56bfd-216">Remove Your Solution</span></span>

<span data-ttu-id="56bfd-217">Caso queira remover sua solução, execute os comandos a seguir usando o Porter, empregando os mesmos arquivos de parâmetro que você criou para a implantação:</span><span class="sxs-lookup"><span data-stu-id="56bfd-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="56bfd-218">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="56bfd-218">Next steps</span></span>

- <span data-ttu-id="56bfd-219">Saiba mais sobre as [Considerações de design do aplicativo híbrido](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="56bfd-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="56bfd-220">Revise e proponha melhorias para [o código desse exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="56bfd-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
