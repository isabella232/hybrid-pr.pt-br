---
title: Implantar um aplicativo que escale entre nuvem no Azure e no Hub de Azure Stack
description: Saiba como implantar um aplicativo que dimensiona entre nuvem no Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909877"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="b6c28-103">Implantar um aplicativo que escale entre nuvem usando o Azure e o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="b6c28-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="b6c28-104">Saiba como criar uma solução de nuvem cruzada para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado pelo Hub Azure Stack para um aplicativo Web hospedado do Azure com dimensionamento automático por meio do Gerenciador de tráfego.</span><span class="sxs-lookup"><span data-stu-id="b6c28-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="b6c28-105">Esse processo garante um utilitário de nuvem flexível e escalonável quando sob carga.</span><span class="sxs-lookup"><span data-stu-id="b6c28-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="b6c28-106">Com esse padrão, seu locatário pode não estar pronto para executar seu aplicativo na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="b6c28-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="b6c28-107">No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="b6c28-108">Seu locatário pode fazer uso da elasticidade da nuvem pública com sua solução local.</span><span class="sxs-lookup"><span data-stu-id="b6c28-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="b6c28-109">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="b6c28-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="b6c28-110">Crie um aplicativo Web de vários nós.</span><span class="sxs-lookup"><span data-stu-id="b6c28-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="b6c28-111">Configure e gerencie o processo de implantação contínua (CD).</span><span class="sxs-lookup"><span data-stu-id="b6c28-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="b6c28-112">Publique o aplicativo Web no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="b6c28-113">Crie uma versão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-113">Create a release.</span></span>
> - <span data-ttu-id="b6c28-114">Aprenda a monitorar e acompanhar suas implantações.</span><span class="sxs-lookup"><span data-stu-id="b6c28-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="b6c28-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="b6c28-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="b6c28-116">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="b6c28-117">Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="b6c28-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="b6c28-118">O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="b6c28-119">As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="b6c28-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b6c28-120">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="b6c28-120">Prerequisites</span></span>

- <span data-ttu-id="b6c28-121">Assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-121">Azure subscription.</span></span> <span data-ttu-id="b6c28-122">Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="b6c28-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="b6c28-123">Um sistema integrado de Hub Azure Stack ou implantação de Kit de Desenvolvimento do Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="b6c28-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="b6c28-124">Para obter instruções sobre como instalar Azure Stack Hub, consulte [instalar o ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="b6c28-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="b6c28-125">Para um script de automação pós-implantação do ASDK, acesse:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="b6c28-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="b6c28-126">Essa instalação pode exigir algumas horas para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="b6c28-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="b6c28-127">Implante serviços de PaaS do [serviço de aplicativo](/azure-stack/operator/azure-stack-app-service-deploy.md) para Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b6c28-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="b6c28-128">[Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) dentro do ambiente de Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="b6c28-129">[Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro do ambiente de Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="b6c28-130">Crie um aplicativo Web dentro da assinatura de locatário.</span><span class="sxs-lookup"><span data-stu-id="b6c28-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="b6c28-131">Anote a nova URL do aplicativo Web para uso posterior.</span><span class="sxs-lookup"><span data-stu-id="b6c28-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="b6c28-132">Implante Azure Pipelines máquina virtual (VM) na assinatura do locatário.</span><span class="sxs-lookup"><span data-stu-id="b6c28-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="b6c28-133">A VM do Windows Server 2016 com o .NET 3,5 é necessária.</span><span class="sxs-lookup"><span data-stu-id="b6c28-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="b6c28-134">Essa VM será criada na assinatura de locatário no Hub de Azure Stack como o agente de compilação particular.</span><span class="sxs-lookup"><span data-stu-id="b6c28-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="b6c28-135">O [Windows Server 2016 com a imagem de VM do SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Marketplace do Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="b6c28-136">Se essa imagem não estiver disponível, trabalhe com um operador de Hub de Azure Stack para garantir que ele seja adicionado ao ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b6c28-137">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="b6c28-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="b6c28-138">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="b6c28-138">Scalability</span></span>

<span data-ttu-id="b6c28-139">O principal componente do dimensionamento entre nuvem é a capacidade de fornecer dimensionamento imediato e sob demanda entre a infraestrutura de nuvem pública e local, fornecendo um serviço consistente e confiável.</span><span class="sxs-lookup"><span data-stu-id="b6c28-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="b6c28-140">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="b6c28-140">Availability</span></span>

<span data-ttu-id="b6c28-141">Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.</span><span class="sxs-lookup"><span data-stu-id="b6c28-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="b6c28-142">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="b6c28-142">Manageability</span></span>

<span data-ttu-id="b6c28-143">A solução de nuvem cruzada garante o gerenciamento contínuo e a interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="b6c28-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="b6c28-144">O PowerShell é recomendado para gerenciamento de plataforma cruzada.</span><span class="sxs-lookup"><span data-stu-id="b6c28-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="b6c28-145">Escala entre nuvens</span><span class="sxs-lookup"><span data-stu-id="b6c28-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="b6c28-146">Obter um domínio personalizado e configurar o DNS</span><span class="sxs-lookup"><span data-stu-id="b6c28-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="b6c28-147">Atualize o arquivo de zona DNS para o domínio.</span><span class="sxs-lookup"><span data-stu-id="b6c28-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="b6c28-148">O AD do Azure verificará a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="b6c28-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="b6c28-149">Use o [DNS do Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) para registros DNS do Azure/Office 365/externos no Azure ou adicione a entrada DNS em [um registrador de DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="b6c28-149">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="b6c28-150">Registre um domínio personalizado com um registrador público.</span><span class="sxs-lookup"><span data-stu-id="b6c28-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="b6c28-151">Entre no registrador de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="b6c28-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="b6c28-152">Um administrador aprovado pode ser necessário para fazer atualizações de DNS.</span><span class="sxs-lookup"><span data-stu-id="b6c28-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="b6c28-153">Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b6c28-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="b6c28-154">(A entrada DNS não afetará o roteamento de email ou os comportamentos de hospedagem na Web.)</span><span class="sxs-lookup"><span data-stu-id="b6c28-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="b6c28-155">Criar um aplicativo Web de vários nós padrão no Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="b6c28-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="b6c28-156">Configure a integração contínua híbrida e a CI/CD (implantação contínua) para implantar aplicativos Web no Azure e no Hub de Azure Stack e para enviar alterações por push a ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="b6c28-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="b6c28-157">O Hub de Azure Stack com imagens apropriadas agregadas para execução (Windows Server e SQL) e implantação do serviço de aplicativo são necessários.</span><span class="sxs-lookup"><span data-stu-id="b6c28-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="b6c28-158">Para obter mais informações, examine os pré-requisitos de documentação do serviço [de aplicativo para implantar o serviço de aplicativo no Hub de Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="b6c28-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="b6c28-159">Adicionar código a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="b6c28-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="b6c28-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="b6c28-160">Azure Repos</span></span>

1. <span data-ttu-id="b6c28-161">Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="b6c28-162">CI/CD híbrido pode ser aplicado tanto ao código do aplicativo quanto ao código de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="b6c28-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="b6c28-163">Use [modelos de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privada e hospedado.</span><span class="sxs-lookup"><span data-stu-id="b6c28-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conectar-se a um projeto no Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="b6c28-165">**Clone o repositório** criando e abrindo o aplicativo Web padrão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonar repositório no aplicativo Web do Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="b6c28-167">Criar implantação de aplicativo Web independente para serviços de aplicativos em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="b6c28-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="b6c28-168">Edite o arquivo **WebApplication. csproj** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="b6c28-169">Selecione `Runtimeidentifier` e adicione `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="b6c28-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="b6c28-170">(Consulte a documentação [de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)</span><span class="sxs-lookup"><span data-stu-id="b6c28-170">(See [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar arquivo de projeto do aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="b6c28-172">Faça check-in no código para Azure Repos usando Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="b6c28-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="b6c28-173">Confirme se o código do aplicativo foi verificado Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="b6c28-174">Criar a definição de compilação</span><span class="sxs-lookup"><span data-stu-id="b6c28-174">Create the build definition</span></span>

1. <span data-ttu-id="b6c28-175">Entre no Azure Pipelines para confirmar a capacidade de criar definições de compilação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="b6c28-176">Adicione **-r win10-código x64** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="b6c28-177">Essa adição é necessária para disparar uma implantação independente com o .NET Core.</span><span class="sxs-lookup"><span data-stu-id="b6c28-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Adicionar código ao aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="b6c28-179">Execute a compilação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-179">Run the build.</span></span> <span data-ttu-id="b6c28-180">O processo de [compilação de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que são executados no Azure e Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b6c28-180">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="b6c28-181">Usar um agente hospedado do Azure</span><span class="sxs-lookup"><span data-stu-id="b6c28-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="b6c28-182">Usar um agente de compilação hospedado no Azure Pipelines é uma opção conveniente para criar e implantar aplicativos Web.</span><span class="sxs-lookup"><span data-stu-id="b6c28-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="b6c28-183">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="b6c28-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="b6c28-184">Gerenciar e configurar o processo de CD</span><span class="sxs-lookup"><span data-stu-id="b6c28-184">Manage and configure the CD process</span></span>

<span data-ttu-id="b6c28-185">Azure Pipelines e Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável para versões para vários ambientes, como desenvolvimento, preparo, QA e ambientes de produção; incluindo a necessidade de aprovações em estágios específicos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="b6c28-186">Criar definição de versão</span><span class="sxs-lookup"><span data-stu-id="b6c28-186">Create release definition</span></span>

1. <span data-ttu-id="b6c28-187">Selecione o botão de **adição** para adicionar uma nova versão na guia **versões** na seção **Build e versão** do Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="b6c28-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="b6c28-189">Aplique o modelo de implantação do serviço de Azure App.</span><span class="sxs-lookup"><span data-stu-id="b6c28-189">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar Azure App modelo de implantação do serviço](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="b6c28-191">Em **Adicionar artefato**, adicione o artefato para o aplicativo de compilação na nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Adicionar artefato à compilação na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="b6c28-193">Na guia pipeline, selecione a **fase,** o link de tarefa do ambiente e defina os valores de ambiente de nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir valores de ambiente de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="b6c28-195">Defina o **nome do ambiente** e selecione a **assinatura do Azure** para o ponto de extremidade de nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecione a assinatura do Azure para o ponto de extremidade de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="b6c28-197">Em **nome do serviço de aplicativo**, defina o nome do serviço de aplicativo do Azure necessário.</span><span class="sxs-lookup"><span data-stu-id="b6c28-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir nome do serviço de aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="b6c28-199">Insira "Hosted VS2017" na **fila do agente** para o ambiente hospedado na nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c28-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Definir fila do agente para o ambiente hospedado na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="b6c28-201">No menu implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="b6c28-202">Selecione **OK** para a **pasta local**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-202">Select **OK** to **folder location**.</span></span>
  
      ![Selecionar pacote ou pasta para o ambiente de serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecionar pacote ou pasta para o ambiente de serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="b6c28-205">Salve todas as alterações e volte para o **pipeline de liberação**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Salvar alterações no pipeline de lançamento](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="b6c28-207">Adicione um novo artefato selecionando a compilação para o aplicativo de Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Adicionar novo artefato para o aplicativo de Hub de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="b6c28-209">Adicione mais um ambiente aplicando a implantação do serviço de Azure App.</span><span class="sxs-lookup"><span data-stu-id="b6c28-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Adicionar ambiente à implantação do serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="b6c28-211">Nomeie o novo ambiente "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="b6c28-211">Name the new environment "Azure Stack".</span></span>

    ![Ambiente de nome na implantação do serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="b6c28-213">Localize o ambiente Azure Stack na guia **tarefa** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Ambiente de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="b6c28-215">Selecione a assinatura para o ponto de extremidade de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Selecione a assinatura para o ponto de extremidade Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="b6c28-217">Defina o nome do aplicativo Web Azure Stack como o nome do serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="b6c28-218">![Definir Azure Stack nome do aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="b6c28-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="b6c28-219">Selecione o agente de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-219">Select the Azure Stack agent.</span></span>

    ![Selecione o agente de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="b6c28-221">Na seção implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="b6c28-222">Selecione **OK** para a pasta local.</span><span class="sxs-lookup"><span data-stu-id="b6c28-222">Select **OK** to folder location.</span></span>

    ![Selecionar pasta para implantação de serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecionar pasta para implantação de serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="b6c28-225">Na guia variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , defina seu valor como **true**e escopo como Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Adicionar variável à implantação de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="b6c28-227">Selecione o ícone de gatilho de implantação **contínua** em ambos os artefatos e habilite o gatilho de implantação **continua** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selecionar gatilho de implantação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="b6c28-229">Selecione o ícone condições de **pré-implantação** no ambiente de Azure Stack e defina o gatilho para **após a liberação.**</span><span class="sxs-lookup"><span data-stu-id="b6c28-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Selecionar condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="b6c28-231">Salve todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="b6c28-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="b6c28-232">Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) ao criar uma definição de versão a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-232">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="b6c28-233">Essas configurações não podem ser modificadas nas configurações da tarefa; em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.</span><span class="sxs-lookup"><span data-stu-id="b6c28-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="b6c28-234">Publicar no Hub de Azure Stack por meio do Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b6c28-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="b6c28-235">Criando pontos de extremidade, um Azure DevOps Services Build pode implantar aplicativos de serviço do Azure para Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b6c28-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="b6c28-236">Azure Pipelines se conecta ao agente de compilação, que se conecta ao Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="b6c28-237">Entre no Azure DevOps Services e vá para a página de configurações do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="b6c28-238">Em **Configurações**, selecione **Segurança**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="b6c28-239">Em **grupos do VSTS**, selecione **criadores de ponto de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="b6c28-240">Na guia **Membros**, selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="b6c28-241">Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.</span><span class="sxs-lookup"><span data-stu-id="b6c28-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="b6c28-242">Selecione **Salvar alterações**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="b6c28-243">Na lista de **grupos do VSTS** , selecione administradores de ponto de **extremidade**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="b6c28-244">Na guia **Membros**, selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="b6c28-245">Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.</span><span class="sxs-lookup"><span data-stu-id="b6c28-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="b6c28-246">Selecione **Salvar alterações**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-246">Select **Save changes**.</span></span>

<span data-ttu-id="b6c28-247">Agora que as informações do ponto de extremidade existem, o Azure Pipelines para Azure Stack conexão de Hub está pronto para uso.</span><span class="sxs-lookup"><span data-stu-id="b6c28-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="b6c28-248">O agente de compilação no Hub Azure Stack Obtém instruções de Azure Pipelines e, em seguida, o agente transmite informações de ponto de extremidade para comunicação com o Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="b6c28-249">Desenvolver a compilação do aplicativo</span><span class="sxs-lookup"><span data-stu-id="b6c28-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="b6c28-250">O Hub de Azure Stack com imagens apropriadas agregadas para execução (Windows Server e SQL) e implantação do serviço de aplicativo são necessários.</span><span class="sxs-lookup"><span data-stu-id="b6c28-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="b6c28-251">Para obter mais informações, consulte [pré-requisitos para implantar o serviço de aplicativo no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="b6c28-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="b6c28-252">Use [modelos de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) como código do aplicativo web de Azure Repos para implantar em ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="b6c28-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="b6c28-253">Adicionar código a um projeto Azure Repos</span><span class="sxs-lookup"><span data-stu-id="b6c28-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="b6c28-254">Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="b6c28-255">**Clone o repositório** criando e abrindo o aplicativo Web padrão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="b6c28-256">Criar implantação de aplicativo Web independente para serviços de aplicativos em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="b6c28-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="b6c28-257">Edite o arquivo **WebApplication. csproj** : selecione `Runtimeidentifier` e adicione `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="b6c28-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="b6c28-258">Para obter mais informações, consulte a documentação de [implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="b6c28-258">For more information, see [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="b6c28-259">Use Team Explorer para verificar o código em Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="b6c28-260">Confirme se o código do aplicativo foi verificado em Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="b6c28-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="b6c28-261">Criar a definição de compilação</span><span class="sxs-lookup"><span data-stu-id="b6c28-261">Create the build definition</span></span>

1. <span data-ttu-id="b6c28-262">Entre no Azure Pipelines com uma conta que possa criar uma definição de compilação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="b6c28-263">Vá para a página **criar aplicativo Web** para o projeto.</span><span class="sxs-lookup"><span data-stu-id="b6c28-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="b6c28-264">Em **argumentos**, adicione o código **win10-x64** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="b6c28-265">Essa adição é necessária para disparar uma implantação independente com o .NET Core.</span><span class="sxs-lookup"><span data-stu-id="b6c28-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="b6c28-266">Execute a compilação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-266">Run the build.</span></span> <span data-ttu-id="b6c28-267">O processo de [compilação de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-267">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="b6c28-268">Usar um agente de compilação hospedado do Azure</span><span class="sxs-lookup"><span data-stu-id="b6c28-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="b6c28-269">Usar um agente de compilação hospedado no Azure Pipelines é uma opção conveniente para criar e implantar aplicativos Web.</span><span class="sxs-lookup"><span data-stu-id="b6c28-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="b6c28-270">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="b6c28-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="b6c28-271">Configurar o processo de implantação contínua (CD)</span><span class="sxs-lookup"><span data-stu-id="b6c28-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="b6c28-272">Azure Pipelines e Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável para versões para vários ambientes, como desenvolvimento, preparo, controle de qualidade (QA) e produção.</span><span class="sxs-lookup"><span data-stu-id="b6c28-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="b6c28-273">Esse processo pode incluir a exigência de aprovações em estágios específicos do ciclo de vida do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="b6c28-274">Criar definição de versão</span><span class="sxs-lookup"><span data-stu-id="b6c28-274">Create release definition</span></span>

<span data-ttu-id="b6c28-275">A criação de uma definição de versão é a etapa final no processo de compilação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="b6c28-276">Esta definição de versão é usada para criar uma versão e implantar uma compilação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="b6c28-277">Entre no Azure Pipelines e vá para **Compilar e liberar** para o projeto.</span><span class="sxs-lookup"><span data-stu-id="b6c28-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="b6c28-278">Na guia **versões** , selecione **[+]** e escolha **criar definição de versão**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="b6c28-279">Em **selecionar um modelo**, escolha **Azure app implantação de serviço**e, em seguida, selecione **aplicar**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="b6c28-280">Em **Adicionar artefato**, na **origem (definição de compilação)**, selecione o aplicativo Azure cloud Build.</span><span class="sxs-lookup"><span data-stu-id="b6c28-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="b6c28-281">Na guia **pipeline** , selecione a **fase 1**, 1 link de **tarefa** para **Exibir tarefas de ambiente**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="b6c28-282">Na guia **tarefas** , insira Azure como o **nome do ambiente** e selecione o AzureCloud Traders-Web EP na lista de **assinaturas do Azure** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="b6c28-283">Insira o **nome do serviço de aplicativo do Azure**, que está `northwindtraders` na próxima captura de tela.</span><span class="sxs-lookup"><span data-stu-id="b6c28-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="b6c28-284">Para a fase do agente, selecione **Hosted VS2017** na lista de **filas do agente** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="b6c28-285">Em **implantar Azure app serviço**, selecione o **pacote ou a pasta** válida para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="b6c28-286">Em **Selecionar arquivo ou pasta**, selecione **OK** para o **local**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="b6c28-287">Salve todas as alterações e volte para o **pipeline**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="b6c28-288">Na guia **pipeline** , selecione **Adicionar artefato**e escolha o **NorthwindCloud Traders-recipiente** da lista de **origem (definição de compilação)** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="b6c28-289">Em **selecionar um modelo**, adicione outro ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="b6c28-290">Escolha **implantação do serviço de Azure app** e, em seguida, selecione **aplicar**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="b6c28-291">Insira `Azure Stack Hub` como o **nome do ambiente**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="b6c28-292">Na guia **tarefas** , localize e selecione Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b6c28-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="b6c28-293">Na lista de **assinaturas do Azure** , selecione **AzureStack Traders-recipiente EP** para o ponto de extremidade do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b6c28-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="b6c28-294">Insira o nome do aplicativo Web do hub de Azure Stack como o **nome do serviço de aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="b6c28-295">Em **seleção do agente**, escolha **AzureStack-b Douglas Fir** na lista **fila do agente** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="b6c28-296">Para **implantar Azure app serviço**, selecione o **pacote ou a pasta** válida para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="b6c28-297">Em **Selecionar arquivo ou pasta**, selecione **OK** para o **local**da pasta.</span><span class="sxs-lookup"><span data-stu-id="b6c28-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="b6c28-298">Na guia **variável** , localize a variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="b6c28-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="b6c28-299">Defina o valor da variável como **true**e defina seu escopo como **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="b6c28-300">Na guia **pipeline** , selecione o ícone de **gatilho de implantação contínua** para o artefato da Web do NorthwindCloud Traders e defina o **gatilho de implantação contínua** como **habilitado**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="b6c28-301">Faça a mesma coisa para o artefato de **embarcação do NorthwindCloud Traders** .</span><span class="sxs-lookup"><span data-stu-id="b6c28-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="b6c28-302">Para o ambiente de Hub de Azure Stack, selecione o ícone **condições de pré-implantação** definir o gatilho para **após a liberação**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="b6c28-303">Salve todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="b6c28-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="b6c28-304">Algumas configurações para tarefas de liberação são definidas automaticamente como [variáveis de ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) ao criar uma definição de versão a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="b6c28-304">Some settings for release tasks are automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="b6c28-305">Essas configurações não podem ser modificadas nas configurações de tarefa, mas podem ser modificadas nos itens do ambiente pai.</span><span class="sxs-lookup"><span data-stu-id="b6c28-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="b6c28-306">Criar uma versão</span><span class="sxs-lookup"><span data-stu-id="b6c28-306">Create a release</span></span>

1. <span data-ttu-id="b6c28-307">Na guia **pipeline** , abra a lista **versão** e selecione **criar versão**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="b6c28-308">Insira uma descrição para a versão, verifique se os artefatos corretos estão selecionados e, em seguida, selecione **criar**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="b6c28-309">Após alguns instantes, uma faixa é exibida indicando que a nova versão foi criada e o nome da versão é exibido como um link.</span><span class="sxs-lookup"><span data-stu-id="b6c28-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="b6c28-310">Selecione o link para ver a página de resumo da versão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="b6c28-311">A página Resumo da versão mostra detalhes sobre a versão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="b6c28-312">Na captura de tela a seguir para a "versão 2", a seção **ambientes** mostra o **status de implantação** do Azure como "em andamento", e o status para Azure Stack Hub é "êxito".</span><span class="sxs-lookup"><span data-stu-id="b6c28-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="b6c28-313">Quando o status de implantação do ambiente do Azure muda para "êxito", é exibida uma faixa indicando que a versão está pronta para aprovação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="b6c28-314">Quando uma implantação está pendente ou falhou, um ícone de informações azul **(i)** é mostrado.</span><span class="sxs-lookup"><span data-stu-id="b6c28-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="b6c28-315">Passe o mouse sobre o ícone para ver um pop-up que contém o motivo de atraso ou falha.</span><span class="sxs-lookup"><span data-stu-id="b6c28-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="b6c28-316">Outras exibições, como a lista de versões, também exibem um ícone que indica que a aprovação está pendente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="b6c28-317">O pop-up deste ícone mostra o nome do ambiente e mais detalhes relacionados à implantação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="b6c28-318">É fácil para um administrador ver o progresso geral das versões e ver quais versões estão aguardando aprovação.</span><span class="sxs-lookup"><span data-stu-id="b6c28-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="b6c28-319">Monitorar e acompanhar implantações</span><span class="sxs-lookup"><span data-stu-id="b6c28-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="b6c28-320">Na página Resumo da **versão 2** , selecione **logs**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="b6c28-321">Durante uma implantação, essa página mostra o log ao vivo do agente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="b6c28-322">O painel esquerdo mostra o status de cada operação na implantação para cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="b6c28-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="b6c28-323">Selecione o ícone de pessoa na coluna **ação** para uma aprovação de pré-implantação ou pós-implantação para ver quem aprovou (ou recusou) a implantação e a mensagem fornecida.</span><span class="sxs-lookup"><span data-stu-id="b6c28-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="b6c28-324">Após a conclusão da implantação, todo o arquivo de log será exibido no painel direito.</span><span class="sxs-lookup"><span data-stu-id="b6c28-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="b6c28-325">Selecione qualquer **etapa** no painel esquerdo para ver o arquivo de log para uma única etapa, como **inicializar trabalho**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="b6c28-326">A capacidade de ver logs individuais torna mais fácil rastrear e depurar partes da implantação geral.</span><span class="sxs-lookup"><span data-stu-id="b6c28-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="b6c28-327">**Salve** o arquivo de log para uma etapa ou **Baixe todos os logs como zip**.</span><span class="sxs-lookup"><span data-stu-id="b6c28-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="b6c28-328">Abra a guia **Resumo** para ver informações gerais sobre a versão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="b6c28-329">Essa exibição mostra detalhes sobre a compilação, os ambientes em que foram implantados, o status da implantação e outras informações sobre a versão.</span><span class="sxs-lookup"><span data-stu-id="b6c28-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="b6c28-330">Selecione um link de ambiente (**Azure** ou **Hub de Azure Stack**) para ver informações sobre implantações existentes e pendentes em um ambiente específico.</span><span class="sxs-lookup"><span data-stu-id="b6c28-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="b6c28-331">Use essas exibições como uma maneira rápida de verificar se a mesma compilação foi implantada em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="b6c28-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="b6c28-332">Abra o **aplicativo de produção implantado** em um navegador.</span><span class="sxs-lookup"><span data-stu-id="b6c28-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="b6c28-333">Por exemplo, para o site do Azure App Services, abra a URL `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="b6c28-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="b6c28-334">A integração do Azure e do hub de Azure Stack fornece uma solução escalonável entre nuvem</span><span class="sxs-lookup"><span data-stu-id="b6c28-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="b6c28-335">Um serviço flexível e robusto de várias nuvens fornece segurança de dados, backup e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escalonáveis e roteamento de conformidade geográfica.</span><span class="sxs-lookup"><span data-stu-id="b6c28-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="b6c28-336">Esse processo disparado manualmente garante a troca de carga confiável e eficiente entre aplicativos Web hospedados e a disponibilidade imediata de dados cruciais.</span><span class="sxs-lookup"><span data-stu-id="b6c28-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b6c28-337">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="b6c28-337">Next steps</span></span>

- <span data-ttu-id="b6c28-338">Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="b6c28-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
