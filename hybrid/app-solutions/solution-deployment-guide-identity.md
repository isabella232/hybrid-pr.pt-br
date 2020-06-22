---
title: Configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack
description: Saiba como configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909847"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="3499a-103">Configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3499a-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="3499a-104">Saiba como configurar uma identidade de nuvem híbrida para seus aplicativos do Azure e do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="3499a-105">Você tem duas opções para conceder acesso aos seus aplicativos no Azure global e no Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="3499a-106">Quando Azure Stack Hub tem uma conexão contínua com a Internet, você pode usar o Azure Active Directory (AD do Azure).</span><span class="sxs-lookup"><span data-stu-id="3499a-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="3499a-107">Quando Azure Stack Hub é desconectado da Internet, você pode usar o AD FS (Serviços Federados do Azure Directory).</span><span class="sxs-lookup"><span data-stu-id="3499a-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="3499a-108">Você usa entidades de serviço para conceder acesso aos seus aplicativos de Hub de Azure Stack para implantação ou configuração usando o Azure Resource Manager no Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="3499a-109">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="3499a-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3499a-110">Estabelecer uma identidade híbrida no Azure global e no Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3499a-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="3499a-111">Recupere um token para acessar a API do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="3499a-112">Você deve ter Azure Stack permissões de operador de Hub para as etapas nesta solução.</span><span class="sxs-lookup"><span data-stu-id="3499a-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="3499a-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3499a-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3499a-114">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="3499a-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3499a-115">Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="3499a-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3499a-116">O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="3499a-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3499a-117">As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="3499a-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="3499a-118">Criar uma entidade de serviço para o Azure AD no portal</span><span class="sxs-lookup"><span data-stu-id="3499a-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="3499a-119">Se você implantou o Hub de Azure Stack usando o Azure AD como o armazenamento de identidade, você pode criar entidades de serviço da mesma forma que faz para o Azure.</span><span class="sxs-lookup"><span data-stu-id="3499a-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="3499a-120">[Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) mostra como executar as etapas por meio do Portal.</span><span class="sxs-lookup"><span data-stu-id="3499a-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="3499a-121">Verifique se você tem as [permissões necessárias do Azure ad](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="3499a-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="3499a-122">Criar uma entidade de serviço para AD FS usando o PowerShell</span><span class="sxs-lookup"><span data-stu-id="3499a-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="3499a-123">Se você implantou Azure Stack Hub com AD FS, você pode usar o PowerShell para criar uma entidade de serviço, atribuir uma função de acesso e entrar do PowerShell usando essa identidade.</span><span class="sxs-lookup"><span data-stu-id="3499a-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="3499a-124">[Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) mostra como executar as etapas necessárias usando o PowerShell.</span><span class="sxs-lookup"><span data-stu-id="3499a-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="3499a-125">Usando a API do hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3499a-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="3499a-126">A solução de [API do hub Azure Stack](/azure-stack/user/azure-stack-rest-api-use.md) orienta você pelo processo de recuperação de um token para acessar a API do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="3499a-127">Conectar-se ao Hub de Azure Stack usando o PowerShell</span><span class="sxs-lookup"><span data-stu-id="3499a-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="3499a-128">O início rápido [para começar a executar o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) orienta você pelas etapas necessárias para instalar o Azure PowerShell e conectar-se à instalação do Hub do Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3499a-129">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="3499a-129">Prerequisites</span></span>

<span data-ttu-id="3499a-130">Você precisa de uma instalação de Hub de Azure Stack conectada ao Azure AD com uma assinatura que você pode acessar.</span><span class="sxs-lookup"><span data-stu-id="3499a-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="3499a-131">Se você não tiver uma instalação de Hub de Azure Stack, poderá usar estas instruções para configurar um [Kit de desenvolvimento do Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="3499a-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="3499a-132">Conectar-se ao Hub de Azure Stack usando código</span><span class="sxs-lookup"><span data-stu-id="3499a-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="3499a-133">Para se conectar ao Hub de Azure Stack usando código, use a API de pontos de extremidade de Azure Resource Manager para obter os pontos de extremidade de autenticação e de grafo para a instalação do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="3499a-134">Em seguida, autentique usando solicitações REST.</span><span class="sxs-lookup"><span data-stu-id="3499a-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="3499a-135">Você pode encontrar um aplicativo cliente de exemplo no [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="3499a-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="3499a-136">A menos que o SDK do Azure para sua linguagem de escolha dê suporte a perfis de API do Azure, o SDK pode não funcionar com o Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3499a-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="3499a-137">Para saber mais sobre os perfis de API do Azure, consulte o artigo [gerenciar perfis de versão da API](/azure-stack/user/azure-stack-version-profiles.md) .</span><span class="sxs-lookup"><span data-stu-id="3499a-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3499a-138">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="3499a-138">Next steps</span></span>

- <span data-ttu-id="3499a-139">Para saber mais sobre como a identidade é manipulada no Hub Azure Stack, consulte [arquitetura de identidade para Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="3499a-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="3499a-140">Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="3499a-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
