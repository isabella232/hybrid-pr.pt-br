---
title: Implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack
description: Saiba como implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909808"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="9663e-103">Implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="9663e-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="9663e-104">Este artigo o orientará em uma implantação automatizada de um cluster de HA (alta disponibilidade) SQL Server 2016 Enterprise com um site de DR (recuperação de desastre assíncrono) em dois ambientes de Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9663e-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="9663e-105">Para saber mais sobre o SQL Server 2016 e a alta disponibilidade, consulte [grupos de disponibilidade Always on: uma solução de alta disponibilidade e recuperação de desastres](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="9663e-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="9663e-106">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="9663e-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="9663e-107">Orquestrar uma implantação em dois hubs de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9663e-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="9663e-108">Use o Docker para minimizar problemas de dependência com perfis de API do Azure.</span><span class="sxs-lookup"><span data-stu-id="9663e-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="9663e-109">Implante um cluster do SQL Server 2016 Enterprise altamente disponível básico com um site de recuperação de desastre.</span><span class="sxs-lookup"><span data-stu-id="9663e-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="9663e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="9663e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="9663e-111">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="9663e-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="9663e-112">Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="9663e-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="9663e-113">O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="9663e-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="9663e-114">As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="9663e-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="9663e-115">Arquitetura para SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="9663e-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="9663e-117">Pré-requisitos para o SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="9663e-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="9663e-118">Dois sistemas integrados de Hub de Azure Stack (Hub de Azure Stack) conectados.</span><span class="sxs-lookup"><span data-stu-id="9663e-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="9663e-119">Essa implantação não funciona no Kit de Desenvolvimento do Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="9663e-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="9663e-120">Para saber mais sobre o Hub de Azure Stack, consulte a [visão geral Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="9663e-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="9663e-121">Uma assinatura de locatário em cada Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9663e-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="9663e-122">**Anote cada ID de assinatura e o ponto de extremidade de Azure Resource Manager para cada Hub de Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="9663e-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="9663e-123">Uma entidade de serviço Azure Active Directory (AD do Azure) que tem permissões para a assinatura de locatário em cada Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9663e-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="9663e-124">Talvez seja necessário criar duas entidades de serviço se os hubs de Azure Stack forem implantados em diferentes locatários do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9663e-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="9663e-125">Para saber como criar uma entidade de serviço para Azure Stack Hub, consulte [criar entidades de serviço para fornecer aos aplicativos acesso aos recursos do Hub Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="9663e-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="9663e-126">**Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) da entidade de serviço.**</span><span class="sxs-lookup"><span data-stu-id="9663e-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="9663e-127">SQL Server 2016 Enterprise distribuído para cada Marketplace de Azure Stack do Hub.</span><span class="sxs-lookup"><span data-stu-id="9663e-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="9663e-128">Para saber mais sobre a distribuição do Marketplace, confira [baixar itens do Marketplace para Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="9663e-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="9663e-129">**Verifique se sua organização tem as licenças SQL apropriadas.**</span><span class="sxs-lookup"><span data-stu-id="9663e-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="9663e-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.</span><span class="sxs-lookup"><span data-stu-id="9663e-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="9663e-131">Obter a imagem do Docker</span><span class="sxs-lookup"><span data-stu-id="9663e-131">Get the Docker image</span></span>

<span data-ttu-id="9663e-132">As imagens do Docker para cada implantação eliminam os problemas de dependência entre diferentes versões do Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="9663e-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="9663e-133">Verifique se Docker for Windows está usando contêineres do Windows.</span><span class="sxs-lookup"><span data-stu-id="9663e-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="9663e-134">Execute o script a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.</span><span class="sxs-lookup"><span data-stu-id="9663e-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="9663e-135">Implantar o grupo de disponibilidade</span><span class="sxs-lookup"><span data-stu-id="9663e-135">Deploy the availability group</span></span>

1. <span data-ttu-id="9663e-136">Depois que a imagem de contêiner tiver sido recebida com êxito, inicie a imagem.</span><span class="sxs-lookup"><span data-stu-id="9663e-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="9663e-137">Depois que o contêiner for iniciado, você receberá um terminal elevado do PowerShell no contêiner.</span><span class="sxs-lookup"><span data-stu-id="9663e-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="9663e-138">Altere os diretórios para obter o script de implantação.</span><span class="sxs-lookup"><span data-stu-id="9663e-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="9663e-139">Execute a implantação.</span><span class="sxs-lookup"><span data-stu-id="9663e-139">Run the deployment.</span></span> <span data-ttu-id="9663e-140">Forneça credenciais e nomes de recursos quando necessário.</span><span class="sxs-lookup"><span data-stu-id="9663e-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="9663e-141">HA refere-se ao Hub de Azure Stack onde o cluster de alta disponibilidade será implantado.</span><span class="sxs-lookup"><span data-stu-id="9663e-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="9663e-142">A recuperação de desastre refere-se ao Hub de Azure Stack onde o cluster de DR será implantado.</span><span class="sxs-lookup"><span data-stu-id="9663e-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="9663e-143">Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-Hybrid" a serem instalados.</span><span class="sxs-lookup"><span data-stu-id="9663e-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="9663e-144">Aguarde a conclusão da implantação de recursos.</span><span class="sxs-lookup"><span data-stu-id="9663e-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="9663e-145">Após a conclusão da implantação de recursos de recuperação de desastre, saia do contêiner.</span><span class="sxs-lookup"><span data-stu-id="9663e-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="9663e-146">Inspecione a implantação exibindo os recursos em cada portal do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9663e-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="9663e-147">Conecte-se a uma das instâncias do SQL no ambiente de HA e inspecione o grupo de disponibilidade por meio do SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="9663e-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![2016 SQL HA de SQL Server](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="9663e-149">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="9663e-149">Next steps</span></span>

- <span data-ttu-id="9663e-150">Use SQL Server Management Studio para fazer failover manual do cluster.</span><span class="sxs-lookup"><span data-stu-id="9663e-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="9663e-151">Consulte [executar um failover manual forçado de um grupo de disponibilidade de Always on (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="9663e-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="9663e-152">Saiba mais sobre os aplicativos de nuvem híbrida.</span><span class="sxs-lookup"><span data-stu-id="9663e-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="9663e-153">Consulte [soluções de nuvem híbrida.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="9663e-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="9663e-154">Use seus próprios dados ou modifique o código para este exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="9663e-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
