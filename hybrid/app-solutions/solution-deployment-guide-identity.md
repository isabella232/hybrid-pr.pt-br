---
title: Configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub
description: Saiba como configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895339"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="51cbd-103">Configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="51cbd-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="51cbd-104">Saiba como configurar uma identidade de nuvem híbrida para seus aplicativos do Azure e do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="51cbd-105">Você tem duas opções para conceder acesso aos seus aplicativos tanto no Azure global quanto no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="51cbd-106">Quando o Azure Stack Hub tiver uma conexão contínua com a Internet, você poderá usar o Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="51cbd-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="51cbd-107">Quando o Azure Stack Hub estiver desconectado da Internet, use o AD FS (Serviços de Federação do Azure Directory).</span><span class="sxs-lookup"><span data-stu-id="51cbd-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="51cbd-108">Você pode usar entidades de serviço para conceder acesso aos seus aplicativos do Azure Stack Hub para fins de implantação ou configuração usando o Azure Resource Manager no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="51cbd-109">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="51cbd-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="51cbd-110">Estabelecer uma identidade híbrida no Azure global e no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="51cbd-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="51cbd-111">Recuperar um token para acessar a API do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="51cbd-112">Você precisa ter permissões de operador do Azure Stack Hub para executar as etapas desta solução.</span><span class="sxs-lookup"><span data-stu-id="51cbd-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="51cbd-113">![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="51cbd-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="51cbd-114">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="51cbd-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="51cbd-115">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="51cbd-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="51cbd-116">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="51cbd-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="51cbd-117">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="51cbd-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="51cbd-118">Crie uma entidade de serviço para o Azure AD no portal</span><span class="sxs-lookup"><span data-stu-id="51cbd-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="51cbd-119">Se você implantou o Azure Stack Hub usando o Azure AD como armazenamento de identidade, pode criar entidades de serviço da mesma forma que faz para o Azure.</span><span class="sxs-lookup"><span data-stu-id="51cbd-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="51cbd-120">[Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) mostra como executar as etapas por meio do portal.</span><span class="sxs-lookup"><span data-stu-id="51cbd-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="51cbd-121">Verifique se você tem as [permissões necessárias do Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="51cbd-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="51cbd-122">Crie uma entidade de serviço para o AD FS usando o PowerShell</span><span class="sxs-lookup"><span data-stu-id="51cbd-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="51cbd-123">Se você implantou o Azure Stack Hub com o AD FS, pode usar o PowerShell para criar uma entidade de serviço, atribuir uma função de acesso e entrar a partir do PowerShell usando essa identidade.</span><span class="sxs-lookup"><span data-stu-id="51cbd-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="51cbd-124">[Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) mostra como executar as etapas necessárias usando o PowerShell.</span><span class="sxs-lookup"><span data-stu-id="51cbd-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="51cbd-125">Usando a API do Azure Stack Hub API</span><span class="sxs-lookup"><span data-stu-id="51cbd-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="51cbd-126">A solução [API do Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use) guiará você pelo processo de recuperação de um token para acessar a API do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="51cbd-127">Conecte-se com o Azure Stack Hub usando o PowerShell</span><span class="sxs-lookup"><span data-stu-id="51cbd-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="51cbd-128">O início rápido [para começar a executar o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) guiará você pelas etapas necessárias para instalar o Azure PowerShell e conectar-se à instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="51cbd-129">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="51cbd-129">Prerequisites</span></span>

<span data-ttu-id="51cbd-130">É preciso uma instalação do Azure Stack Hub conectada ao Azure AD com uma assinatura que você possa acessar.</span><span class="sxs-lookup"><span data-stu-id="51cbd-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="51cbd-131">Se você não tiver uma instalação do Azure Stack Hub, poderá usar estas instruções para configurar um [ASDK (Kit de Desenvolvimento do Azure Stack)](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="51cbd-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="51cbd-132">Conecte-se com o Azure Stack Hub usando código</span><span class="sxs-lookup"><span data-stu-id="51cbd-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="51cbd-133">Para conectar-se com o Azure Stack Hub usando código, empregue a API de pontos de extremidade do Azure Resource Manager para obter os pontos de extremidade de autenticação e de grafo de sua instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="51cbd-134">Em seguida, autentique-se usando solicitações REST.</span><span class="sxs-lookup"><span data-stu-id="51cbd-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="51cbd-135">Você pode encontrar um exemplo de aplicativo cliente no[GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="51cbd-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="51cbd-136">A menos que o SDK do Azure para sua linguagem de escolha dê suporte a perfis de API do Azure, o SDK pode não funcionar com o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="51cbd-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="51cbd-137">Para saber mais sobre os perfis de API do Azure, confira o artigo [gerenciar perfis de versão de API](/azure-stack/user/azure-stack-version-profiles).</span><span class="sxs-lookup"><span data-stu-id="51cbd-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="51cbd-138">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="51cbd-138">Next steps</span></span>

- <span data-ttu-id="51cbd-139">Para saber mais sobre como a identidade é manipulada no Azure Stack Hub, confira [Arquitetura de identidade do Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span><span class="sxs-lookup"><span data-stu-id="51cbd-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="51cbd-140">Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="51cbd-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
