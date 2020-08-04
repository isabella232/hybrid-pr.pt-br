---
title: Implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub
description: Saiba como implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477075"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="63e07-103">Implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="63e07-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="63e07-104">Este artigo guiará você na implantação automatizada de um cluster SQL Server 2016 Enterprise básico de alta disponibilidade (HA) com um site de DR (recuperação de desastre) assíncrona em dois ambientes do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="63e07-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="63e07-105">Para saber mais sobre o SQL Server 2016 e a alta disponibilidade, confira [Grupos de disponibilidade Always On: uma solução de alta disponibilidade e recuperação de desastre](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="63e07-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="63e07-106">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="63e07-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="63e07-107">Coordenar uma implantação em dois Azure Stack Hubs.</span><span class="sxs-lookup"><span data-stu-id="63e07-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="63e07-108">Usar o Docker para minimizar problemas de dependência com perfis de API do Azure.</span><span class="sxs-lookup"><span data-stu-id="63e07-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="63e07-109">Implantar um cluster SQL Server 2016 Enterprise básico e de alta disponibilidade com um site de recuperação de desastre.</span><span class="sxs-lookup"><span data-stu-id="63e07-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="63e07-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="63e07-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="63e07-111">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="63e07-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="63e07-112">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="63e07-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="63e07-113">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) analisa os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="63e07-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="63e07-114">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="63e07-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="63e07-115">Arquitetura para o SQL Server2016</span><span class="sxs-lookup"><span data-stu-id="63e07-115">Architecture for SQL Server 2016</span></span>

![Azure Stack Hub de alta disponibilidade para o SQL Server 2016](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="63e07-117">Pré -requisitos do SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="63e07-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="63e07-118">Dois sistemas integrados e conectados do Azure Stack Hub (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="63e07-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="63e07-119">Essa implantação não funciona no ASDK (Kit de Desenvolvimento do Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="63e07-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="63e07-120">Para saber mais sobre o Azure Stack Hub, confira [Visão geral do Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="63e07-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="63e07-121">Uma assinatura de locatário em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="63e07-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="63e07-122">**Anote cada ID de assinatura e o ponto de extremidade do Azure Resource Manager para cada Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="63e07-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="63e07-123">Uma entidade de serviço do Azure Active Directory (Azure AD) com permissões para a assinatura de locatário em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="63e07-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="63e07-124">Você pode ter que criar duas entidades de serviço se os Azure Stack Hubs forem implantados em diferentes locatários do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="63e07-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="63e07-125">Para saber como criar uma entidade de serviço para o Azure Stack Hub, confira [Criar entidades de serviço para permitir que os aplicativos acessem recursos do Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="63e07-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="63e07-126">**Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) de cada entidade de serviço.**</span><span class="sxs-lookup"><span data-stu-id="63e07-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="63e07-127">O SQL Server 2016 Enterprise é sindicalizado para o marketplace de cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="63e07-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="63e07-128">Para saber mais sobre a sindicalização de marketplace, confira [Baixar itens do marketplace para o Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="63e07-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="63e07-129">**Verifique se sua organização tem as licenças do SQL apropriadas.**</span><span class="sxs-lookup"><span data-stu-id="63e07-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="63e07-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.</span><span class="sxs-lookup"><span data-stu-id="63e07-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="63e07-131">Obter a imagem do Docker</span><span class="sxs-lookup"><span data-stu-id="63e07-131">Get the Docker image</span></span>

<span data-ttu-id="63e07-132">Ter imagens do Docker para cada implantação elimina os problemas de dependência entre diferentes versões do Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="63e07-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="63e07-133">Verifique se o Docker for Windows está usando contêineres do Windows.</span><span class="sxs-lookup"><span data-stu-id="63e07-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="63e07-134">Execute o script a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.</span><span class="sxs-lookup"><span data-stu-id="63e07-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="63e07-135">Implante o grupo de disponibilidade</span><span class="sxs-lookup"><span data-stu-id="63e07-135">Deploy the availability group</span></span>

1. <span data-ttu-id="63e07-136">Inicie a imagem de contêiner após ela ser recebida.</span><span class="sxs-lookup"><span data-stu-id="63e07-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="63e07-137">Após o contêiner ser iniciado, você verá um terminal elevado do PowerShell.</span><span class="sxs-lookup"><span data-stu-id="63e07-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="63e07-138">Mude de diretório para acessar o script de implantação.</span><span class="sxs-lookup"><span data-stu-id="63e07-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="63e07-139">Execute a implantação.</span><span class="sxs-lookup"><span data-stu-id="63e07-139">Run the deployment.</span></span> <span data-ttu-id="63e07-140">Forneça credenciais e nomes de recursos quando necessário.</span><span class="sxs-lookup"><span data-stu-id="63e07-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="63e07-141">HA se refere ao Azure Stack Hub onde o cluster de alta disponibilidade será implantado.</span><span class="sxs-lookup"><span data-stu-id="63e07-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="63e07-142">DR se refere ao Azure Stack Hub onde o cluster de recuperação de desastre será implantado.</span><span class="sxs-lookup"><span data-stu-id="63e07-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="63e07-143">Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-hybrid" a serem instalados.</span><span class="sxs-lookup"><span data-stu-id="63e07-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="63e07-144">Aguarde até que a implantação do recurso seja concluída.</span><span class="sxs-lookup"><span data-stu-id="63e07-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="63e07-145">Após a implantação do recurso de recuperação de desastre ser concluída, saia do contêiner.</span><span class="sxs-lookup"><span data-stu-id="63e07-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="63e07-146">Inspecione a implantação visualizando os recursos no portal de cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="63e07-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="63e07-147">Conecte-se a uma das instâncias do SQL no ambiente de HA e inspecione o grupo de disponibilidade por meio do SSMS (SQL Server Management Studio).</span><span class="sxs-lookup"><span data-stu-id="63e07-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="63e07-149">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="63e07-149">Next steps</span></span>

- <span data-ttu-id="63e07-150">Use o SQL Server Management Studio para fazer o failover manual do cluster.</span><span class="sxs-lookup"><span data-stu-id="63e07-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="63e07-151">Confira [Executar um failover manual forçado de um grupo de disponibilidade Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="63e07-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="63e07-152">Saiba mais sobre aplicativos de nuvem híbrida.</span><span class="sxs-lookup"><span data-stu-id="63e07-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="63e07-153">Confira [Soluções de Nuvem Híbrida.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="63e07-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="63e07-154">Use seus próprios dados ou modifique o código deste exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="63e07-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
