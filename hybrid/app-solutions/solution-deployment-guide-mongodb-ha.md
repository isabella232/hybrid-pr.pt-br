---
title: Implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub
description: Saiba como implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477262"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="8faa1-103">Implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="8faa1-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="8faa1-104">Este artigo guiará você na implantação automatizada de um cluster MongoDB básico de alta disponibilidade (HA) com um site de DR (recuperação de desastre) em dois ambientes do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="8faa1-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="8faa1-105">Para saber mais sobre o MongoDB e a alta disponibilidade, confira [Membros do conjunto de réplicas](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="8faa1-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="8faa1-106">Nesta solução, você criará um exemplo de ambiente para:</span><span class="sxs-lookup"><span data-stu-id="8faa1-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="8faa1-107">Coordenar uma implantação em dois Azure Stack Hubs.</span><span class="sxs-lookup"><span data-stu-id="8faa1-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="8faa1-108">Usar o Docker para minimizar problemas de dependência com perfis de API do Azure.</span><span class="sxs-lookup"><span data-stu-id="8faa1-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="8faa1-109">Implantar um cluster MongoDB básico e de alta disponibilidade com um site de recuperação de desastre.</span><span class="sxs-lookup"><span data-stu-id="8faa1-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="8faa1-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="8faa1-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="8faa1-111">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="8faa1-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="8faa1-112">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="8faa1-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="8faa1-113">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) analisa os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="8faa1-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="8faa1-114">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="8faa1-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="8faa1-115">Arquitetura para o MongoDB com o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="8faa1-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![arquitetura de alta disponibilidade do MongoDB no Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="8faa1-117">Pré-requisitos do MongoDB com o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="8faa1-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="8faa1-118">Dois sistemas integrados e conectados do Azure Stack Hub (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="8faa1-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="8faa1-119">Essa implantação não funciona no ASDK (Kit de Desenvolvimento do Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="8faa1-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="8faa1-120">Para saber mais sobre o Azure Stack Hub, confira [O que é o Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="8faa1-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="8faa1-121">Uma assinatura de locatário em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="8faa1-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="8faa1-122">**Anote cada ID de assinatura e o ponto de extremidade do Azure Resource Manager para cada Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="8faa1-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="8faa1-123">Uma entidade de serviço do Azure Active Directory (Azure AD) com permissões para a assinatura de locatário em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="8faa1-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="8faa1-124">Você pode ter que criar duas entidades de serviço se os Azure Stack Hubs forem implantados em diferentes locatários do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="8faa1-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="8faa1-125">Para saber como criar uma entidade de serviço para o Azure Stack Hub, confira [Usar uma identidade de aplicativo para acessar recursos do Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="8faa1-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="8faa1-126">**Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) de cada entidade de serviço.**</span><span class="sxs-lookup"><span data-stu-id="8faa1-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="8faa1-127">O Ubuntu 16.04 é sindicalizado para o marketplace de cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="8faa1-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="8faa1-128">Para saber mais sobre a sindicalização de marketplace, confira [Baixar itens do marketplace para o Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="8faa1-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="8faa1-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.</span><span class="sxs-lookup"><span data-stu-id="8faa1-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="8faa1-130">Obter a imagem do Docker</span><span class="sxs-lookup"><span data-stu-id="8faa1-130">Get the Docker image</span></span>

<span data-ttu-id="8faa1-131">Ter imagens do Docker para cada implantação elimina os problemas de dependência entre diferentes versões do Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="8faa1-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="8faa1-132">Verifique se o Docker for Windows está usando contêineres do Windows.</span><span class="sxs-lookup"><span data-stu-id="8faa1-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="8faa1-133">Execute o comando a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.</span><span class="sxs-lookup"><span data-stu-id="8faa1-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="8faa1-134">Implante os clusters</span><span class="sxs-lookup"><span data-stu-id="8faa1-134">Deploy the clusters</span></span>

1. <span data-ttu-id="8faa1-135">Inicie a imagem de contêiner após ela ser recebida.</span><span class="sxs-lookup"><span data-stu-id="8faa1-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="8faa1-136">Após o contêiner ser iniciado, você verá um terminal elevado do PowerShell.</span><span class="sxs-lookup"><span data-stu-id="8faa1-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="8faa1-137">Mude de diretório para acessar o script de implantação.</span><span class="sxs-lookup"><span data-stu-id="8faa1-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="8faa1-138">Execute a implantação.</span><span class="sxs-lookup"><span data-stu-id="8faa1-138">Run the deployment.</span></span> <span data-ttu-id="8faa1-139">Forneça credenciais e nomes de recursos quando necessário.</span><span class="sxs-lookup"><span data-stu-id="8faa1-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="8faa1-140">HA se refere ao Azure Stack Hub onde o cluster de alta disponibilidade será implantado.</span><span class="sxs-lookup"><span data-stu-id="8faa1-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="8faa1-141">DR se refere ao Azure Stack Hub onde o cluster de recuperação de desastre será implantado.</span><span class="sxs-lookup"><span data-stu-id="8faa1-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="8faa1-142">Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-hybrid" a serem instalados.</span><span class="sxs-lookup"><span data-stu-id="8faa1-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="8faa1-143">Os recursos de HA serão implantados primeiro.</span><span class="sxs-lookup"><span data-stu-id="8faa1-143">The HA resources will deploy first.</span></span> <span data-ttu-id="8faa1-144">Monitore a implantação e aguarde sua conclusão.</span><span class="sxs-lookup"><span data-stu-id="8faa1-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="8faa1-145">Quando você receber a mensagem informando que a implantação HA foi concluída, poderá verificar no portal do Azure Stack Hub de alta disponibilidade para ver os recursos implantados.</span><span class="sxs-lookup"><span data-stu-id="8faa1-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="8faa1-146">Continue com a implantação dos recursos de DR e decida se deseja habilitar uma caixa de atalhos no Azure Stack Hub de recuperação de desastre para interagir com o cluster.</span><span class="sxs-lookup"><span data-stu-id="8faa1-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="8faa1-147">Aguarde até que a implantação do recurso de recuperação de desastre seja concluída.</span><span class="sxs-lookup"><span data-stu-id="8faa1-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="8faa1-148">Após a implantação do recurso de recuperação de desastre ser concluída, saia do contêiner.</span><span class="sxs-lookup"><span data-stu-id="8faa1-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="8faa1-149">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="8faa1-149">Next steps</span></span>

- <span data-ttu-id="8faa1-150">Se você habilitou a VM de caixa de atalhos no Azure Stack Hub de recuperação de desastre, poderá se conectar via SSH e interagir com o cluster do MongoDB instalando a CLI do Mongo.</span><span class="sxs-lookup"><span data-stu-id="8faa1-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="8faa1-151">Para saber mais sobre como interagir com o MongoDB, confira [O Mongo Shell](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="8faa1-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="8faa1-152">Para saber mais sobre os aplicativos de nuvem híbrida, confira [Soluções de Nuvem Híbrida.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="8faa1-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="8faa1-153">Modifique o código deste exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="8faa1-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
