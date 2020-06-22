---
title: Implantar o aplicativo híbrido com dados locais que dimensionam entre nuvem
description: Saiba como implantar um aplicativo que usa dados locais e dimensiona entre nuvem usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909786"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="71cbe-103">Implantar o aplicativo híbrido com dados locais que dimensionam entre nuvem</span><span class="sxs-lookup"><span data-stu-id="71cbe-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="71cbe-104">Este guia de solução mostra como implantar um aplicativo híbrido que abrange o Hub do Azure e do Azure Stack e usa uma única fonte de dados local.</span><span class="sxs-lookup"><span data-stu-id="71cbe-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="71cbe-105">Usando uma solução de nuvem híbrida, você pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="71cbe-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="71cbe-106">Seus desenvolvedores também podem aproveitar o ecossistema do desenvolvedor da Microsoft e aplicar suas habilidades aos ambientes locais e na nuvem.</span><span class="sxs-lookup"><span data-stu-id="71cbe-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="71cbe-107">Visão geral e suposições</span><span class="sxs-lookup"><span data-stu-id="71cbe-107">Overview and assumptions</span></span>

<span data-ttu-id="71cbe-108">Siga este tutorial para configurar um fluxo de trabalho que permite aos desenvolvedores implantar um aplicativo Web idêntico em uma nuvem pública e em uma nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="71cbe-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="71cbe-109">Este aplicativo pode acessar uma rede roteável que não seja da Internet hospedada na nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="71cbe-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="71cbe-110">Esses aplicativos Web são monitorados e quando há um pico no tráfego, um programa modifica os registros DNS para redirecionar o tráfego para a nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="71cbe-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="71cbe-111">Quando o tráfego cai para o nível antes do pico, o tráfego é roteado de volta para a nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="71cbe-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="71cbe-112">Este tutorial cobre as seguintes tarefas:</span><span class="sxs-lookup"><span data-stu-id="71cbe-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="71cbe-113">Implante um servidor de banco de dados SQL Server conectado por híbrido.</span><span class="sxs-lookup"><span data-stu-id="71cbe-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="71cbe-114">Conecte um aplicativo Web no Azure global a uma rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="71cbe-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="71cbe-115">Configure o DNS para dimensionamento entre nuvem.</span><span class="sxs-lookup"><span data-stu-id="71cbe-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="71cbe-116">Configurar certificados SSL para dimensionamento entre nuvem.</span><span class="sxs-lookup"><span data-stu-id="71cbe-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="71cbe-117">Configure e implante o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="71cbe-118">Crie um perfil do Gerenciador de tráfego e configure-o para o dimensionamento entre nuvem.</span><span class="sxs-lookup"><span data-stu-id="71cbe-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="71cbe-119">Configure Application Insights monitoramento e alertas para aumentar o tráfego.</span><span class="sxs-lookup"><span data-stu-id="71cbe-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="71cbe-120">Configure a alternância automática de tráfego entre o Azure global e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="71cbe-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="71cbe-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="71cbe-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="71cbe-122">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="71cbe-123">Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="71cbe-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="71cbe-124">O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="71cbe-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="71cbe-125">As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="71cbe-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="71cbe-126">Suposições</span><span class="sxs-lookup"><span data-stu-id="71cbe-126">Assumptions</span></span>

<span data-ttu-id="71cbe-127">Este tutorial pressupõe que você tenha um conhecimento básico do Azure global e do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="71cbe-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="71cbe-128">Se você quiser saber mais antes de iniciar o tutorial, examine estes artigos:</span><span class="sxs-lookup"><span data-stu-id="71cbe-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="71cbe-129">Introdução ao Azure</span><span class="sxs-lookup"><span data-stu-id="71cbe-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="71cbe-130">Conceitos de chave de Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="71cbe-131">Este tutorial também pressupõe que você tenha uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="71cbe-132">Se você não tiver uma assinatura, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="71cbe-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="71cbe-133">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="71cbe-133">Prerequisites</span></span>

<span data-ttu-id="71cbe-134">Antes de iniciar essa solução, verifique se você atende aos seguintes requisitos:</span><span class="sxs-lookup"><span data-stu-id="71cbe-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="71cbe-135">Um Kit de Desenvolvimento do Azure Stack (ASDK) ou uma assinatura em um sistema integrado de Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="71cbe-136">Para implantar o ASDK, siga as instruções em [implantar o ASDK usando o instalador](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="71cbe-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="71cbe-137">A instalação do hub de Azure Stack deve ter o seguinte instalado:</span><span class="sxs-lookup"><span data-stu-id="71cbe-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="71cbe-138">O serviço Azure App.</span><span class="sxs-lookup"><span data-stu-id="71cbe-138">The Azure App Service.</span></span> <span data-ttu-id="71cbe-139">Trabalhe com seu operador de Hub de Azure Stack para implantar e configurar o serviço de Azure App em seu ambiente.</span><span class="sxs-lookup"><span data-stu-id="71cbe-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="71cbe-140">Este tutorial requer que o serviço de aplicativo tenha pelo menos uma (1) função de trabalho dedicada disponível.</span><span class="sxs-lookup"><span data-stu-id="71cbe-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="71cbe-141">Uma imagem do Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="71cbe-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="71cbe-142">Um Windows Server 2016 com uma imagem Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="71cbe-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="71cbe-143">Os planos e ofertas apropriados.</span><span class="sxs-lookup"><span data-stu-id="71cbe-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="71cbe-144">Um nome de domínio para seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-144">A domain name for your web app.</span></span> <span data-ttu-id="71cbe-145">Se você não tiver um nome de domínio, poderá comprar um de um provedor de domínio como GoDaddy, BlueHost e InMotion.</span><span class="sxs-lookup"><span data-stu-id="71cbe-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="71cbe-146">Um certificado SSL para seu domínio de uma autoridade de certificação confiável como LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="71cbe-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="71cbe-147">Um aplicativo Web que se comunica com um banco de dados SQL Server e oferece suporte a Application Insights.</span><span class="sxs-lookup"><span data-stu-id="71cbe-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="71cbe-148">Você pode baixar o aplicativo de exemplo [dotnetcore-SQLDB-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do github.</span><span class="sxs-lookup"><span data-stu-id="71cbe-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="71cbe-149">Uma rede híbrida entre uma rede virtual do Azure e uma rede virtual do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="71cbe-150">Para obter instruções detalhadas, consulte [Configurar a conectividade de nuvem híbrida com o Azure e o Hub de Azure Stack](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="71cbe-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="71cbe-151">Um pipeline de CI/CD (integração contínua/implantação contínua) com um agente de compilação particular no Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="71cbe-152">Para obter instruções detalhadas, consulte [Configurar a identidade de nuvem híbrida com o Azure e os aplicativos de Hub de Azure Stack](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="71cbe-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="71cbe-153">Implantar um servidor de banco de dados SQL Server conectado por híbrido</span><span class="sxs-lookup"><span data-stu-id="71cbe-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="71cbe-154">Entre no portal do usuário do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="71cbe-155">No **painel**, selecione **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Marketplace do Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="71cbe-157">No **Marketplace**, selecione **computação**e, em seguida, escolha **mais**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="71cbe-158">Em **mais**, selecione a **licença de SQL Server gratuita: SQL Server 2017 desenvolvedor na imagem do Windows Server** .</span><span class="sxs-lookup"><span data-stu-id="71cbe-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Selecionar uma imagem de máquina virtual no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="71cbe-160">Em **licença gratuita de SQL Server: SQL Server desenvolvedor 2017 no Windows Server**, selecione **criar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="71cbe-161">Em **noções básicas > definir configurações básicas**, forneça um **nome** para a VM (máquina virtual), um **nome de usuário** para o SA do SQL Server e uma **senha** para o SA.</span><span class="sxs-lookup"><span data-stu-id="71cbe-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="71cbe-162">Na lista suspensa **assinatura** , selecione a assinatura na qual você está implantando.</span><span class="sxs-lookup"><span data-stu-id="71cbe-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="71cbe-163">Para **grupo de recursos**, use **escolher existente** e coloque a VM no mesmo grupo de recursos que o seu aplicativo Web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="71cbe-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Definir configurações básicas para VM no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="71cbe-165">Em **tamanho**, escolha um tamanho para a VM.</span><span class="sxs-lookup"><span data-stu-id="71cbe-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="71cbe-166">Para este tutorial, recomendamos A2_Standard ou uma DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="71cbe-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="71cbe-167">Em **configurações > configurar recursos opcionais**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="71cbe-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="71cbe-168">**Conta de armazenamento**: Crie uma nova conta se precisar de uma.</span><span class="sxs-lookup"><span data-stu-id="71cbe-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="71cbe-169">**Rede virtual**:</span><span class="sxs-lookup"><span data-stu-id="71cbe-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="71cbe-170">Verifique se sua VM SQL Server está implantada na mesma rede virtual que os gateways de VPN.</span><span class="sxs-lookup"><span data-stu-id="71cbe-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="71cbe-171">**Endereço IP público**: Use as configurações padrão.</span><span class="sxs-lookup"><span data-stu-id="71cbe-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="71cbe-172">**Grupo de segurança de rede**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="71cbe-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="71cbe-173">Crie um novo NSG.</span><span class="sxs-lookup"><span data-stu-id="71cbe-173">Create a new NSG.</span></span>
   - <span data-ttu-id="71cbe-174">**Extensões e monitoramento**: Mantenha as configurações padrão.</span><span class="sxs-lookup"><span data-stu-id="71cbe-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="71cbe-175">**Conta de armazenamento de diagnóstico**: Crie uma nova conta se precisar de uma.</span><span class="sxs-lookup"><span data-stu-id="71cbe-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="71cbe-176">Selecione **OK** para salvar sua configuração.</span><span class="sxs-lookup"><span data-stu-id="71cbe-176">Select **OK** to save your configuration.</span></span>

     ![Configurar recursos de VM opcionais no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="71cbe-178">Em **configurações de SQL Server**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="71cbe-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="71cbe-179">Para **conectividade do SQL**, selecione **público (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="71cbe-180">Para **porta**, mantenha o padrão, **1433**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="71cbe-181">Para **autenticação do SQL**, selecione **habilitar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="71cbe-182">Quando você habilita a autenticação do SQL, ela deve ser preenchida automaticamente com as informações de "sqladmin" que você configurou em **noções básicas**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="71cbe-183">Para o restante das configurações, mantenha os padrões.</span><span class="sxs-lookup"><span data-stu-id="71cbe-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="71cbe-184">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-184">Select **OK**.</span></span>

     ![Definir configurações de SQL Server no portal de usuário de Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="71cbe-186">Em **Resumo**, examine a configuração da VM e, em seguida, selecione **OK** para iniciar a implantação.</span><span class="sxs-lookup"><span data-stu-id="71cbe-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Resumo da configuração no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="71cbe-188">Leva algum tempo para criar a nova VM.</span><span class="sxs-lookup"><span data-stu-id="71cbe-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="71cbe-189">Você pode exibir o STATUS de suas VMs em **máquinas virtuais**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Status das máquinas virtuais no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="71cbe-191">Criar aplicativos Web no Azure e no Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-192">O serviço de Azure App simplifica a execução e o gerenciamento de um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="71cbe-193">Como o Hub de Azure Stack é consistente com o Azure, o serviço de aplicativo pode ser executado em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="71cbe-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="71cbe-194">Você usará o serviço de aplicativo para hospedar seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="71cbe-195">Criar aplicativos Web</span><span class="sxs-lookup"><span data-stu-id="71cbe-195">Create web apps</span></span>

1. <span data-ttu-id="71cbe-196">Crie um aplicativo Web no Azure seguindo as instruções em [gerenciar um plano do serviço de aplicativo no Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="71cbe-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="71cbe-197">Certifique-se de colocar o aplicativo Web na mesma assinatura e grupo de recursos que a sua rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="71cbe-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="71cbe-198">Repita a etapa anterior (1) no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="71cbe-199">Adicionar rota para o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-200">O serviço de aplicativo no Hub de Azure Stack deve ser roteável da Internet pública para permitir que os usuários acessem seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="71cbe-201">Se o Hub de Azure Stack estiver acessível pela Internet, anote o endereço IP ou a URL voltada para o público para o aplicativo Web do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="71cbe-202">Se você estiver usando um ASDK, poderá [configurar um mapeamento NAT estático](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o serviço de aplicativo fora do ambiente virtual.</span><span class="sxs-lookup"><span data-stu-id="71cbe-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="71cbe-203">Conectar um aplicativo Web no Azure a uma rede híbrida</span><span class="sxs-lookup"><span data-stu-id="71cbe-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="71cbe-204">Para fornecer conectividade entre o front-end da Web no Azure e o banco de dados SQL Server no Hub Azure Stack, o aplicativo Web deve estar conectado à rede híbrida entre o Azure e o Hub do Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="71cbe-205">Para habilitar a conectividade, você terá que:</span><span class="sxs-lookup"><span data-stu-id="71cbe-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="71cbe-206">Configure a conectividade ponto a site.</span><span class="sxs-lookup"><span data-stu-id="71cbe-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="71cbe-207">Configure o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-207">Configure the web app.</span></span>
- <span data-ttu-id="71cbe-208">Modifique o gateway de rede local no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="71cbe-209">Configurar a rede virtual do Azure para conectividade ponto a site</span><span class="sxs-lookup"><span data-stu-id="71cbe-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="71cbe-210">O gateway de rede virtual no lado do Azure da rede híbrida deve permitir conexões ponto a site a serem integradas ao serviço Azure App.</span><span class="sxs-lookup"><span data-stu-id="71cbe-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="71cbe-211">No Azure, vá para a página gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="71cbe-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="71cbe-212">Em **configurações**, selecione **configuração ponto a site**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opção ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="71cbe-214">Selecione **Configurar agora** para configurar o ponto a site.</span><span class="sxs-lookup"><span data-stu-id="71cbe-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Iniciar configuração de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="71cbe-216">Na página configuração **ponto a site** , insira o intervalo de endereços IP privado que você deseja usar no **pool de endereços**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="71cbe-217">Certifique-se de que o intervalo especificado não se sobrepõe a nenhum dos intervalos de endereços já usados por sub-redes nos componentes globais do Azure ou do hub de Azure Stack da rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="71cbe-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="71cbe-218">Em **tipo de túnel**, desmarque a **VPN IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="71cbe-219">Selecione **salvar** para concluir a configuração de ponto a site.</span><span class="sxs-lookup"><span data-stu-id="71cbe-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Configurações de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="71cbe-221">Integrar o aplicativo de serviço de Azure App com a rede híbrida</span><span class="sxs-lookup"><span data-stu-id="71cbe-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="71cbe-222">Para conectar o aplicativo à VNet do Azure, siga as instruções em [integração vnet necessária do gateway](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="71cbe-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="71cbe-223">Vá para **configurações** para o plano do serviço de aplicativo que hospeda o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="71cbe-224">Em **Configurações**, selecione **Rede**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-224">In **Settings**, select **Networking**.</span></span>

    ![Configurar a rede para o plano do serviço de aplicativo](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="71cbe-226">Em **integração VNET**, selecione **clique aqui para gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Gerenciar a integração VNET para o plano do serviço de aplicativo](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="71cbe-228">Selecione a VNET que você deseja configurar.</span><span class="sxs-lookup"><span data-stu-id="71cbe-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="71cbe-229">Em **endereços IP roteados para VNET**, insira o intervalo de endereços IP para a vnet do Azure, a vnet do Hub de Azure Stack e os espaços de endereço ponto a site.</span><span class="sxs-lookup"><span data-stu-id="71cbe-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="71cbe-230">Selecione **salvar** para validar e salvar essas configurações.</span><span class="sxs-lookup"><span data-stu-id="71cbe-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalos de endereços IP a serem roteados na integração de rede virtual](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="71cbe-232">Para saber mais sobre como o serviço de aplicativo se integra ao Azure VNets, consulte [integrar seu aplicativo a uma rede virtual do Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="71cbe-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="71cbe-233">Configurar a rede virtual do hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="71cbe-234">O gateway de rede local na rede virtual do hub de Azure Stack precisa ser configurado para rotear o tráfego do intervalo de endereços de ponto a site do serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="71cbe-235">No Hub Azure Stack, vá para **Gateway de rede local**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="71cbe-236">Em **Configurações**, escolha **Configuração**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opção de configuração de gateway no gateway de rede local do Hub Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="71cbe-238">Em **espaço de endereço**, insira o intervalo de endereços de ponto a site para o gateway de rede virtual no Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Espaço de endereço ponto a site no gateway de rede local do Hub Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="71cbe-240">Selecione **salvar** para validar e salvar a configuração.</span><span class="sxs-lookup"><span data-stu-id="71cbe-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="71cbe-241">Configurar o DNS para dimensionamento entre nuvem</span><span class="sxs-lookup"><span data-stu-id="71cbe-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="71cbe-242">Ao configurar corretamente o DNS para aplicativos de nuvem cruzada, os usuários podem acessar as instâncias globais do Azure e do hub de Azure Stack do seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="71cbe-243">A configuração de DNS para este tutorial também permite que o Gerenciador de tráfego do Azure encaminhe o tráfego quando a carga aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="71cbe-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="71cbe-244">Este tutorial usa o DNS do Azure para gerenciar o DNS porque os domínios do serviço de aplicativo não funcionarão.</span><span class="sxs-lookup"><span data-stu-id="71cbe-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="71cbe-245">Criar subdomínios</span><span class="sxs-lookup"><span data-stu-id="71cbe-245">Create subdomains</span></span>

<span data-ttu-id="71cbe-246">Como o Gerenciador de tráfego depende de CNAMEs DNS, um subdomínio é necessário para rotear corretamente o tráfego para os pontos de extremidade.</span><span class="sxs-lookup"><span data-stu-id="71cbe-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="71cbe-247">Para obter mais informações sobre os registros DNS e o mapeamento de domínio, consulte [mapear domínios com o Gerenciador de tráfego](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="71cbe-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="71cbe-248">Para o ponto de extremidade do Azure, você criará um subdomínio que os usuários podem usar para acessar seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="71cbe-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="71cbe-249">Para este tutorial, pode usar **app.Northwind.com**, mas você deve personalizar esse valor com base em seu próprio domínio.</span><span class="sxs-lookup"><span data-stu-id="71cbe-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="71cbe-250">Você também precisará criar um subdomínio com um registro A para o ponto de extremidade do Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="71cbe-251">Você pode usar **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="71cbe-252">Configurar um domínio personalizado no Azure</span><span class="sxs-lookup"><span data-stu-id="71cbe-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="71cbe-253">Adicione o nome de host **app.Northwind.com** ao aplicativo Web do Azure [mapeando um CNAME para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="71cbe-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="71cbe-254">Configurar domínios personalizados no Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="71cbe-255">Adicione o nome de host **azurestack.Northwind.com** ao aplicativo Web Hub de Azure Stack [mapeando um registro a para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="71cbe-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="71cbe-256">Use o endereço IP roteável da Internet para o aplicativo do serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="71cbe-257">Adicione o nome de host **app.Northwind.com** ao aplicativo Web Hub de Azure Stack [mapeando um CNAME para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="71cbe-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="71cbe-258">Use o nome de host que você configurou na etapa anterior (1) como o destino para o CNAME.</span><span class="sxs-lookup"><span data-stu-id="71cbe-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="71cbe-259">Configurar certificados SSL para dimensionamento entre nuvem</span><span class="sxs-lookup"><span data-stu-id="71cbe-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="71cbe-260">É importante garantir que os dados confidenciais coletados pelo seu aplicativo Web sejam protegidos em trânsito e quando armazenados no banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="71cbe-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="71cbe-261">Você configurará seus aplicativos Web do Azure e do Azure Stack Hub para usar certificados SSL para todo o tráfego de entrada.</span><span class="sxs-lookup"><span data-stu-id="71cbe-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="71cbe-262">Adicionar SSL ao Azure e ao Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-263">Para adicionar SSL ao Azure:</span><span class="sxs-lookup"><span data-stu-id="71cbe-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="71cbe-264">Certifique-se de que o certificado SSL obtido é válido para o subdomínio que você criou.</span><span class="sxs-lookup"><span data-stu-id="71cbe-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="71cbe-265">(Não há problema em usar certificados curinga.)</span><span class="sxs-lookup"><span data-stu-id="71cbe-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="71cbe-266">No Azure, siga as instruções nas seções **preparar seu aplicativo Web** e **associar seu certificado SSL** do artigo [associar um certificado SSL personalizado existente a aplicativos Web do Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="71cbe-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="71cbe-267">Selecione **SSL baseado em SNI** como o **tipo de SSL**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="71cbe-268">Redirecionar todo o tráfego para a porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="71cbe-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="71cbe-269">Siga as instruções na seção **impor https** do artigo [associar um certificado SSL personalizado existente a aplicativos Web do Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="71cbe-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="71cbe-270">Para adicionar SSL ao Hub Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="71cbe-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="71cbe-271">Repita as etapas a 1-3 que você usou para o Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="71cbe-272">Configurar e implantar o aplicativo Web</span><span class="sxs-lookup"><span data-stu-id="71cbe-272">Configure and deploy the web app</span></span>

<span data-ttu-id="71cbe-273">Você configurará o código do aplicativo para relatar telemetria para a instância de Application Insights correta e configurará os aplicativos Web com as cadeias de conexão corretas.</span><span class="sxs-lookup"><span data-stu-id="71cbe-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="71cbe-274">Para saber mais sobre Application Insights, confira [o que é Application insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="71cbe-274">To learn more about Application Insights, see [What is Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="71cbe-275">Adicionar Application Insights</span><span class="sxs-lookup"><span data-stu-id="71cbe-275">Add Application Insights</span></span>

1. <span data-ttu-id="71cbe-276">Abra seu aplicativo Web no Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="71cbe-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="71cbe-277">[Adicione Application insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application insights usa para criar alertas quando o tráfego da Web aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="71cbe-277">[Add Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="71cbe-278">Configurar cadeias de conexão dinâmicas</span><span class="sxs-lookup"><span data-stu-id="71cbe-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="71cbe-279">Cada instância do aplicativo Web usará um método diferente para se conectar ao banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="71cbe-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="71cbe-280">O aplicativo no Azure usa o endereço IP privado da VM SQL Server e o aplicativo no Hub Azure Stack usa o endereço IP público da VM SQL Server.</span><span class="sxs-lookup"><span data-stu-id="71cbe-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="71cbe-281">Em um sistema integrado de Hub Azure Stack, o endereço IP público não deve ser roteável pela Internet.</span><span class="sxs-lookup"><span data-stu-id="71cbe-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="71cbe-282">Em um ASDK, o endereço IP público não é roteável fora do ASDK.</span><span class="sxs-lookup"><span data-stu-id="71cbe-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="71cbe-283">Você pode usar variáveis de ambiente do serviço de aplicativo para passar uma cadeia de conexão diferente para cada instância do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="71cbe-284">Abra o aplicativo no Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="71cbe-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="71cbe-285">Abra Startup.cs e localize o seguinte bloco de código:</span><span class="sxs-lookup"><span data-stu-id="71cbe-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="71cbe-286">Substitua o bloco de código anterior pelo código a seguir, que usa uma cadeia de conexão definida no *appsettings.jsno* arquivo:</span><span class="sxs-lookup"><span data-stu-id="71cbe-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="71cbe-287">Definir configurações do aplicativo do serviço de aplicativo</span><span class="sxs-lookup"><span data-stu-id="71cbe-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="71cbe-288">Crie cadeias de conexão para o Azure e o Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="71cbe-289">As cadeias de caracteres devem ser as mesmas, exceto os endereços IP que são usados.</span><span class="sxs-lookup"><span data-stu-id="71cbe-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="71cbe-290">No Azure e no Hub de Azure Stack, adicione a cadeia de conexão apropriada [como uma configuração de aplicativo](https://docs.microsoft.com/azure/app-service/web-sites-configure) no aplicativo Web, usando `SQLCONNSTR\_` como um prefixo no nome.</span><span class="sxs-lookup"><span data-stu-id="71cbe-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](https://docs.microsoft.com/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="71cbe-291">**Salve** as configurações do aplicativo Web e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="71cbe-292">Habilitar o dimensionamento automático no Azure global</span><span class="sxs-lookup"><span data-stu-id="71cbe-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="71cbe-293">Quando você cria seu aplicativo Web em um ambiente do serviço de aplicativo, ele começa com uma instância.</span><span class="sxs-lookup"><span data-stu-id="71cbe-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="71cbe-294">Você pode escalar horizontalmente automaticamente para adicionar instâncias para fornecer mais recursos de computação para seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="71cbe-295">Da mesma forma, você pode dimensionar e reduzir automaticamente o número de instâncias de que seu aplicativo precisa.</span><span class="sxs-lookup"><span data-stu-id="71cbe-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="71cbe-296">Você precisa ter um plano do serviço de aplicativo para configurar o scale out e reduzir horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="71cbe-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="71cbe-297">Se você não tiver um plano, crie um antes de iniciar as próximas etapas.</span><span class="sxs-lookup"><span data-stu-id="71cbe-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="71cbe-298">Habilitar expansão automática</span><span class="sxs-lookup"><span data-stu-id="71cbe-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="71cbe-299">No Azure, localize o plano do serviço de aplicativo para os sites que você deseja escalar horizontalmente e selecione **escalar horizontalmente (plano do serviço de aplicativo)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Escalar horizontalmente Azure App serviço](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="71cbe-301">Selecione **Habilitar dimensionamento automático**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-301">Select **Enable autoscale**.</span></span>

    ![Habilitar dimensionamento automático no serviço Azure App](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="71cbe-303">Insira um nome para o **nome da configuração de dimensionamento automático**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="71cbe-304">Para a regra de dimensionamento automático **padrão** , selecione **escala com base em uma métrica**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="71cbe-305">Defina os **limites da instância** como **mínimo: 1**, **máximo: 10**e **padrão: 1**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurar o dimensionamento automático no serviço Azure App](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="71cbe-307">Selecione **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="71cbe-308">Em **origem da métrica**, selecione **recurso atual**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="71cbe-309">Use os critérios e as ações a seguir para a regra.</span><span class="sxs-lookup"><span data-stu-id="71cbe-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="71cbe-310">Critérios</span><span class="sxs-lookup"><span data-stu-id="71cbe-310">Criteria</span></span>

1. <span data-ttu-id="71cbe-311">Em **agregação de tempo,** selecione **média**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="71cbe-312">Em **nome da métrica**, selecione **percentual de CPU**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="71cbe-313">Em **operador**, selecione **maior que**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="71cbe-314">Defina o **limite** como **50**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="71cbe-315">Defina a **duração** como **10**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="71cbe-316">Ação</span><span class="sxs-lookup"><span data-stu-id="71cbe-316">Action</span></span>

1. <span data-ttu-id="71cbe-317">Em **operação**, selecione **aumentar contagem por**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="71cbe-318">Defina a **contagem de instâncias** como **2**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="71cbe-319">Defina o **resfriamento** como **5**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="71cbe-320">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-320">Select **Add**.</span></span>

5. <span data-ttu-id="71cbe-321">Selecione **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="71cbe-322">Em **origem da métrica**, selecione **recurso atual.**</span><span class="sxs-lookup"><span data-stu-id="71cbe-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="71cbe-323">O recurso atual conterá o nome/GUID do plano do serviço de aplicativo e as listas suspensas **tipo de recurso** e **recurso** não estarão disponíveis.</span><span class="sxs-lookup"><span data-stu-id="71cbe-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="71cbe-324">Habilitar escala automática em</span><span class="sxs-lookup"><span data-stu-id="71cbe-324">Enable automatic scale in</span></span>

<span data-ttu-id="71cbe-325">Quando o tráfego diminui, o aplicativo Web do Azure pode reduzir automaticamente o número de instâncias ativas para reduzir os custos.</span><span class="sxs-lookup"><span data-stu-id="71cbe-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="71cbe-326">Essa ação é menos agressiva do que a expansão e minimiza o impacto nos usuários do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="71cbe-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="71cbe-327">Vá para a condição de expansão **padrão** e, em seguida, selecione **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="71cbe-328">Use os critérios e as ações a seguir para a regra.</span><span class="sxs-lookup"><span data-stu-id="71cbe-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="71cbe-329">Critérios</span><span class="sxs-lookup"><span data-stu-id="71cbe-329">Criteria</span></span>

1. <span data-ttu-id="71cbe-330">Em **agregação de tempo,** selecione **média**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="71cbe-331">Em **nome da métrica**, selecione **percentual de CPU**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="71cbe-332">Em **operador**, selecione **menor que**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="71cbe-333">Defina o **limite** como **30**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="71cbe-334">Defina a **duração** como **10**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="71cbe-335">Ação</span><span class="sxs-lookup"><span data-stu-id="71cbe-335">Action</span></span>

1. <span data-ttu-id="71cbe-336">Em **operação**, selecione **diminuir contagem por**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="71cbe-337">Defina a **contagem de instâncias** como **1**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="71cbe-338">Defina o **resfriamento** como **5**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="71cbe-339">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="71cbe-340">Criar um perfil do Gerenciador de tráfego e configurar o dimensionamento entre nuvens</span><span class="sxs-lookup"><span data-stu-id="71cbe-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="71cbe-341">Crie um perfil do Gerenciador de tráfego no Azure e, em seguida, configure pontos de extremidade para habilitar o dimensionamento entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="71cbe-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="71cbe-342">Criar perfil do Gerenciador de tráfego</span><span class="sxs-lookup"><span data-stu-id="71cbe-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="71cbe-343">Selecione **Criar um recurso**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="71cbe-344">Selecione **rede**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-344">Select **Networking**.</span></span>
3. <span data-ttu-id="71cbe-345">Selecione **perfil do Gerenciador de tráfego** e defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="71cbe-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="71cbe-346">Em **nome**, insira um nome para o seu perfil.</span><span class="sxs-lookup"><span data-stu-id="71cbe-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="71cbe-347">Esse nome **deve** ser exclusivo na zona trafficmanager.net e é usado para criar um novo nome DNS (por exemplo, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="71cbe-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="71cbe-348">Para o **método de roteamento**, selecione o **peso**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="71cbe-349">Para **assinatura**, selecione a assinatura na qual você deseja criar esse perfil.</span><span class="sxs-lookup"><span data-stu-id="71cbe-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="71cbe-350">Em **grupo de recursos**, crie um novo grupo de recursos para esse perfil.</span><span class="sxs-lookup"><span data-stu-id="71cbe-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="71cbe-351">Em **Local do grupo de recursos**, selecione o local do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="71cbe-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="71cbe-352">Essa configuração refere-se ao local do grupo de recursos e não tem impacto sobre o perfil do Gerenciador de tráfego implantado globalmente.</span><span class="sxs-lookup"><span data-stu-id="71cbe-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="71cbe-353">Selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-353">Select **Create**.</span></span>

    ![Criar perfil do Gerenciador de tráfego](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="71cbe-355">Quando a implantação global do seu perfil do Gerenciador de tráfego for concluída, ela será mostrada na lista de recursos para o grupo de recursos no qual você o criou.</span><span class="sxs-lookup"><span data-stu-id="71cbe-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="71cbe-356">Adicionar pontos de extremidade do Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="71cbe-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="71cbe-357">Pesquise o perfil do Gerenciador de tráfego que você criou.</span><span class="sxs-lookup"><span data-stu-id="71cbe-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="71cbe-358">Se você navegou até o grupo de recursos para o perfil, selecione o perfil.</span><span class="sxs-lookup"><span data-stu-id="71cbe-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="71cbe-359">No **perfil do Gerenciador de tráfego**, em **configurações**, selecione **pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="71cbe-360">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-360">Select **Add**.</span></span>

4. <span data-ttu-id="71cbe-361">Em **Adicionar ponto de extremidade**, use as seguintes configurações para Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="71cbe-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="71cbe-362">Para **tipo**, selecione **ponto de extremidade externo**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="71cbe-363">Insira um **nome** para o ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="71cbe-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="71cbe-364">Para **nome de domínio totalmente qualificado (FQDN) ou IP**, insira a URL externa para seu aplicativo Web de Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="71cbe-365">Para **peso**, mantenha o padrão, **1**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="71cbe-366">Esse peso resulta em todo o tráfego indo para esse ponto de extremidade se ele estiver íntegro.</span><span class="sxs-lookup"><span data-stu-id="71cbe-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="71cbe-367">Deixe **Adicionar como desabilitado** desmarcado.</span><span class="sxs-lookup"><span data-stu-id="71cbe-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="71cbe-368">Selecione **OK** para salvar o ponto de extremidade do Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="71cbe-369">Você configurará o ponto de extremidade do Azure em seguida.</span><span class="sxs-lookup"><span data-stu-id="71cbe-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="71cbe-370">No **perfil do Gerenciador de tráfego**, selecione **pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="71cbe-371">Selecione **+ Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-371">Select **+Add**.</span></span>
3. <span data-ttu-id="71cbe-372">Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure:</span><span class="sxs-lookup"><span data-stu-id="71cbe-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="71cbe-373">Para **tipo**, selecione **ponto de extremidade do Azure**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="71cbe-374">Insira um **nome** para o ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="71cbe-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="71cbe-375">Para **tipo de recurso de destino**, selecione **serviço de aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="71cbe-376">Para **recurso de destino**, selecione **escolher um serviço de aplicativo** para ver uma lista de aplicativos Web na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="71cbe-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="71cbe-377">Em **Recursos**, escolha o Serviço de Aplicativo que deseja adicionar como o primeiro ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="71cbe-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="71cbe-378">Para **peso**, selecione **2**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="71cbe-379">Essa configuração resulta em todo o tráfego indo para esse ponto de extremidade se o ponto de extremidade primário não estiver íntegro ou se você tiver uma regra/alerta que redireciona o tráfego quando disparado.</span><span class="sxs-lookup"><span data-stu-id="71cbe-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="71cbe-380">Deixe **Adicionar como desabilitado** desmarcado.</span><span class="sxs-lookup"><span data-stu-id="71cbe-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="71cbe-381">Selecione **OK** para salvar o ponto de extremidade do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="71cbe-382">Depois que os dois pontos de extremidade são configurados, eles são listados no **perfil do Gerenciador de tráfego** quando você seleciona pontos de **extremidade**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="71cbe-383">O exemplo na captura de tela a seguir mostra dois pontos de extremidade, com status e informações de configuração para cada um.</span><span class="sxs-lookup"><span data-stu-id="71cbe-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Pontos de extremidade no perfil do Gerenciador de tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="71cbe-385">Configurar Application Insights monitoramento e alertas</span><span class="sxs-lookup"><span data-stu-id="71cbe-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="71cbe-386">Aplicativo Azure insights permite monitorar seu aplicativo e enviar alertas com base nas condições que você configurar.</span><span class="sxs-lookup"><span data-stu-id="71cbe-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="71cbe-387">Alguns exemplos são: o aplicativo está indisponível, está apresentando falhas ou está mostrando problemas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="71cbe-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="71cbe-388">Você usará Application Insights métricas para criar alertas.</span><span class="sxs-lookup"><span data-stu-id="71cbe-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="71cbe-389">Quando esses alertas forem disparados, a instância do aplicativo Web mudará automaticamente do Hub Azure Stack para o Azure para escalar horizontalmente e, em seguida, de volta para o Hub de Azure Stack para reduzir horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="71cbe-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="71cbe-390">Crie um alerta de métricas</span><span class="sxs-lookup"><span data-stu-id="71cbe-390">Create an alert from metrics</span></span>

<span data-ttu-id="71cbe-391">Vá para o grupo de recursos deste tutorial e selecione a instância de Application Insights para abrir **Application insights**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="71cbe-393">Você usará essa exibição para criar um alerta de expansão e um alerta de redução.</span><span class="sxs-lookup"><span data-stu-id="71cbe-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="71cbe-394">Criar o alerta de expansão</span><span class="sxs-lookup"><span data-stu-id="71cbe-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="71cbe-395">Em **Configurar**, selecione **alertas (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="71cbe-396">Selecione **adicionar alerta de métrica (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="71cbe-397">Em **Adicionar regra**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="71cbe-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="71cbe-398">Para **nome**, insira **intermitência na nuvem do Azure**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="71cbe-399">Uma **Descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="71cbe-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="71cbe-400">Em **Source**  >  **alerta de origem em**, selecione **métricas**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="71cbe-401">Em **critérios**, selecione sua assinatura, o grupo de recursos para seu perfil do Gerenciador de tráfego e o nome do perfil do Gerenciador de tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="71cbe-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="71cbe-402">Para **métrica**, selecione **taxa de solicitação**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="71cbe-403">Para **condição**, selecione **maior que**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="71cbe-404">Para **limite**, digite **2**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="71cbe-405">Para **período**, selecione **nos últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="71cbe-406">Em **notificar via**:</span><span class="sxs-lookup"><span data-stu-id="71cbe-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="71cbe-407">Marque a caixa de seleção de **proprietários, colaboradores e leitores de email**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="71cbe-408">Insira seu endereço de email para **emails de administrador adicionais**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="71cbe-409">Na barra de menus, selecione **salvar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="71cbe-410">Criar o alerta de redução</span><span class="sxs-lookup"><span data-stu-id="71cbe-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="71cbe-411">Em **Configurar**, selecione **alertas (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="71cbe-412">Selecione **adicionar alerta de métrica (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="71cbe-413">Em **Adicionar regra**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="71cbe-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="71cbe-414">Para **nome**, digite **dimensionar novamente no Hub de Azure Stack**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="71cbe-415">Uma **Descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="71cbe-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="71cbe-416">Em **Source**  >  **alerta de origem em**, selecione **métricas**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="71cbe-417">Em **critérios**, selecione sua assinatura, o grupo de recursos para seu perfil do Gerenciador de tráfego e o nome do perfil do Gerenciador de tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="71cbe-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="71cbe-418">Para **métrica**, selecione **taxa de solicitação**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="71cbe-419">Para **condição**, selecione **menor que**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="71cbe-420">Para **limite**, digite **2**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="71cbe-421">Para **período**, selecione **nos últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="71cbe-422">Em **notificar via**:</span><span class="sxs-lookup"><span data-stu-id="71cbe-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="71cbe-423">Marque a caixa de seleção de **proprietários, colaboradores e leitores de email**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="71cbe-424">Insira seu endereço de email para **emails de administrador adicionais**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="71cbe-425">Na barra de menus, selecione **salvar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="71cbe-426">A captura de tela a seguir mostra os alertas de expansão e redução horizontal.</span><span class="sxs-lookup"><span data-stu-id="71cbe-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alertas de Application Insights (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="71cbe-428">Redirecionar o tráfego entre o Azure e o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-429">Você pode configurar a alternância manual ou automática de seu tráfego de aplicativo Web entre o Azure e o Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="71cbe-430">Configurar a alternância manual entre o Azure e o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-431">Quando o site atingir os limites que você configurar, você receberá um alerta.</span><span class="sxs-lookup"><span data-stu-id="71cbe-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="71cbe-432">Use as etapas a seguir para redirecionar manualmente o tráfego para o Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="71cbe-433">No portal do Azure, selecione seu perfil do Gerenciador de tráfego.</span><span class="sxs-lookup"><span data-stu-id="71cbe-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Pontos de extremidade do Gerenciador de tráfego no portal do Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="71cbe-435">Selecione **Pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="71cbe-436">Selecione o **ponto de extremidade do Azure**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="71cbe-437">Em **status**, selecione **habilitado**e, em seguida, selecione **salvar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Habilitar o ponto de extremidade do Azure no portal do Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="71cbe-439">Em **pontos** de extremidade para o perfil do Gerenciador de tráfego, selecione **ponto de extremidade externo**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="71cbe-440">Em **status**, selecione **desabilitado**e, em seguida, selecione **salvar**.</span><span class="sxs-lookup"><span data-stu-id="71cbe-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Desabilitar ponto de extremidade do hub de Azure Stack em portal do Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="71cbe-442">Depois que os pontos de extremidade são configurados, o tráfego do aplicativo vai para o aplicativo Web de expansão do Azure em vez do aplicativo Web do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Pontos de extremidade alterados no tráfego do aplicativo Web do Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="71cbe-444">Para reverter o fluxo de volta para Azure Stack Hub, use as etapas anteriores para:</span><span class="sxs-lookup"><span data-stu-id="71cbe-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="71cbe-445">Habilite o ponto de extremidade do hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="71cbe-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="71cbe-446">Desabilite o ponto de extremidade do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="71cbe-447">Configurar a alternância automática entre o Azure e o Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="71cbe-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="71cbe-448">Você também pode usar o monitoramento de Application Insights se seu aplicativo for executado em um ambiente sem [servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelo Azure functions.</span><span class="sxs-lookup"><span data-stu-id="71cbe-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="71cbe-449">Nesse cenário, você pode configurar Application Insights para usar um webhook que chama um aplicativo de funções.</span><span class="sxs-lookup"><span data-stu-id="71cbe-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="71cbe-450">Esse aplicativo habilita ou desabilita automaticamente um ponto de extremidade em resposta a um alerta.</span><span class="sxs-lookup"><span data-stu-id="71cbe-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="71cbe-451">Use as etapas a seguir como um guia para configurar a alternância automática de tráfego.</span><span class="sxs-lookup"><span data-stu-id="71cbe-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="71cbe-452">Crie um aplicativo de funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="71cbe-453">Crie uma função disparada por HTTP.</span><span class="sxs-lookup"><span data-stu-id="71cbe-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="71cbe-454">Importe os SDKs do Azure para o Gerenciador de recursos, aplicativos Web e Gerenciador de tráfego.</span><span class="sxs-lookup"><span data-stu-id="71cbe-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="71cbe-455">Desenvolver código para:</span><span class="sxs-lookup"><span data-stu-id="71cbe-455">Develop code to:</span></span>

   - <span data-ttu-id="71cbe-456">Autentique em sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="71cbe-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="71cbe-457">Use um parâmetro que alterna os pontos de extremidade do Gerenciador de tráfego para direcionar o tráfego para o Azure ou Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="71cbe-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="71cbe-458">Salve seu código e adicione a URL do aplicativo de funções com os parâmetros apropriados à seção **webhook** do Application insights configurações da regra de alerta.</span><span class="sxs-lookup"><span data-stu-id="71cbe-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="71cbe-459">O tráfego é redirecionado automaticamente quando um alerta de Application Insights é disparado.</span><span class="sxs-lookup"><span data-stu-id="71cbe-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="71cbe-460">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="71cbe-460">Next steps</span></span>

- <span data-ttu-id="71cbe-461">Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="71cbe-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
