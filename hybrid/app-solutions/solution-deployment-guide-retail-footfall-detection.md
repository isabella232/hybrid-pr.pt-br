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
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="d08e1-103">Implantar uma solução de detecção de footfall com base em ia usando o Azure e o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="d08e1-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d08e1-104">Este artigo descreve como implantar uma solução baseada em ia que gera informações de ações do mundo real usando o Azure, o Hub de Azure Stack e o kit de desenvolvimento de ia Visão Personalizada.</span><span class="sxs-lookup"><span data-stu-id="d08e1-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="d08e1-105">Nesta solução, você aprenderá a:</span><span class="sxs-lookup"><span data-stu-id="d08e1-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d08e1-106">Implantar CNAB (pacotes de aplicativos nativos de nuvem) na borda.</span><span class="sxs-lookup"><span data-stu-id="d08e1-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="d08e1-107">Implante um aplicativo que abranja os limites de nuvem.</span><span class="sxs-lookup"><span data-stu-id="d08e1-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="d08e1-108">Use o Visão Personalizada ia dev Kit para inferência na borda.</span><span class="sxs-lookup"><span data-stu-id="d08e1-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="d08e1-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d08e1-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d08e1-110">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d08e1-111">Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="d08e1-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d08e1-112">O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="d08e1-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d08e1-113">As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="d08e1-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d08e1-114">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="d08e1-114">Prerequisites</span></span>

<span data-ttu-id="d08e1-115">Antes de começar a usar este guia de implantação, verifique se você:</span><span class="sxs-lookup"><span data-stu-id="d08e1-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="d08e1-116">Examine o tópico [padrão de detecção de footfall](pattern-retail-footfall-detection.md) .</span><span class="sxs-lookup"><span data-stu-id="d08e1-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="d08e1-117">Obtenha acesso de usuário a uma instância de sistema integrada de Kit de Desenvolvimento do Azure Stack (ASDK) ou Azure Stack Hub, com:</span><span class="sxs-lookup"><span data-stu-id="d08e1-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="d08e1-118">O [serviço Azure app no provedor de recursos do Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md) instalado.</span><span class="sxs-lookup"><span data-stu-id="d08e1-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="d08e1-119">Você precisa de acesso de operador à sua instância de Hub de Azure Stack ou trabalhar com o administrador para instalar o.</span><span class="sxs-lookup"><span data-stu-id="d08e1-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="d08e1-120">Uma assinatura para uma oferta que fornece a cota de armazenamento e serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d08e1-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="d08e1-121">Você precisa de acesso de operador para criar uma oferta.</span><span class="sxs-lookup"><span data-stu-id="d08e1-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="d08e1-122">Obter acesso a uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="d08e1-123">Se você não tiver uma assinatura do Azure, Inscreva-se para uma [conta de avaliação gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="d08e1-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="d08e1-124">Crie duas entidades de serviço em seu diretório:</span><span class="sxs-lookup"><span data-stu-id="d08e1-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="d08e1-125">Uma configuração para uso com recursos do Azure, com acesso no escopo de assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="d08e1-126">Uma configuração para uso com Azure Stack recursos do Hub, com acesso no escopo de assinatura do Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d08e1-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="d08e1-127">Para saber mais sobre como criar entidades de serviço e autorizar o acesso, consulte [usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="d08e1-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="d08e1-128">Se preferir usar CLI do Azure, consulte [criar uma entidade de serviço do Azure com CLI do Azure](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="d08e1-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="d08e1-129">Implante serviços cognitivas do Azure no Azure ou Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d08e1-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="d08e1-130">Primeiro, [saiba mais sobre os serviços cognitivas](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="d08e1-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="d08e1-131">Em seguida, visite [implantar serviços cognitivas do Azure para Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar serviços cognitivas no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d08e1-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="d08e1-132">Primeiro, você precisa inscrever-se para acessar a versão prévia.</span><span class="sxs-lookup"><span data-stu-id="d08e1-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="d08e1-133">Clone ou baixe um kit de desenvolvimento de ia Visão Personalizada do Azure não configurado.</span><span class="sxs-lookup"><span data-stu-id="d08e1-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="d08e1-134">Para obter detalhes, consulte a [visão de ai devkit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="d08e1-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="d08e1-135">Inscreva-se para uma conta de Power BI.</span><span class="sxs-lookup"><span data-stu-id="d08e1-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="d08e1-136">Um serviço cognitiva do Azure API de Detecção Facial a chave de assinatura e a URL do ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="d08e1-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="d08e1-137">Você pode obter ambos com a avaliação gratuita [experimentar serviços cognitivas](https://azure.microsoft.com/try/cognitive-services/?api=face-api) .</span><span class="sxs-lookup"><span data-stu-id="d08e1-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="d08e1-138">Ou siga as instruções em [criar uma conta de serviços cognitivas](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="d08e1-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="d08e1-139">Instale os seguintes recursos de desenvolvimento:</span><span class="sxs-lookup"><span data-stu-id="d08e1-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="d08e1-140">CLI do Azure 2.0</span><span class="sxs-lookup"><span data-stu-id="d08e1-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="d08e1-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="d08e1-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="d08e1-142">[Carregador](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="d08e1-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="d08e1-143">Você usa o carregador para implantar aplicativos de nuvem usando manifestos de pacote do CNAB que são fornecidos para você.</span><span class="sxs-lookup"><span data-stu-id="d08e1-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="d08e1-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d08e1-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="d08e1-145">Ferramentas de IoT do Azure para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d08e1-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="d08e1-146">Extensão do Python para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="d08e1-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="d08e1-147">Python</span><span class="sxs-lookup"><span data-stu-id="d08e1-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="d08e1-148">Implantar o aplicativo de nuvem híbrida</span><span class="sxs-lookup"><span data-stu-id="d08e1-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="d08e1-149">Primeiro, você usa a CLI carregador para gerar um conjunto de credenciais e, em seguida, implantar o aplicativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="d08e1-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="d08e1-150">Clone ou baixe o código de exemplo da solução de https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="d08e1-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="d08e1-151">O carregador irá gerar um conjunto de credenciais que automatizará a implantação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d08e1-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="d08e1-152">Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="d08e1-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="d08e1-153">Uma entidade de serviço para acessar recursos do Azure, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="d08e1-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d08e1-154">A ID da assinatura para sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="d08e1-155">Uma entidade de serviço para acessar Azure Stack recursos do Hub, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="d08e1-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d08e1-156">A ID da assinatura para sua assinatura do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d08e1-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="d08e1-157">Os serviços cognitivas do Azure API de Detecção Facial URL de ponto de extremidade de recurso e chave.</span><span class="sxs-lookup"><span data-stu-id="d08e1-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="d08e1-158">Execute o processo de geração de credencial carregador e siga os prompts:</span><span class="sxs-lookup"><span data-stu-id="d08e1-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="d08e1-159">O carregador também requer um conjunto de parâmetros a serem executados.</span><span class="sxs-lookup"><span data-stu-id="d08e1-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="d08e1-160">Crie um arquivo de texto de parâmetro e insira os seguintes pares de nome/valor.</span><span class="sxs-lookup"><span data-stu-id="d08e1-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="d08e1-161">Pergunte ao administrador do Hub do Azure Stack se você precisar de assistência com qualquer um dos valores necessários.</span><span class="sxs-lookup"><span data-stu-id="d08e1-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="d08e1-162">O `resource suffix` valor é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="d08e1-163">Ele deve ser uma cadeia de caracteres exclusiva de letras e números, com até 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="d08e1-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="d08e1-164">Salve o arquivo de texto e anote seu caminho.</span><span class="sxs-lookup"><span data-stu-id="d08e1-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="d08e1-165">Agora você está pronto para implantar o aplicativo de nuvem híbrida usando o carregador.</span><span class="sxs-lookup"><span data-stu-id="d08e1-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="d08e1-166">Execute o comando de instalação e assista à medida que os recursos são implantados no Azure e no Hub de Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="d08e1-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="d08e1-167">Após a conclusão da implantação, anote os seguintes valores:</span><span class="sxs-lookup"><span data-stu-id="d08e1-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="d08e1-168">A cadeia de conexão da câmera.</span><span class="sxs-lookup"><span data-stu-id="d08e1-168">The camera's connection string.</span></span>
    - <span data-ttu-id="d08e1-169">A cadeia de conexão da conta de armazenamento de imagens.</span><span class="sxs-lookup"><span data-stu-id="d08e1-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="d08e1-170">Os nomes do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d08e1-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="d08e1-171">Preparar o Visão Personalizada ia DevKit</span><span class="sxs-lookup"><span data-stu-id="d08e1-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="d08e1-172">Em seguida, configure o kit de desenvolvimento do ia Visão Personalizada como mostrado no guia de [início rápido do ia devkit de visão](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="d08e1-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="d08e1-173">Você também configura e testa sua câmera usando a cadeia de conexão fornecida na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="d08e1-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="d08e1-174">Implantar o aplicativo de câmera</span><span class="sxs-lookup"><span data-stu-id="d08e1-174">Deploy the camera app</span></span>

<span data-ttu-id="d08e1-175">Use a CLI do carregador para gerar um conjunto de credenciais e, em seguida, implante o aplicativo de câmera.</span><span class="sxs-lookup"><span data-stu-id="d08e1-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="d08e1-176">O carregador irá gerar um conjunto de credenciais que automatizará a implantação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d08e1-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="d08e1-177">Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="d08e1-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="d08e1-178">Uma entidade de serviço para acessar recursos do Azure, incluindo a ID da entidade de serviço, a chave e o DNS do locatário.</span><span class="sxs-lookup"><span data-stu-id="d08e1-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="d08e1-179">A ID da assinatura para sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="d08e1-180">A cadeia de conexão da conta de armazenamento de imagem fornecida quando você implantou o aplicativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="d08e1-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="d08e1-181">Execute o processo de geração de credencial carregador e siga os prompts:</span><span class="sxs-lookup"><span data-stu-id="d08e1-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="d08e1-182">O carregador também requer um conjunto de parâmetros a serem executados.</span><span class="sxs-lookup"><span data-stu-id="d08e1-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="d08e1-183">Crie um arquivo de texto de parâmetro e insira o texto a seguir.</span><span class="sxs-lookup"><span data-stu-id="d08e1-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="d08e1-184">Pergunte ao administrador do Hub do Azure Stack se você não souber alguns dos valores necessários.</span><span class="sxs-lookup"><span data-stu-id="d08e1-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d08e1-185">O `deployment suffix` valor é usado para garantir que os recursos da implantação tenham nomes exclusivos no Azure.</span><span class="sxs-lookup"><span data-stu-id="d08e1-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="d08e1-186">Ele deve ser uma cadeia de caracteres exclusiva de letras e números, com até 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="d08e1-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="d08e1-187">Salve o arquivo de texto e anote seu caminho.</span><span class="sxs-lookup"><span data-stu-id="d08e1-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="d08e1-188">Agora você está pronto para implantar o aplicativo de câmera usando o carregador.</span><span class="sxs-lookup"><span data-stu-id="d08e1-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="d08e1-189">Execute o comando install e veja como a implantação do IoT Edge é criada.</span><span class="sxs-lookup"><span data-stu-id="d08e1-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="d08e1-190">Verifique se a implantação da câmera está concluída exibindo o feed de câmera em `https://<camera-ip>:3000/` , em que `<camara-ip>` é o endereço IP da câmera.</span><span class="sxs-lookup"><span data-stu-id="d08e1-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="d08e1-191">Esta etapa pode levar até 10 minutos.</span><span class="sxs-lookup"><span data-stu-id="d08e1-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="d08e1-192">Configurar o Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="d08e1-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="d08e1-193">Agora que os dados estão fluindo para Azure Stream Analytics da câmera, precisamos autorizá-lo manualmente para se comunicar com Power BI.</span><span class="sxs-lookup"><span data-stu-id="d08e1-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="d08e1-194">No portal do Azure, abra **todos os recursos**e o trabalho \* \[ yoursuffix \] de footfall de processo\* .</span><span class="sxs-lookup"><span data-stu-id="d08e1-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="d08e1-195">Na seção **Topologia do Trabalho** do painel do trabalho do Stream Analytics, selecione a opção **Saídas**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="d08e1-196">Selecione o coletor de saída de **tráfego de saída** .</span><span class="sxs-lookup"><span data-stu-id="d08e1-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="d08e1-197">Selecione **renovar autorização** e entre em sua conta do Power bi.</span><span class="sxs-lookup"><span data-stu-id="d08e1-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Renovar prompt de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="d08e1-199">Salve as configurações de saída.</span><span class="sxs-lookup"><span data-stu-id="d08e1-199">Save the output settings.</span></span>

6. <span data-ttu-id="d08e1-200">Vá para o painel **visão geral** e selecione **Iniciar** para começar a enviar dados para Power bi.</span><span class="sxs-lookup"><span data-stu-id="d08e1-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="d08e1-201">Selecione **Agora** para a hora de início da saída do trabalho e selecione **Iniciar**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="d08e1-202">Você pode exibir o status do trabalho na barra de notificação.</span><span class="sxs-lookup"><span data-stu-id="d08e1-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="d08e1-203">Criar um painel de Power BI</span><span class="sxs-lookup"><span data-stu-id="d08e1-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="d08e1-204">Quando o trabalho for bem sucedido, vá para [Power bi](https://powerbi.com/) e entre com sua conta corporativa ou de estudante.</span><span class="sxs-lookup"><span data-stu-id="d08e1-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="d08e1-205">Se a consulta do trabalho de Stream Analytics estiver gerando resultados, o conjunto de *footfall* do conjunto de valores que você criou existirá na guia **DataSets** .</span><span class="sxs-lookup"><span data-stu-id="d08e1-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="d08e1-206">No espaço de trabalho Power BI, selecione **+ criar** para criar um novo painel chamado *análise de footfall.*</span><span class="sxs-lookup"><span data-stu-id="d08e1-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="d08e1-207">Na parte superior da janela, escolha **Adicionar bloco**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="d08e1-208">Em seguida, escolha **Fluxo de Dados Personalizado** e **Avançar**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="d08e1-209">Escolha o **footfall-DataSet** em **seus conjuntos de seus DataSets**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="d08e1-210">Selecione **cartão** na lista suspensa **tipo de visualização** e adicione **idade** a **campos**.</span><span class="sxs-lookup"><span data-stu-id="d08e1-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="d08e1-211">Escolha **Avançar** para inserir um nome para o bloco e escolha **Aplicar** para criar o bloco.</span><span class="sxs-lookup"><span data-stu-id="d08e1-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="d08e1-212">Você pode adicionar outros campos e cartões conforme desejado.</span><span class="sxs-lookup"><span data-stu-id="d08e1-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="d08e1-213">Testar sua solução</span><span class="sxs-lookup"><span data-stu-id="d08e1-213">Test Your Solution</span></span>

<span data-ttu-id="d08e1-214">Observe como os dados nos cartões criados no Power BI mudam conforme as pessoas diferentes se movimentam na frente da câmera.</span><span class="sxs-lookup"><span data-stu-id="d08e1-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="d08e1-215">As inferências podem levar até 20 segundos para aparecer uma vez registradas.</span><span class="sxs-lookup"><span data-stu-id="d08e1-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="d08e1-216">Remover sua solução</span><span class="sxs-lookup"><span data-stu-id="d08e1-216">Remove Your Solution</span></span>

<span data-ttu-id="d08e1-217">Se você quiser remover sua solução, execute os comandos a seguir usando carregador, usando os mesmos arquivos de parâmetro que você criou para implantação:</span><span class="sxs-lookup"><span data-stu-id="d08e1-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="d08e1-218">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="d08e1-218">Next steps</span></span>

- <span data-ttu-id="d08e1-219">Saiba mais sobre [considerações de design de aplicativo híbrido]. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="d08e1-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="d08e1-220">Revise e proponha melhorias ao [código para este exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="d08e1-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
