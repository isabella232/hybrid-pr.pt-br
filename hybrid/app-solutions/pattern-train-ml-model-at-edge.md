---
title: Treinar o modelo de aprendizado de máquina no padrão de borda
description: Saiba como fazer treinamento de modelo do Machine Learning na borda com o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909816"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="b6aff-103">Treinar o modelo de aprendizado de máquina no padrão de borda</span><span class="sxs-lookup"><span data-stu-id="b6aff-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="b6aff-104">Gere modelos de ML (aprendizado de máquina portátil) a partir de dados que só existem localmente.</span><span class="sxs-lookup"><span data-stu-id="b6aff-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b6aff-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="b6aff-105">Context and problem</span></span>

<span data-ttu-id="b6aff-106">Muitas organizações gostariam de revelar informações de seus dados locais ou herdados usando as ferramentas que seus cientistas de dados entendem.</span><span class="sxs-lookup"><span data-stu-id="b6aff-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="b6aff-107">O [Azure Machine Learning](/azure/machine-learning/) fornece ferramentas nativas de nuvem para treinar, ajustar e implantar modelos de aprendizado profundo e ml.</span><span class="sxs-lookup"><span data-stu-id="b6aff-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="b6aff-108">No entanto, alguns dados são muito grandes enviar para a nuvem ou não podem ser enviados para a nuvem por motivos regulatórios.</span><span class="sxs-lookup"><span data-stu-id="b6aff-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="b6aff-109">Usando esse padrão, os cientistas de dados podem usar Azure Machine Learning para treinar modelos usando dados e computação locais.</span><span class="sxs-lookup"><span data-stu-id="b6aff-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="b6aff-110">Solução</span><span class="sxs-lookup"><span data-stu-id="b6aff-110">Solution</span></span>

<span data-ttu-id="b6aff-111">O treinamento no padrão de borda usa uma VM (máquina virtual) em execução no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6aff-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="b6aff-112">A VM é registrada como um destino de computação no Azure ML, permitindo que ele acesse dados somente disponíveis localmente.</span><span class="sxs-lookup"><span data-stu-id="b6aff-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="b6aff-113">Nesse caso, os dados são armazenados no armazenamento de BLOBs do Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6aff-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="b6aff-114">Depois que o modelo é treinado, ele é registrado com o Azure ML, em contêiner e adicionado a um registro de contêiner do Azure para implantação.</span><span class="sxs-lookup"><span data-stu-id="b6aff-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="b6aff-115">Para essa iteração do padrão, a VM de treinamento do Hub Azure Stack deve estar acessível pela Internet pública.</span><span class="sxs-lookup"><span data-stu-id="b6aff-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="b6aff-116">[![Treinar modelo ML na arquitetura de borda](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="b6aff-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="b6aff-117">Veja como o padrão funciona:</span><span class="sxs-lookup"><span data-stu-id="b6aff-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="b6aff-118">A VM do hub de Azure Stack é implantada e registrada como um destino de computação com o Azure ML.</span><span class="sxs-lookup"><span data-stu-id="b6aff-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="b6aff-119">Um experimento é criado no Azure ML que usa a VM do Hub Azure Stack como um destino de computação.</span><span class="sxs-lookup"><span data-stu-id="b6aff-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="b6aff-120">Depois que o modelo é treinado, ele é registrado e em contêiner.</span><span class="sxs-lookup"><span data-stu-id="b6aff-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="b6aff-121">O modelo agora pode ser implantado em locais que estejam no local ou na nuvem.</span><span class="sxs-lookup"><span data-stu-id="b6aff-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="b6aff-122">Componentes</span><span class="sxs-lookup"><span data-stu-id="b6aff-122">Components</span></span>

<span data-ttu-id="b6aff-123">Essa solução usa os seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="b6aff-123">This solution uses the following components:</span></span>

| <span data-ttu-id="b6aff-124">Camada</span><span class="sxs-lookup"><span data-stu-id="b6aff-124">Layer</span></span> | <span data-ttu-id="b6aff-125">Componente</span><span class="sxs-lookup"><span data-stu-id="b6aff-125">Component</span></span> | <span data-ttu-id="b6aff-126">Descrição</span><span class="sxs-lookup"><span data-stu-id="b6aff-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="b6aff-127">Azure</span><span class="sxs-lookup"><span data-stu-id="b6aff-127">Azure</span></span> | <span data-ttu-id="b6aff-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="b6aff-128">Azure Machine Learning</span></span> | <span data-ttu-id="b6aff-129">[Azure Machine Learning](/azure/machine-learning/) orquestra o treinamento do modelo ml.</span><span class="sxs-lookup"><span data-stu-id="b6aff-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="b6aff-130">Registro de Contêiner do Azure</span><span class="sxs-lookup"><span data-stu-id="b6aff-130">Azure Container Registry</span></span> | <span data-ttu-id="b6aff-131">O Azure ML empacota o modelo em um contêiner e o armazena em um [registro de contêiner do Azure](/azure/container-registry/) para implantação.</span><span class="sxs-lookup"><span data-stu-id="b6aff-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="b6aff-132">Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="b6aff-132">Azure Stack Hub</span></span> | <span data-ttu-id="b6aff-133">Serviço de Aplicativo</span><span class="sxs-lookup"><span data-stu-id="b6aff-133">App Service</span></span> | <span data-ttu-id="b6aff-134">[Azure Stack Hub com o serviço de aplicativo](/azure-stack/operator/azure-stack-app-service-overview) fornece a base para os componentes na borda.</span><span class="sxs-lookup"><span data-stu-id="b6aff-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="b6aff-135">Computação</span><span class="sxs-lookup"><span data-stu-id="b6aff-135">Compute</span></span> | <span data-ttu-id="b6aff-136">Uma VM de Hub de Azure Stack que executa o Ubuntu com o Docker é usada para treinar o modelo de ML.</span><span class="sxs-lookup"><span data-stu-id="b6aff-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="b6aff-137">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="b6aff-137">Storage</span></span> | <span data-ttu-id="b6aff-138">Os dados privados podem ser hospedados no armazenamento de blobs de Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6aff-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="b6aff-139">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="b6aff-139">Issues and considerations</span></span>

<span data-ttu-id="b6aff-140">Considere os seguintes pontos ao decidir como implementar essa solução:</span><span class="sxs-lookup"><span data-stu-id="b6aff-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="b6aff-141">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="b6aff-141">Scalability</span></span>

<span data-ttu-id="b6aff-142">Para habilitar essa solução para escala, você precisará criar uma VM de tamanho adequado no Hub Azure Stack para treinamento.</span><span class="sxs-lookup"><span data-stu-id="b6aff-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="b6aff-143">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="b6aff-143">Availability</span></span>

<span data-ttu-id="b6aff-144">Verifique se os scripts de treinamento e a VM do hub de Azure Stack têm acesso aos dados locais usados para treinamento.</span><span class="sxs-lookup"><span data-stu-id="b6aff-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="b6aff-145">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="b6aff-145">Manageability</span></span>

<span data-ttu-id="b6aff-146">Verifique se os modelos e experimentos estão adequadamente registrados, com versão e marcados para evitar confusão durante a implantação do modelo.</span><span class="sxs-lookup"><span data-stu-id="b6aff-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="b6aff-147">Segurança</span><span class="sxs-lookup"><span data-stu-id="b6aff-147">Security</span></span>

<span data-ttu-id="b6aff-148">Esse padrão permite que o Azure ML acesse dados confidenciais possíveis no local.</span><span class="sxs-lookup"><span data-stu-id="b6aff-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="b6aff-149">Verifique se a conta usada para SSH em Azure Stack VM do Hub tem uma senha forte e os scripts de treinamento não preservam nem carregam dados na nuvem.</span><span class="sxs-lookup"><span data-stu-id="b6aff-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b6aff-150">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="b6aff-150">Next steps</span></span>

<span data-ttu-id="b6aff-151">Para saber mais sobre os tópicos apresentados neste artigo:</span><span class="sxs-lookup"><span data-stu-id="b6aff-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="b6aff-152">Consulte a [documentação do Azure Machine Learning](/azure/machine-learning) para obter uma visão geral do ml e tópicos relacionados.</span><span class="sxs-lookup"><span data-stu-id="b6aff-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="b6aff-153">Consulte [registro de contêiner do Azure](/azure/container-registry/) para saber como criar, armazenar e gerenciar imagens para implantações de contêiner.</span><span class="sxs-lookup"><span data-stu-id="b6aff-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="b6aff-154">Consulte [serviço de aplicativo no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) para saber mais sobre o provedor de recursos e como implantá-lo.</span><span class="sxs-lookup"><span data-stu-id="b6aff-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="b6aff-155">Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e para obter respostas adicionais.</span><span class="sxs-lookup"><span data-stu-id="b6aff-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="b6aff-156">Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="b6aff-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="b6aff-157">Quando você estiver pronto para testar o exemplo de solução, continue com o [modelo treinar ml no guia de implantação do Edge](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="b6aff-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="b6aff-158">O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.</span><span class="sxs-lookup"><span data-stu-id="b6aff-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
