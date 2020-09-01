---
title: Implantar um aplicativo que escale entre nuvens no Azure e no Azure Stack Hub
description: Saiba como implantar um aplicativo que escala entre nuvens no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886808"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="44cd3-103">Implantar um aplicativo que escale entre nuvens usando o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="44cd3-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="44cd3-104">Saiba como criar uma solução entre nuvens para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado no Azure Stack Hub para um aplicativo Web hospedado no Azure com dimensionamento automático por meio do gerenciador de tráfego.</span><span class="sxs-lookup"><span data-stu-id="44cd3-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="44cd3-105">Esse processo garante um utilitário de nuvem flexível e escalonável quando sob carga.</span><span class="sxs-lookup"><span data-stu-id="44cd3-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="44cd3-106">Com esse padrão, seu locatário pode não estar pronto para executar seu aplicativo na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="44cd3-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="44cd3-107">No entanto, pode ser economicamente inviável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="44cd3-108">Seu locatário pode fazer uso da elasticidade da nuvem pública com sua solução local.</span><span class="sxs-lookup"><span data-stu-id="44cd3-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="44cd3-109">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="44cd3-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="44cd3-110">Criar um aplicativo Web com vários nós.</span><span class="sxs-lookup"><span data-stu-id="44cd3-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="44cd3-111">Configurar e gerenciar o processo de CD (implantação contínua).</span><span class="sxs-lookup"><span data-stu-id="44cd3-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="44cd3-112">Publique o aplicativo Web no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="44cd3-113">Crie uma versão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-113">Create a release.</span></span>
> - <span data-ttu-id="44cd3-114">Aprenda a monitorar e acompanhar suas implantações.</span><span class="sxs-lookup"><span data-stu-id="44cd3-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="44cd3-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="44cd3-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="44cd3-116">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="44cd3-117">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="44cd3-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="44cd3-118">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="44cd3-119">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="44cd3-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="44cd3-120">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="44cd3-120">Prerequisites</span></span>

- <span data-ttu-id="44cd3-121">Assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-121">Azure subscription.</span></span> <span data-ttu-id="44cd3-122">Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="44cd3-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="44cd3-123">Um sistema integrado do Azure Stack Hub ou implantação do ASDK (Kit de Desenvolvimento do Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="44cd3-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="44cd3-124">Para obter instruções sobre como instalar o Azure Stack Hub, confira [Instalar o ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="44cd3-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="44cd3-125">Para um script de automação pós-implantação do ASDK, acesse: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="44cd3-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="44cd3-126">É possível que essa instalação precise de algumas horas para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="44cd3-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="44cd3-127">Implante serviços PaaS do [Serviço de Aplicativo](/azure-stack/operator/azure-stack-app-service-deploy.md) no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="44cd3-128">[Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) no ambiente do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="44cd3-129">[Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) no ambiente do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="44cd3-130">Crie um aplicativo Web na assinatura de locatário.</span><span class="sxs-lookup"><span data-stu-id="44cd3-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="44cd3-131">Anote a nova URL do aplicativo Web para uso posterior.</span><span class="sxs-lookup"><span data-stu-id="44cd3-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="44cd3-132">Implante a máquina virtual (VM) do Azure Pipelines na assinatura do locatário.</span><span class="sxs-lookup"><span data-stu-id="44cd3-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="44cd3-133">É necessário ter a VM do Windows Server 2016 com o .NET 3.5.</span><span class="sxs-lookup"><span data-stu-id="44cd3-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="44cd3-134">Essa VM será criada na assinatura de locatário no Azure Stack Hub como o agente de compilação particular.</span><span class="sxs-lookup"><span data-stu-id="44cd3-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="44cd3-135">[O Windows Server 2016 com a imagem da VM do SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Marketplace do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="44cd3-136">Caso essa imagem esteja indisponível, trabalhe com um operador do Azure Stack Hub para garantir que ele seja adicionado ao ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="44cd3-137">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="44cd3-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="44cd3-138">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="44cd3-138">Scalability</span></span>

<span data-ttu-id="44cd3-139">O principal componente da escala entre nuvens é a capacidade de fornecer colocação em escala imediata e sob demanda entre a infraestrutura de nuvem pública e local, fornecendo um serviço consistente e confiável.</span><span class="sxs-lookup"><span data-stu-id="44cd3-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="44cd3-140">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="44cd3-140">Availability</span></span>

<span data-ttu-id="44cd3-141">Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.</span><span class="sxs-lookup"><span data-stu-id="44cd3-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="44cd3-142">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="44cd3-142">Manageability</span></span>

<span data-ttu-id="44cd3-143">A solução entre nuvens garante o gerenciamento contínuo e a interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="44cd3-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="44cd3-144">O PowerShell é recomendado para o gerenciamento entre plataformas.</span><span class="sxs-lookup"><span data-stu-id="44cd3-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="44cd3-145">Escala entre nuvens</span><span class="sxs-lookup"><span data-stu-id="44cd3-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="44cd3-146">Obter um domínio personalizado e configurar o DNS</span><span class="sxs-lookup"><span data-stu-id="44cd3-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="44cd3-147">Atualize o arquivo de zona DNS do domínio.</span><span class="sxs-lookup"><span data-stu-id="44cd3-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="44cd3-148">O Azure AD verificará a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="44cd3-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="44cd3-149">Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Microsoft 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="44cd3-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="44cd3-150">Registre um domínio personalizado com um registrador público.</span><span class="sxs-lookup"><span data-stu-id="44cd3-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="44cd3-151">Entre no registrador de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="44cd3-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="44cd3-152">Talvez seja necessário um administrador aprovado para fazer atualizações de DNS.</span><span class="sxs-lookup"><span data-stu-id="44cd3-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="44cd3-153">Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="44cd3-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="44cd3-154">A entrada DNS não afetará o roteamento de email nem os comportamentos de hospedagem na Web.</span><span class="sxs-lookup"><span data-stu-id="44cd3-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="44cd3-155">Criar um aplicativo Web de vários nós padrão no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="44cd3-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="44cd3-156">Configure o CI/CD (integração contínua/implantação contínua) para implantar aplicativos Web no Azure e no Azure Stack Hub e enviar alterações por push a ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="44cd3-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="44cd3-157">São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="44cd3-158">Para obter mais informações, confira a documentação do Serviço de Aplicativo sobre os [Pré-requisitos de implantação do Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="44cd3-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="44cd3-159">Adicionar código ao Azure Repos</span><span class="sxs-lookup"><span data-stu-id="44cd3-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="44cd3-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="44cd3-160">Azure Repos</span></span>

1. <span data-ttu-id="44cd3-161">Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="44cd3-162">O CI/CD híbrido pode ser aplicado tanto ao código do aplicativo quanto ao código de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="44cd3-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="44cd3-163">Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privado e hospedado.</span><span class="sxs-lookup"><span data-stu-id="44cd3-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conectar-se a um projeto no Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="44cd3-165">**Clone o repositório** criando e abrindo o aplicativo Web padrão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonar repositório no aplicativo Web do Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="44cd3-167">Criar implantação de aplicativo Web independente para Serviços de Aplicativos em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="44cd3-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="44cd3-168">Edite o arquivo **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="44cd3-169">Selecione `Runtimeidentifier` e adicione `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="44cd3-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="44cd3-170">Confira a documentação [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="44cd3-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar arquivo de projeto de aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="44cd3-172">Faça check-in do código no Azure Repos usando o Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="44cd3-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="44cd3-173">Confirme se o código do aplicativo foi verificado no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="44cd3-174">Criar a definição de build</span><span class="sxs-lookup"><span data-stu-id="44cd3-174">Create the build definition</span></span>

1. <span data-ttu-id="44cd3-175">Entre no Azure Pipelines para confirmar a capacidade de criar definições de build.</span><span class="sxs-lookup"><span data-stu-id="44cd3-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="44cd3-176">Adicione o código **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="44cd3-177">Essa adição é necessária para disparar uma implantação independente com o .NET Core.</span><span class="sxs-lookup"><span data-stu-id="44cd3-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Adicionar código ao aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="44cd3-179">Execute o build.</span><span class="sxs-lookup"><span data-stu-id="44cd3-179">Run the build.</span></span> <span data-ttu-id="44cd3-180">O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que são executados no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="44cd3-181">Usar um agente hospedado no Azure</span><span class="sxs-lookup"><span data-stu-id="44cd3-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="44cd3-182">Usar um agente de build hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web.</span><span class="sxs-lookup"><span data-stu-id="44cd3-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="44cd3-183">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="44cd3-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="44cd3-184">Gerenciar e configurar o processo de CD</span><span class="sxs-lookup"><span data-stu-id="44cd3-184">Manage and configure the CD process</span></span>

<span data-ttu-id="44cd3-185">O Azure Pipelines e o Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade e ambientes de produção, incluindo a exigência de aprovações em estágios específicos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="44cd3-186">Criar definição de versão</span><span class="sxs-lookup"><span data-stu-id="44cd3-186">Create release definition</span></span>

1. <span data-ttu-id="44cd3-187">Selecione o botão **mais** para adicionar uma nova versão na guia **Versões** na seção **Build e Versão** do Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="44cd3-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="44cd3-189">Aplique o modelo de implantação do Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-189">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar o modelo de implantação do Serviço de Aplicativo do Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="44cd3-191">Em **Adicionar artefato**, adicione o artefato para o aplicativo de build no Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="44cd3-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Adicionar artefato ao build do Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="44cd3-193">Na guia Pipeline, selecione o link **Fase, Tarefa** do ambiente e defina os valores do ambiente de nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir os valores do ambiente de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="44cd3-195">Defina o **nome do ambiente** e selecione a **assinatura do Azure** do ponto de extremidade da Nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecione a assinatura do Azure para o ponto de extremidade do Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="44cd3-197">Em **Nome do serviço de aplicativo**, defina o nome do serviço de aplicativo necessário do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir o nome do serviço de aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="44cd3-199">Insira “Hosted VS2017” em **Fila de agentes** para o ambiente hospedado na nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Definir fila de agentes para o ambiente hospedado na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="44cd3-201">No menu Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="44cd3-202">Selecione **OK** para a **localização da pasta**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-202">Select **OK** to **folder location**.</span></span>
  
      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="44cd3-205">Salve todas as alterações e volte para o **pipeline de lançamento**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Salvar alterações no pipeline de lançamento](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="44cd3-207">Adicione um novo artefato selecionando o build do aplicativo Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Adicionar novo artefato para o aplicativo do Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="44cd3-209">Adicione mais um ambiente aplicando a Implantação do Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="44cd3-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Adicionar ambiente à implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="44cd3-211">Nomeie o novo ambiente “Azure Stack”.</span><span class="sxs-lookup"><span data-stu-id="44cd3-211">Name the new environment "Azure Stack".</span></span>

    ![Nomear ambiente na implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="44cd3-213">Localize o ambiente do Azure Stack na guia **Tarefa**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Ambiente do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="44cd3-215">Selecione a assinatura para o ponto de extremidade do Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="44cd3-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Selecionar a assinatura para o ponto de extremidade do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="44cd3-217">Defina o nome do aplicativo Web Azure Stack como o nome do serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="44cd3-218">![Definir o nome do aplicativo Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="44cd3-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="44cd3-219">Selecione o agente do Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="44cd3-219">Select the Azure Stack agent.</span></span>

    ![Selecionar o agente do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="44cd3-221">Na seção Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="44cd3-222">Selecione **OK** para a localização da pasta.</span><span class="sxs-lookup"><span data-stu-id="44cd3-222">Select **OK** to folder location.</span></span>

    ![Selecionar a pasta para a Implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecionar a pasta para a Implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="44cd3-225">Na guia Variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, defina seu valor como **true** e configura o escopo no Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="44cd3-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Adicionar variável à implantação do Aplicativo Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="44cd3-227">Selecione o ícone de gatilho de implantação **Contínuo** em ambos os artefatos e habilite o gatilho de implantação **Continuar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selecionar gatilho de implantação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="44cd3-229">Selecione o ícone de condições de **Pré-implantação** no ambiente do Azure Stack e defina o gatilho como **Após o lançamento.**</span><span class="sxs-lookup"><span data-stu-id="44cd3-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Selecionar condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="44cd3-231">Salvar todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="44cd3-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="44cd3-232">Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) na criação de uma definição da versão a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="44cd3-233">Essas configurações não podem ser modificadas nas configurações da tarefa. Em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.</span><span class="sxs-lookup"><span data-stu-id="44cd3-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="44cd3-234">Publicar no Azure Stack Hub por meio do Visual Studio</span><span class="sxs-lookup"><span data-stu-id="44cd3-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="44cd3-235">Criando pontos de extremidade, um build do Azure DevOps Services pode implantar aplicativos de serviço do Azure no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="44cd3-236">O Azure Pipelines se conecta ao agente de build, que se conecta ao Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="44cd3-237">Entre no Azure DevOps Services e acesse a página de configurações do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="44cd3-238">Em **Configurações**, selecione **Segurança**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="44cd3-239">Em **grupos do VSTS**, selecione **Criadores de ponto de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="44cd3-240">Na guia **Membros**, selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="44cd3-241">Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.</span><span class="sxs-lookup"><span data-stu-id="44cd3-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="44cd3-242">Selecione **Salvar alterações**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="44cd3-243">Na lista **Grupos do VSTS**, selecione **Administradores do ponto de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="44cd3-244">Na guia **Membros**, selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="44cd3-245">Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.</span><span class="sxs-lookup"><span data-stu-id="44cd3-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="44cd3-246">Selecione **Salvar alterações**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-246">Select **Save changes**.</span></span>

<span data-ttu-id="44cd3-247">Agora que as informações do ponto de extremidade existem, a conexão do Azure Pipelines com o Azure Stack Hub está pronta para uso.</span><span class="sxs-lookup"><span data-stu-id="44cd3-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="44cd3-248">O agente de build no Azure Stack Hub obtém instruções do Azure Pipelines e transmite informações do ponto de extremidade para comunicação com o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="44cd3-249">Desenvolver o build do aplicativo</span><span class="sxs-lookup"><span data-stu-id="44cd3-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="44cd3-250">São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="44cd3-251">Para obter mais informações, confira [Pré-requisitos para implantar o Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="44cd3-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="44cd3-252">Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) como código do aplicativo Web do Azure Repos para implantar em ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="44cd3-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="44cd3-253">Adicionar código a um projeto do Azure Repos</span><span class="sxs-lookup"><span data-stu-id="44cd3-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="44cd3-254">Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="44cd3-255">**Clone o repositório** criando e abrindo o aplicativo Web padrão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="44cd3-256">Criar implantação de aplicativo Web independente para Serviços de Aplicativos em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="44cd3-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="44cd3-257">Edite o arquivo **WebApplication.csproj**: Selecione `Runtimeidentifier` e, em seguida, adicione `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="44cd3-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="44cd3-258">Para obter mais informações, confira a documentação [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="44cd3-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="44cd3-259">Use o Team Explorer para verificar o código no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="44cd3-260">Confirme se o código do aplicativo foi verificado no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="44cd3-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="44cd3-261">Criar a definição de build</span><span class="sxs-lookup"><span data-stu-id="44cd3-261">Create the build definition</span></span>

1. <span data-ttu-id="44cd3-262">Entre no Azure Pipelines com uma conta que possa criar uma definição de build.</span><span class="sxs-lookup"><span data-stu-id="44cd3-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="44cd3-263">Acesse a página do projeto **Aplicativo Web do build**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="44cd3-264">Em **Argumentos**, adicione o código **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="44cd3-265">Essa adição é necessária para disparar uma implantação autossuficiente com o .NET Core.</span><span class="sxs-lookup"><span data-stu-id="44cd3-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="44cd3-266">Execute o build.</span><span class="sxs-lookup"><span data-stu-id="44cd3-266">Run the build.</span></span> <span data-ttu-id="44cd3-267">O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="44cd3-268">Usar um agente de build hospedado no Azure</span><span class="sxs-lookup"><span data-stu-id="44cd3-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="44cd3-269">Usar um agente de build hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web.</span><span class="sxs-lookup"><span data-stu-id="44cd3-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="44cd3-270">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="44cd3-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="44cd3-271">Configure o processo de CD (implantação contínua)</span><span class="sxs-lookup"><span data-stu-id="44cd3-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="44cd3-272">O Azure Pipelines e o Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade (QA) e produção.</span><span class="sxs-lookup"><span data-stu-id="44cd3-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="44cd3-273">Esse processo pode incluir a exigência de aprovações em estágios específicos do ciclo de vida do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="44cd3-274">Criar definição de versão</span><span class="sxs-lookup"><span data-stu-id="44cd3-274">Create release definition</span></span>

<span data-ttu-id="44cd3-275">Criar uma definição da versão é a etapa final no processo de build do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="44cd3-276">Esta definição da versão é usada para criar uma versão e implantar um build.</span><span class="sxs-lookup"><span data-stu-id="44cd3-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="44cd3-277">Entre no Azure Pipelines e acesse **Build e Versão** para navegar até o projeto.</span><span class="sxs-lookup"><span data-stu-id="44cd3-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="44cd3-278">Na guia **Versões**, selecione **[+]** e escolha **Criar definição de versão**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="44cd3-279">Em **Selecionar um modelo**, escolha **Implantação do Serviço de Aplicativo do Azure**e selecione **Aplicar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="44cd3-280">Em **Adicionar artefato** na **Origem (Definição de build)** , selecione o aplicativo de build do Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="44cd3-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="44cd3-281">Na guia **Pipeline**, selecione o link **1 Fase**, **1 Tarefa** para **Exibir tarefas de ambiente**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="44cd3-282">Na guia **Tarefas**, insira Azure como **Nome do ambiente** e selecione AzureCloud Traders-Web EP na lista de **Assinaturas do Azure**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="44cd3-283">Insira o **nome do Serviço de Aplicativo do Azure**, que é `northwindtraders` na próxima captura de tela.</span><span class="sxs-lookup"><span data-stu-id="44cd3-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="44cd3-284">Na fase de agente, selecione **VS2017 Hospedado** na lista **Fila de agentes**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="44cd3-285">Em **Implantar Serviço de Aplicativo do Azure**, selecione o **Pacote ou pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="44cd3-286">Em **Selecionar Arquivo ou Pasta**, selecione **OK** para a **Localização**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="44cd3-287">Salve todas as alterações e volte para o **Pipeline**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="44cd3-288">Na guia **Pipeline**, selecione **Adicionar artefato** e escolha **NorthwindCloud Traders-Vessel** na lista **Origem (Definição de build)** .</span><span class="sxs-lookup"><span data-stu-id="44cd3-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="44cd3-289">Em **Selecionar um Modelo**, adicione outro ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="44cd3-290">Selecione **Implantação do Serviço de Aplicativo do Azure** e, em seguida, **Aplicar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="44cd3-291">Insira `Azure Stack Hub` como o **Nome do ambiente**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="44cd3-292">Na guia **Tarefas**, localize e selecione Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="44cd3-293">Na lista de **Assinaturas do Azure**, selecione **AzureStack Traders-Vessel EP** para o ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="44cd3-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="44cd3-294">Insira o aplicativo Web do Azure Stack Hub como o **Nome do serviço de aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="44cd3-295">Em **Seleção do agente**, selecione **AzureStack -b Douglas Fir** na lista **Fila de agentes**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="44cd3-296">Em **Implantar Serviço de Aplicativo do Azure**, selecione o **Pacote ou pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="44cd3-297">Em **Selecionar Arquivo ou Pasta**, selecione **OK** para a pasta **Localização**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="44cd3-298">Na guia **Variável**, localize a variável denominada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="44cd3-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="44cd3-299">Defina o valor da variável como **true** e defina seu escopo como **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="44cd3-300">Na guia **Pipeline**, selecione o ícone do **Gatilho de implantação contínua** para o artefato NorthwindCloud Traders-Web e defina o **Gatilho de implantação contínua** como **Habilitado**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="44cd3-301">Faça o mesmo para o artefato **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="44cd3-302">No ambiente do Azure Stack Hub, selecione o ícone das **Condições de pré-implantação** para definir o gatilho como **Após o lançamento**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="44cd3-303">Salvar todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="44cd3-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="44cd3-304">Algumas configurações para as tarefas de lançamento são automaticamente definidas como [variável de ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição da versão de um modelo.</span><span class="sxs-lookup"><span data-stu-id="44cd3-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="44cd3-305">Essas configurações não podem ser modificadas nas configurações de tarefa, mas podem ser modificadas nos itens do ambiente pai.</span><span class="sxs-lookup"><span data-stu-id="44cd3-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="44cd3-306">Criar uma versão</span><span class="sxs-lookup"><span data-stu-id="44cd3-306">Create a release</span></span>

1. <span data-ttu-id="44cd3-307">Na guia **Pipeline**, abra a lista **Versão** e selecione **Criar versão**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="44cd3-308">Insira uma descrição para a versão, verifique se os artefatos corretos estão selecionados e, em seguida, selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="44cd3-309">Após alguns instantes, um banner aparece indicando que a nova versão foi criada e o nome da versão é exibido como um link.</span><span class="sxs-lookup"><span data-stu-id="44cd3-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="44cd3-310">Selecione o link para ver a página de resumo da versão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="44cd3-311">A página de resumo da versão mostra detalhes sobre ela.</span><span class="sxs-lookup"><span data-stu-id="44cd3-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="44cd3-312">Na captura de tela a seguir da “Versão-2”, a seção **Ambientes** mostra o **Status de implantação** do Azure como “EM ANDAMENTO”, e o status do Azure Stack Hub aparece como “BEM-SUCEDIDO”.</span><span class="sxs-lookup"><span data-stu-id="44cd3-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="44cd3-313">Quando o status de implantação do ambiente do Azure muda para “BEM-SUCEDIDO”, um banner aparece indicando que a versão está pronta para aprovação.</span><span class="sxs-lookup"><span data-stu-id="44cd3-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="44cd3-314">Quando uma implantação está pendente ou apresentou falhas, um ícone de informações azul **(i)** aparece.</span><span class="sxs-lookup"><span data-stu-id="44cd3-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="44cd3-315">Passe o mouse sobre o ícone para ver um pop-up que contém o motivo de atraso ou falha.</span><span class="sxs-lookup"><span data-stu-id="44cd3-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="44cd3-316">Outras exibições, como a lista de versões, também exibem um ícone que indica que a aprovação está pendente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="44cd3-317">O pop-up deste ícone mostra o nome do ambiente e mais detalhes relacionados à implantação.</span><span class="sxs-lookup"><span data-stu-id="44cd3-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="44cd3-318">É fácil para um administrador ver o progresso geral das versões e ver quais versões estão aguardando aprovação.</span><span class="sxs-lookup"><span data-stu-id="44cd3-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="44cd3-319">Monitorar e acompanhar implantações</span><span class="sxs-lookup"><span data-stu-id="44cd3-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="44cd3-320">Na página de resumo **Versão-2**, selecione **Logs**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="44cd3-321">Durante uma implantação, essa página mostra o log ativo do agente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="44cd3-322">O painel esquerdo mostra o status de cada operação na implantação para cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="44cd3-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="44cd3-323">Selecione o ícone de pessoa na coluna **Ação** para obter uma aprovação de pré-implantação ou pós-implantação e ver quem aprovou (ou recusou) a implantação e a mensagem que a pessoa forneceu.</span><span class="sxs-lookup"><span data-stu-id="44cd3-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="44cd3-324">Após a conclusão da implantação, todo o arquivo de log será exibido no painel direito.</span><span class="sxs-lookup"><span data-stu-id="44cd3-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="44cd3-325">Selecione qualquer **Etapa** no painel esquerdo para ver o arquivo de log de uma etapa única, como **Inicializar Trabalho**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="44cd3-326">A habilidade de ver logs individuais facilita o rastreamento e a depuração de partes da implantação geral.</span><span class="sxs-lookup"><span data-stu-id="44cd3-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="44cd3-327">**Salve** o arquivo de log de uma etapa ou **Baixe todos os logs como zip**.</span><span class="sxs-lookup"><span data-stu-id="44cd3-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="44cd3-328">Abra a guia **Resumo** para ver informações gerais sobre a versão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="44cd3-329">Essa exibição mostra detalhes sobre o build, os ambientes em que foram implantados, o status da implantação e outras informações sobre a versão.</span><span class="sxs-lookup"><span data-stu-id="44cd3-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="44cd3-330">Selecione um link de ambiente (**Azure** ou **Azure Stack Hub**) para ver informações sobre implantações existentes ou pendentes em um ambiente específico.</span><span class="sxs-lookup"><span data-stu-id="44cd3-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="44cd3-331">Use essas exibições como uma maneira rápida de verificar se o mesmo build foi implantado em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="44cd3-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="44cd3-332">Abra o **aplicativo de produção implantado** em um navegador.</span><span class="sxs-lookup"><span data-stu-id="44cd3-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="44cd3-333">Por exemplo, para o site dos Serviços de Aplicativos do Azure, abra a URL `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="44cd3-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="44cd3-334">A integração do Azure e do Azure Stack Hub fornece uma solução escalonável entre nuvens</span><span class="sxs-lookup"><span data-stu-id="44cd3-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="44cd3-335">Um serviço flexível e robusto de várias nuvens fornece segurança de dados, backup e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escalonáveis e roteamento de conformidade geográfica.</span><span class="sxs-lookup"><span data-stu-id="44cd3-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="44cd3-336">Esse processo disparado manualmente garante uma troca de cargas confiável e eficiente entre aplicativos Web hospedados e a disponibilidade imediata de dados cruciais.</span><span class="sxs-lookup"><span data-stu-id="44cd3-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="44cd3-337">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="44cd3-337">Next steps</span></span>

- <span data-ttu-id="44cd3-338">Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="44cd3-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
