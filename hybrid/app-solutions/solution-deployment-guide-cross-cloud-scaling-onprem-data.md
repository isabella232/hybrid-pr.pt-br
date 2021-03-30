---
title: Implantar o aplicativo híbrido com dados locais que escalam entre nuvens
description: Saiba como implantar um aplicativo que usa dados locais e escala entre nuvens usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895390"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="d0450-103">Implantar o aplicativo híbrido com dados locais que escalam entre nuvens</span><span class="sxs-lookup"><span data-stu-id="d0450-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="d0450-104">Este guia de solução mostra como implantar um aplicativo híbrido que abranja o Azure e o Azure Stack Hub e use uma única fonte de dados local.</span><span class="sxs-lookup"><span data-stu-id="d0450-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="d0450-105">Usando uma solução de nuvem híbrida, você pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="d0450-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="d0450-106">Seus desenvolvedores também podem aproveitar o ecossistema de desenvolvimento da Microsoft e aplicar suas habilidades aos ambientes locais e na nuvem.</span><span class="sxs-lookup"><span data-stu-id="d0450-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="d0450-107">Visão geral e suposições</span><span class="sxs-lookup"><span data-stu-id="d0450-107">Overview and assumptions</span></span>

<span data-ttu-id="d0450-108">Siga este tutorial para configurar um fluxo de trabalho que permita aos desenvolvedores implantar um aplicativo Web idêntico em uma nuvem pública e em uma nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="d0450-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="d0450-109">Esse aplicativo pode acessar uma rede roteável fora da Internet que esteja hospedada na nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="d0450-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="d0450-110">Esses aplicativos Web são monitorados e, quando há um pico de tráfego, um programa modifica os registros DNS para redirecionar o tráfego para a nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="d0450-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="d0450-111">Quando o tráfego cai para o nível anterior ao pico, ele é roteado de volta para a nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="d0450-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="d0450-112">Este tutorial cobre as seguintes tarefas:</span><span class="sxs-lookup"><span data-stu-id="d0450-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d0450-113">Implante um servidor de banco de dados do SQL Server com conexão híbrida.</span><span class="sxs-lookup"><span data-stu-id="d0450-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="d0450-114">Conecte um aplicativo Web no Azure global a uma rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="d0450-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="d0450-115">Configure o DNS para escala entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="d0450-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d0450-116">Configure certificados SSL para escala entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="d0450-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d0450-117">Configure e implante o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="d0450-118">Crie um perfil do Gerenciador de Tráfego e configure-o para escala entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="d0450-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d0450-119">Configure o monitoramento e os alertas do Application Insights para aumentar o tráfego.</span><span class="sxs-lookup"><span data-stu-id="d0450-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="d0450-120">Configure a alternância automática de tráfego entre o Azure global e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="d0450-121">![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d0450-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d0450-122">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d0450-123">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="d0450-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d0450-124">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="d0450-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d0450-125">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="d0450-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="d0450-126">Suposições</span><span class="sxs-lookup"><span data-stu-id="d0450-126">Assumptions</span></span>

<span data-ttu-id="d0450-127">Este tutorial pressupõe que você tenha um conhecimento básico do Azure global e do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d0450-128">Se você quiser saber mais antes de iniciar o tutorial, confira estes artigos:</span><span class="sxs-lookup"><span data-stu-id="d0450-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="d0450-129">Introdução ao Azure</span><span class="sxs-lookup"><span data-stu-id="d0450-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="d0450-130">Conceitos de chave do Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

<span data-ttu-id="d0450-131">Este tutorial pressupõe que você tenha uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="d0450-132">Se você não tem uma assinatura, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="d0450-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d0450-133">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="d0450-133">Prerequisites</span></span>

<span data-ttu-id="d0450-134">Antes de iniciar esta solução, verifique se você atende aos seguintes requisitos:</span><span class="sxs-lookup"><span data-stu-id="d0450-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="d0450-135">Um ASDK (Kit de Desenvolvimento do Azure Stack) ou uma assinatura em um sistema integrado do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="d0450-136">Para implantar o ASDK, siga as instruções em [Implantar o ASDK usando o instalador](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="d0450-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install).</span></span>
- <span data-ttu-id="d0450-137">A instalação do Azure Stack Hub deve ter instalado:</span><span class="sxs-lookup"><span data-stu-id="d0450-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="d0450-138">O Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-138">The Azure App Service.</span></span> <span data-ttu-id="d0450-139">Trabalhe com seu operador do Azure Stack Hub para implantar e configurar o Serviço de Aplicativo do Azure no seu ambiente.</span><span class="sxs-lookup"><span data-stu-id="d0450-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="d0450-140">Este tutorial requer que o Serviço de Aplicativo tenha pelo menos uma (1) função de trabalho dedicada disponível.</span><span class="sxs-lookup"><span data-stu-id="d0450-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="d0450-141">Uma imagem do Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="d0450-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="d0450-142">Um Windows Server 2016 com uma imagem do Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d0450-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="d0450-143">Os planos e as ofertas apropriados.</span><span class="sxs-lookup"><span data-stu-id="d0450-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="d0450-144">Um nome de domínio para seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-144">A domain name for your web app.</span></span> <span data-ttu-id="d0450-145">Se você não tem um, é possível comprá-lo de um provedor de domínio, como GoDaddy, Bluehost e InMotion.</span><span class="sxs-lookup"><span data-stu-id="d0450-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="d0450-146">Um certificado SSL para seu domínio de uma autoridade de certificação confiável, como o LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="d0450-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="d0450-147">Um aplicativo Web que se comunica com um banco de dados do SQL Server e oferece suporte ao Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d0450-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="d0450-148">Você pode baixar o aplicativo de exemplo [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0450-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="d0450-149">Uma rede híbrida entre uma rede virtual do Azure e outra do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="d0450-150">Para obter instruções detalhadas, confira [Configurar a conectividade de nuvem híbrida com o Azure e o Azure Stack Hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="d0450-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="d0450-151">Um pipeline de CI/CD (integração contínua/implantação contínua) com um agente de compilação particular no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="d0450-152">Para obter instruções detalhadas, confira [Configurar a identidade de nuvem híbrida com os aplicativos do Azure e do Azure Stack Hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="d0450-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="d0450-153">Implantar um servidor de banco de dados do SQL Server com conexão híbrida</span><span class="sxs-lookup"><span data-stu-id="d0450-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="d0450-154">Entre no portal do usuário do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="d0450-155">No **Dashboard**, selecione **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="d0450-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Marketplace do Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="d0450-157">Em **Marketplace**, selecione **Computação** e, em seguida, escolha **Mais**.</span><span class="sxs-lookup"><span data-stu-id="d0450-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="d0450-158">Em **Mais**, selecione a **Licença gratuita do SQL Server: Imagem do SQL Server 2017 Developer no Windows Server**.</span><span class="sxs-lookup"><span data-stu-id="d0450-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Selecionar uma imagem de máquina virtual no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="d0450-160">Em **Licença gratuita do SQL Server: Em SQL Server 2017 Developer no Windows Server**, selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="d0450-161">Em **Noções básicas > Definir configurações básicas**, forneça um **Nome** para a VM (máquina virtual), um **Nome de usuário** para o SA do SQL Server e uma **Senha** para o SA.</span><span class="sxs-lookup"><span data-stu-id="d0450-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="d0450-162">Na lista suspensa **Assinatura**, selecione a assinatura na qual você está efetuando a implantação.</span><span class="sxs-lookup"><span data-stu-id="d0450-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="d0450-163">Para **Grupo de recursos**, use **Escolher existente** e coloque a VM no mesmo grupo de recursos que o seu aplicativo Web do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Definir as configurações básicas para VM no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="d0450-165">Em **Tamanho**, escolha um para a sua VM.</span><span class="sxs-lookup"><span data-stu-id="d0450-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="d0450-166">Para este tutorial, recomendamos A2_Standard ou DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="d0450-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="d0450-167">Em **Configurações > Configurar recursos opcionais**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="d0450-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="d0450-168">**Conta de armazenamento**: se precisar, crie outra conta.</span><span class="sxs-lookup"><span data-stu-id="d0450-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d0450-169">**Rede virtual**:</span><span class="sxs-lookup"><span data-stu-id="d0450-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="d0450-170">verifique se sua VM do SQL Server está implantada na mesma rede virtual que os gateways de VPN.</span><span class="sxs-lookup"><span data-stu-id="d0450-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="d0450-171">**Endereço IP público**: use as configurações padrão.</span><span class="sxs-lookup"><span data-stu-id="d0450-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="d0450-172">**Grupo de segurança de rede**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="d0450-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="d0450-173">Crie um NSG.</span><span class="sxs-lookup"><span data-stu-id="d0450-173">Create a new NSG.</span></span>
   - <span data-ttu-id="d0450-174">**Extensões e Monitoramento**: Mantenha as configurações padrão.</span><span class="sxs-lookup"><span data-stu-id="d0450-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="d0450-175">**Conta de armazenamento de diagnóstico**: se precisar, crie outra conta.</span><span class="sxs-lookup"><span data-stu-id="d0450-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d0450-176">Selecione **OK** para salvar a configuração.</span><span class="sxs-lookup"><span data-stu-id="d0450-176">Select **OK** to save your configuration.</span></span>

     ![Configurar recursos opcionais de VM no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="d0450-178">Em **Configurações do SQL Server**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="d0450-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="d0450-179">Em **Conectividade do SQL**, selecione **Pública (Internet)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="d0450-180">Em **Porta**, mantenha o padrão **1433**.</span><span class="sxs-lookup"><span data-stu-id="d0450-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="d0450-181">Em **Autenticação do SQL**, selecione **Habilitar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="d0450-182">Quando você habilita a autenticação do SQL, ela deve preencher automaticamente as informações de “SQLAdmin” que você configurou em **Noções básicas**.</span><span class="sxs-lookup"><span data-stu-id="d0450-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="d0450-183">Para o restante das configurações, mantenha as opções padrão.</span><span class="sxs-lookup"><span data-stu-id="d0450-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="d0450-184">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="d0450-184">Select **OK**.</span></span>

     ![Definir as configurações básicas do SQL Server no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="d0450-186">Em **Resumo**, examine a configuração da VM e, em seguida, selecione **OK** para iniciar a implantação.</span><span class="sxs-lookup"><span data-stu-id="d0450-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Resumo das configurações no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="d0450-188">A criação da VM leva algum tempo.</span><span class="sxs-lookup"><span data-stu-id="d0450-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="d0450-189">Você pode exibir o STATUS de suas VMs em **Máquinas virtuais**.</span><span class="sxs-lookup"><span data-stu-id="d0450-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Status das máquinas virtuais no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="d0450-191">Criar aplicativos Web no Azure e no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d0450-192">O Serviço de Aplicativo do Azure simplifica a execução e o gerenciamento de um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="d0450-193">Como o Azure Stack Hub é consistente com o Azure, o Serviço de Aplicativo pode ser executado em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="d0450-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="d0450-194">Você usará o Serviço de Aplicativo para hospedar seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="d0450-195">Criar aplicativos Web</span><span class="sxs-lookup"><span data-stu-id="d0450-195">Create web apps</span></span>

1. <span data-ttu-id="d0450-196">Crie um aplicativo Web no Azure seguindo as instruções em [Gerenciar um Plano do Serviço de Aplicativo no Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="d0450-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="d0450-197">Coloque o aplicativo Web na mesma assinatura e defina o grupo de recursos como sua rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="d0450-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="d0450-198">Repita a etapa anterior (1) no Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d0450-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="d0450-199">Adicionar rota para o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="d0450-200">É necessário que o Serviço de Aplicativo no Azure Stack Hub possa ser roteado da Internet pública para permitir que os usuários acessem seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="d0450-201">Caso o Azure Stack Hub esteja acessível pela Internet, anote o endereço IP ou a URL voltada para o público para o aplicativo Web do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="d0450-202">Se você está usando um ASDK, é possível [configurar um mapeamento NAT estático](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o Serviço de Aplicativo fora do ambiente virtual.</span><span class="sxs-lookup"><span data-stu-id="d0450-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="d0450-203">Conectar um aplicativo Web no Azure a uma rede híbrida</span><span class="sxs-lookup"><span data-stu-id="d0450-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="d0450-204">Para fornecer conectividade entre o front-end da Web no Azure e o banco de dados do SQL Server no Azure Stack Hub, é necessário que o aplicativo Web esteja conectado à rede híbrida entre o Azure e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d0450-205">Para habilitar a conectividade, você terá que:</span><span class="sxs-lookup"><span data-stu-id="d0450-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="d0450-206">Configurar conectividade ponto a site.</span><span class="sxs-lookup"><span data-stu-id="d0450-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="d0450-207">Configurar o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-207">Configure the web app.</span></span>
- <span data-ttu-id="d0450-208">Modificar o gateway de rede local no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="d0450-209">Configurar a rede virtual do Azure para conectividade ponto a site</span><span class="sxs-lookup"><span data-stu-id="d0450-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="d0450-210">É necessário que o gateway de rede virtual no lado do Azure da rede híbrida permita que conexões ponto a site sejam integradas ao Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="d0450-211">No portal do Azure, acesse a página do gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="d0450-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="d0450-212">Em **Configurações**, selecione **Configuração ponto a site**.</span><span class="sxs-lookup"><span data-stu-id="d0450-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opção ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="d0450-214">Selecione **Configurar agora** para configurar o ponto a site.</span><span class="sxs-lookup"><span data-stu-id="d0450-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Iniciar a configuração ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="d0450-216">Na página de configuração **Ponto a site**, insira o intervalo de endereços IP privado que você deseja usar no **Pool de endereços**.</span><span class="sxs-lookup"><span data-stu-id="d0450-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="d0450-217">Verifique se o intervalo especificado não se sobrepõe a nenhum intervalo de endereços já usado por sub-redes nos componentes globais da rede híbrida do Azure ou do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="d0450-218">Em **Tipo de Túnel**, desmarque **IKEv2 VPN**.</span><span class="sxs-lookup"><span data-stu-id="d0450-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="d0450-219">Selecione **Salvar** para concluir a configuração de ponto a site.</span><span class="sxs-lookup"><span data-stu-id="d0450-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Configurações de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="d0450-221">Integrar o aplicativo do Serviço de Aplicativo do Azure com a rede híbrida</span><span class="sxs-lookup"><span data-stu-id="d0450-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="d0450-222">Para conectar o aplicativo à VNet do Azure, siga as instruções em [Integração VNet exigida pelo gateway](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="d0450-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="d0450-223">Acesse **Configurações** para ver o plano do Serviço de Aplicativo que hospeda o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="d0450-224">Em **Configurações**, selecione **Rede**.</span><span class="sxs-lookup"><span data-stu-id="d0450-224">In **Settings**, select **Networking**.</span></span>

    ![Configurar a Rede para o Plano do Serviço de Aplicativo](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="d0450-226">Em **Integração VNET**, selecione **Clique aqui para gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Gerenciar a integração VNET para o Plano do Serviço de Aplicativo](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="d0450-228">Selecione o VNET que você deseja configurar.</span><span class="sxs-lookup"><span data-stu-id="d0450-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="d0450-229">Em **ENDEREÇOS IP ROTEADOS PARA VNET**, insira o intervalo de endereços IP para a VNet do Azure, a do Azure Stack Hub e os espaços de endereço ponto a site.</span><span class="sxs-lookup"><span data-stu-id="d0450-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="d0450-230">Selecione **Salvar** para validar e salvar essas configurações.</span><span class="sxs-lookup"><span data-stu-id="d0450-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalos de endereços IP a serem roteados na integração da Rede Virtual](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="d0450-232">Para saber mais sobre como o Serviço de Aplicativo se integra às VNets do Azure, confira [Integrar seu aplicativo a uma Rede Virtual do Azure](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="d0450-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="d0450-233">Configurar a rede virtual do Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="d0450-234">O gateway de rede local na rede virtual do Azure Stack Hub precisa ser configurado para rotear o tráfego pelo intervalo de endereços ponto a site do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="d0450-235">No portal do Azure Stack Hub, acesse **Gateway de rede local**.</span><span class="sxs-lookup"><span data-stu-id="d0450-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="d0450-236">Em **Configurações**, escolha **Configuração**.</span><span class="sxs-lookup"><span data-stu-id="d0450-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opção de configuração de gateway no gateway de rede local do Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="d0450-238">Em **Espaço de endereço**, insira o intervalo de endereços ponto a site para o gateway de rede virtual no Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Espaço de endereço ponto a site no gateway de rede local do Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="d0450-240">Selecione **Salvar** para validar e salvar a configuração.</span><span class="sxs-lookup"><span data-stu-id="d0450-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="d0450-241">Configurar o DNS para escala entre nuvens</span><span class="sxs-lookup"><span data-stu-id="d0450-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="d0450-242">Ao configurar corretamente o DNS para aplicativos entre nuvens, os usuários podem acessar as instâncias globais do Azure e do Azure Stack Hub do seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="d0450-243">A configuração de DNS para este tutorial também permite que o Gerenciador de Tráfego do Azure roteie o tráfego quando a carga aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="d0450-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="d0450-244">Este tutorial usa o DNS do Azure para gerenciar o DNS porque os domínios do Serviço de Aplicativo não funcionarão.</span><span class="sxs-lookup"><span data-stu-id="d0450-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="d0450-245">Criar subdomínios</span><span class="sxs-lookup"><span data-stu-id="d0450-245">Create subdomains</span></span>

<span data-ttu-id="d0450-246">Como o Gerenciador de Tráfego depende de CNAMEs de DNS, um subdomínio é necessário para rotear corretamente o tráfego para os pontos de extremidade.</span><span class="sxs-lookup"><span data-stu-id="d0450-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="d0450-247">Para obter mais informações sobre os registros DNS e o mapeamento de domínio, confira [mapear domínios com o Gerenciador de Tráfego](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="d0450-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="d0450-248">Para o ponto de extremidade do Azure, você criará um subdomínio que os usuários possam usar para acessar seu aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="d0450-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="d0450-249">Neste tutorial, você pode usar **app.northwind.com**, mas deve personalizar esse valor com base no seu domínio.</span><span class="sxs-lookup"><span data-stu-id="d0450-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="d0450-250">Também será necessário criar um subdomínio com um registro A para o ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="d0450-251">Você pode usar **azurestack.northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="d0450-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="d0450-252">Configurar um domínio personalizado no Azure</span><span class="sxs-lookup"><span data-stu-id="d0450-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="d0450-253">Adicione o nome do host **app.northwind.com** ao aplicativo Web do Azure [mapeando um CNAME para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="d0450-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="d0450-254">Configurar domínios personalizados no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="d0450-255">Adicione o nome do host **azurestack.northwind.com** ao aplicativo Web do Azure Stack Hub [mapeando um registro A para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="d0450-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="d0450-256">Use o endereço IP roteável da Internet para o aplicativo do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="d0450-257">Adicione o nome do host **app.northwind.com** ao aplicativo Web do Azure Stack Hub [mapeando um CNAME para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="d0450-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="d0450-258">Use o nome de host que você configurou na etapa anterior (1) como o destino para o CNAME.</span><span class="sxs-lookup"><span data-stu-id="d0450-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="d0450-259">Configure certificados SSL para escala entre nuvens</span><span class="sxs-lookup"><span data-stu-id="d0450-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="d0450-260">É importante garantir que os dados confidenciais coletados pelo seu aplicativo Web estejam protegidos enquanto estão em trânsito e quando são armazenados no banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="d0450-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="d0450-261">Você configurará seus aplicativos Web do Azure e do Azure Stack Hub para usar certificados SSL para todo o tráfego de entrada.</span><span class="sxs-lookup"><span data-stu-id="d0450-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="d0450-262">Adicionar SSL ao Azure e ao Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d0450-263">Para adicionar SSL ao Azure:</span><span class="sxs-lookup"><span data-stu-id="d0450-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="d0450-264">Verifique se o certificado SSL obtido é válido para o subdomínio que você criou.</span><span class="sxs-lookup"><span data-stu-id="d0450-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="d0450-265">Não há problema em usar certificados curinga.</span><span class="sxs-lookup"><span data-stu-id="d0450-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="d0450-266">No portal do Azure, siga as instruções nas seções **Preparar o aplicativo Web** e **Associar certificado SSL** do artigo [Associar um certificado SSL personalizado existente a aplicativos Web do Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="d0450-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="d0450-267">Selecione **SSL baseado em SNI** como o **Tipo de SSL**.</span><span class="sxs-lookup"><span data-stu-id="d0450-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="d0450-268">Redirecione todo o tráfego para a porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="d0450-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="d0450-269">Siga as instruções na seção **Impor HTTPS** do artigo [Associar um certificado SSL personalizado existente a aplicativos Web do Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="d0450-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="d0450-270">Para adicionar SSL ao Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="d0450-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="d0450-271">Repita as etapas de 1 a 3 que você usou para o Azure usando o portal do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="d0450-272">Configurar e implantar o aplicativo Web</span><span class="sxs-lookup"><span data-stu-id="d0450-272">Configure and deploy the web app</span></span>

<span data-ttu-id="d0450-273">Você configurará o código do aplicativo para relatar telemetria para a instância correta do Application Insights e configurará os aplicativos Web com as cadeias de conexão corretas.</span><span class="sxs-lookup"><span data-stu-id="d0450-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="d0450-274">Para saber mais sobre o Application Insights, confira [O que é o Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="d0450-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="d0450-275">Adicionar o Application Insights</span><span class="sxs-lookup"><span data-stu-id="d0450-275">Add Application Insights</span></span>

1. <span data-ttu-id="d0450-276">Abra seu aplicativo Web no Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d0450-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="d0450-277">[Adicione o Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application Insights usa para criar alertas quando o tráfego da Web aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="d0450-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="d0450-278">Configurar cadeias de conexão dinâmicas</span><span class="sxs-lookup"><span data-stu-id="d0450-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="d0450-279">Cada instância do aplicativo Web usará um método diferente para se conectar ao banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="d0450-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="d0450-280">O aplicativo no Azure usa o endereço IP privado da VM do SQL Server, e o aplicativo no Azure Stack Hub usa o endereço IP público da VM do SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d0450-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="d0450-281">Em um sistema integrado do Azure Stack Hub, o endereço IP público não deve ser roteável pela Internet.</span><span class="sxs-lookup"><span data-stu-id="d0450-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="d0450-282">Em um ASDK, o endereço IP público não é roteável fora do ASDK.</span><span class="sxs-lookup"><span data-stu-id="d0450-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="d0450-283">Você pode usar variáveis de ambiente do Serviço de Aplicativo para passar uma cadeia de conexão diferente para cada instância do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="d0450-284">Abra o aplicativo no Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d0450-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="d0450-285">Abra Startup.cs e localize o seguinte bloco de código:</span><span class="sxs-lookup"><span data-stu-id="d0450-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="d0450-286">Substitua o bloco de código anterior pelo código a seguir, que usa uma cadeia de conexão definida no arquivo *appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="d0450-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="d0450-287">Definir as configurações do Serviço de Aplicativo</span><span class="sxs-lookup"><span data-stu-id="d0450-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="d0450-288">Crie cadeias de conexão para o Azure e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d0450-289">As cadeias de caracteres devem ser as mesmas, exceto para os endereços IP que são usados.</span><span class="sxs-lookup"><span data-stu-id="d0450-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="d0450-290">No Azure e no Azure Stack Hub, adicione a cadeia de conexão apropriada [como uma configuração de aplicativo](/azure/app-service/web-sites-configure) no aplicativo Web, usando `SQLCONNSTR\_` como um prefixo no nome.</span><span class="sxs-lookup"><span data-stu-id="d0450-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="d0450-291">**Salve** as configurações do aplicativo Web e reinicie o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="d0450-292">Habilitar a colocação em escala automática no Azure global</span><span class="sxs-lookup"><span data-stu-id="d0450-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="d0450-293">Quando você cria seu aplicativo Web em um ambiente do Serviço de Aplicativo, ele começa com uma instância.</span><span class="sxs-lookup"><span data-stu-id="d0450-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="d0450-294">Você pode escalar horizontalmente para adicionar instâncias e fornecer mais recursos de computação para o seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="d0450-295">Da mesma forma, você pode reduzir horizontalmente o número de instâncias de que seu aplicativo precisa.</span><span class="sxs-lookup"><span data-stu-id="d0450-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="d0450-296">Você precisa ter um Plano do Serviço de Aplicativo para configurar a ação de escalar e reduzir horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="d0450-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="d0450-297">Se você não tem um plano, crie um antes de iniciar as próximas etapas.</span><span class="sxs-lookup"><span data-stu-id="d0450-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="d0450-298">Habilitar a expansão automática</span><span class="sxs-lookup"><span data-stu-id="d0450-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="d0450-299">No portal do Azure, localize o Plano do Serviço de Aplicativo para os sites que você deseja escalar horizontalmente e, em seguida, selecione **Expansão (Plano do Serviço de Aplicativo)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Escalar horizontalmente o Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="d0450-301">Clique em **Habilitar dimensionamento automático**.</span><span class="sxs-lookup"><span data-stu-id="d0450-301">Select **Enable autoscale**.</span></span>

    ![Habilitar o dimensionamento automático no Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="d0450-303">Forneça um nome para o **Nome de configuração do dimensionamento automático**.</span><span class="sxs-lookup"><span data-stu-id="d0450-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="d0450-304">Na regra de dimensionamento automático **Padrão**, selecione **Escala com base em uma métrica**.</span><span class="sxs-lookup"><span data-stu-id="d0450-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="d0450-305">Defina os **Limites da instância** como **Mínimo: 1**, **Máximo: 10** e **Padrão: 1**.</span><span class="sxs-lookup"><span data-stu-id="d0450-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurar o dimensionamento automático no Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="d0450-307">Selecione **+Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="d0450-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="d0450-308">Em **Origem da Métrica**, selecione **Recurso Atual**.</span><span class="sxs-lookup"><span data-stu-id="d0450-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="d0450-309">Use os critérios e as ações a seguir para a regra.</span><span class="sxs-lookup"><span data-stu-id="d0450-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d0450-310">Critérios</span><span class="sxs-lookup"><span data-stu-id="d0450-310">Criteria</span></span>

1. <span data-ttu-id="d0450-311">Em **Agregação de Tempo,** selecione **Média**.</span><span class="sxs-lookup"><span data-stu-id="d0450-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d0450-312">Em **Nome da Métrica**, selecione **Percentual de CPU**.</span><span class="sxs-lookup"><span data-stu-id="d0450-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d0450-313">Em **Operador**, selecione **Maior que**.</span><span class="sxs-lookup"><span data-stu-id="d0450-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="d0450-314">Defina o **Limite** como **50**.</span><span class="sxs-lookup"><span data-stu-id="d0450-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="d0450-315">Defina a **Duração** como **10**.</span><span class="sxs-lookup"><span data-stu-id="d0450-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d0450-316">Ação</span><span class="sxs-lookup"><span data-stu-id="d0450-316">Action</span></span>

1. <span data-ttu-id="d0450-317">Em **Operação**, selecione **Aumentar Contagem por**.</span><span class="sxs-lookup"><span data-stu-id="d0450-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="d0450-318">Defina a **Contagem de Instâncias** como **2**.</span><span class="sxs-lookup"><span data-stu-id="d0450-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="d0450-319">Defina o **Resfriamento** como **5**.</span><span class="sxs-lookup"><span data-stu-id="d0450-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="d0450-320">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-320">Select **Add**.</span></span>

5. <span data-ttu-id="d0450-321">Selecione **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="d0450-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="d0450-322">Em **Origem da Métrica**, selecione **Recurso Atual.**</span><span class="sxs-lookup"><span data-stu-id="d0450-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="d0450-323">O recurso atual conterá o nome/GUID do Plano do Serviço de Aplicativo, e as listas suspensas **Tipo de Recurso** e **Recurso** ficarão indisponíveis.</span><span class="sxs-lookup"><span data-stu-id="d0450-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="d0450-324">Habilitar a redução horizontal automática</span><span class="sxs-lookup"><span data-stu-id="d0450-324">Enable automatic scale in</span></span>

<span data-ttu-id="d0450-325">Quando o tráfego diminui, o aplicativo Web do Azure pode reduzir automaticamente o número de instâncias ativas para reduzir os custos.</span><span class="sxs-lookup"><span data-stu-id="d0450-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="d0450-326">Essa ação é menos agressiva do que a expansão e minimiza o impacto nos usuários do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d0450-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="d0450-327">Acesse a condição de expansão **Padrão** e selecione **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="d0450-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="d0450-328">Use os critérios e as ações a seguir para a regra.</span><span class="sxs-lookup"><span data-stu-id="d0450-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d0450-329">Critérios</span><span class="sxs-lookup"><span data-stu-id="d0450-329">Criteria</span></span>

1. <span data-ttu-id="d0450-330">Em **Agregação de Tempo,** selecione **Média**.</span><span class="sxs-lookup"><span data-stu-id="d0450-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d0450-331">Em **Nome da Métrica**, selecione **Percentual de CPU**.</span><span class="sxs-lookup"><span data-stu-id="d0450-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d0450-332">Em **Operador**, selecione **Menor que**.</span><span class="sxs-lookup"><span data-stu-id="d0450-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="d0450-333">Defina o **Limite** como **30**.</span><span class="sxs-lookup"><span data-stu-id="d0450-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="d0450-334">Defina a **Duração** como **10**.</span><span class="sxs-lookup"><span data-stu-id="d0450-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d0450-335">Ação</span><span class="sxs-lookup"><span data-stu-id="d0450-335">Action</span></span>

1. <span data-ttu-id="d0450-336">Em **Operação**, selecione **Diminuir contagem por**.</span><span class="sxs-lookup"><span data-stu-id="d0450-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="d0450-337">Defina a **Contagem de Instâncias** como **1**.</span><span class="sxs-lookup"><span data-stu-id="d0450-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="d0450-338">Defina o **Resfriamento** como **5**.</span><span class="sxs-lookup"><span data-stu-id="d0450-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="d0450-339">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="d0450-340">Crie um perfil do Gerenciador de Tráfego e configure a escala entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="d0450-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="d0450-341">Crie um perfil do Gerenciador de Tráfego usando o portal do Azure e, em seguida, configure os pontos de extremidade para habilitar a escala entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="d0450-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="d0450-342">Criar perfil do Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="d0450-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="d0450-343">Selecione **Criar um recurso**.</span><span class="sxs-lookup"><span data-stu-id="d0450-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="d0450-344">Selecione **Rede**.</span><span class="sxs-lookup"><span data-stu-id="d0450-344">Select **Networking**.</span></span>
3. <span data-ttu-id="d0450-345">Selecione **perfil do Gerenciador de Tráfego** e defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="d0450-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="d0450-346">Em **Nome**, insira um nome para o seu perfil.</span><span class="sxs-lookup"><span data-stu-id="d0450-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="d0450-347">Esse nome **deve** ser exclusivo na zona trafficmanager.net e usado para criar um nome DNS (por exemplo, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="d0450-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="d0450-348">Para **Método de roteamento**, selecione o **Ponderado**.</span><span class="sxs-lookup"><span data-stu-id="d0450-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="d0450-349">Em **Assinatura**, selecione a aquela na qual deseja criar esse perfil.</span><span class="sxs-lookup"><span data-stu-id="d0450-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="d0450-350">Em **Grupo de Recursos**, crie um grupo de recursos para esse perfil.</span><span class="sxs-lookup"><span data-stu-id="d0450-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="d0450-351">Em **Local do grupo de recursos**, selecione o local do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d0450-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="d0450-352">Essa configuração refere-se ao local do grupo de recursos e não tem impacto no perfil do Gerenciador de Tráfego implantado globalmente.</span><span class="sxs-lookup"><span data-stu-id="d0450-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="d0450-353">Selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-353">Select **Create**.</span></span>

    ![Criar perfil do Gerenciador de Tráfego](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="d0450-355">Quando a implantação global do seu perfil do Gerenciador de Tráfego estiver concluída, ela será mostrada na lista do grupo de recursos no qual você a criou.</span><span class="sxs-lookup"><span data-stu-id="d0450-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="d0450-356">Adicionar pontos de extremidade do Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="d0450-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="d0450-357">Pesquise o perfil do Gerenciador de Tráfego que você criou.</span><span class="sxs-lookup"><span data-stu-id="d0450-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="d0450-358">Se você navegou até o grupo de recursos para pesquisar o perfil, selecione-o.</span><span class="sxs-lookup"><span data-stu-id="d0450-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="d0450-359">No **Perfil do Gerenciador de Tráfego**, em **CONFIGURAÇÕES**, selecione **Ponto de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="d0450-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="d0450-360">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-360">Select **Add**.</span></span>

4. <span data-ttu-id="d0450-361">Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="d0450-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="d0450-362">Para **Tipo**, selecione **Ponto de extremidade externo**.</span><span class="sxs-lookup"><span data-stu-id="d0450-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="d0450-363">Insira um **Nome** para o ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="d0450-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d0450-364">Para **FQDN (Nome de domínio totalmente qualificado) ou IP**, insira a URL externa para o seu aplicativo Web do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="d0450-365">Em **Peso**, mantenha o valor padrão **1**.</span><span class="sxs-lookup"><span data-stu-id="d0450-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="d0450-366">Esse peso faz com que todo o tráfego passe para esse ponto de extremidade caso ele esteja íntegro.</span><span class="sxs-lookup"><span data-stu-id="d0450-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="d0450-367">Deixe a opção **Adicionar como desabilitado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="d0450-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="d0450-368">Selecione **OK** para salvar o ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="d0450-369">Você configurará o ponto de extremidade do Azure em seguida.</span><span class="sxs-lookup"><span data-stu-id="d0450-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="d0450-370">No **perfil do Gerenciador de Tráfego**, selecione **Ponto de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="d0450-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="d0450-371">Selecione **+Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-371">Select **+Add**.</span></span>
3. <span data-ttu-id="d0450-372">Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure:</span><span class="sxs-lookup"><span data-stu-id="d0450-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="d0450-373">Em **Tipo**, selecione **Ponto de extremidade do Azure**.</span><span class="sxs-lookup"><span data-stu-id="d0450-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="d0450-374">Insira um **Nome** para o ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="d0450-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d0450-375">Para **Tipo de recurso de destino**, selecione **Serviço de Aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="d0450-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="d0450-376">Em **Recurso de destino**, selecione **Escolher um serviço de aplicativo** para ver uma lista de aplicativos Web na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="d0450-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="d0450-377">Em **Recursos**, escolha o Serviço de Aplicativo que deseja adicionar como o primeiro ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="d0450-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="d0450-378">Em **Peso**, selecione **2**.</span><span class="sxs-lookup"><span data-stu-id="d0450-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="d0450-379">Essa configuração faz com que todo o tráfego vá para esse ponto de extremidade caso o ponto de extremidade primário não esteja íntegro ou caso você tenha uma regra/alerta que redirecione o tráfego ao ser disparado.</span><span class="sxs-lookup"><span data-stu-id="d0450-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="d0450-380">Deixe a opção **Adicionar como desabilitado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="d0450-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="d0450-381">Selecione **OK** para salvar o ponto de extremidade do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="d0450-382">Depois que todos os pontos de extremidade são configurados, eles são listados no **Perfil do Gerenciador de Tráfego** quando você seleciona os **Pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="d0450-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="d0450-383">O exemplo na captura de tela a seguir mostra dois pontos de extremidade, com status e informações de configuração para cada um deles.</span><span class="sxs-lookup"><span data-stu-id="d0450-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Pontos de extremidades no perfil do Gerenciador de Tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="d0450-385">Configurar o monitoramento e alertas do Application Insights no Azure</span><span class="sxs-lookup"><span data-stu-id="d0450-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="d0450-386">O Application Insights do Azure permite que você monitore seu aplicativo e envie alertas com base nas condições que você configurar.</span><span class="sxs-lookup"><span data-stu-id="d0450-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="d0450-387">Por exemplo: o aplicativo está indisponível, está apresentando falhas ou está mostrando problemas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="d0450-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="d0450-388">Você usará as métricas do Azure Application Insights para criar alertas.</span><span class="sxs-lookup"><span data-stu-id="d0450-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="d0450-389">Quando esses alertas são disparados, a instância do aplicativo Web alterna automaticamente do Azure Stack Hub para o Azure para escalar horizontalmente e, em seguida, alterna novamente para o Azure Stack Hub para reduzir horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="d0450-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="d0450-390">Crie um alerta de métricas</span><span class="sxs-lookup"><span data-stu-id="d0450-390">Create an alert from metrics</span></span>

<span data-ttu-id="d0450-391">No portal do Azure, acesse o grupo de recursos deste tutorial e selecione a instância do Application Insights para abrir o **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="d0450-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="d0450-393">Você usará essa exibição para criar um alerta de expansão e outro de redução.</span><span class="sxs-lookup"><span data-stu-id="d0450-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="d0450-394">Criar o alerta de expansão</span><span class="sxs-lookup"><span data-stu-id="d0450-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="d0450-395">Em **CONFIGURAR**, selecione **Alertas (clássico)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d0450-396">Selecione **adicionar alerta de métrica (clássico)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d0450-397">Em **Adicionar regra**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="d0450-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d0450-398">Em **Nome**, insira **Intermitência no Azure Cloud**.</span><span class="sxs-lookup"><span data-stu-id="d0450-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="d0450-399">A **Descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="d0450-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="d0450-400">Em **Origem** > **Alerta no**, selecione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="d0450-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d0450-401">Em **Critérios**, selecione sua assinatura, o grupo de recursos do seu perfil do Gerenciador de Tráfego e o nome do perfil do Gerenciador de Tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="d0450-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d0450-402">Em **Métrica**, selecione **Taxa de Solicitação**.</span><span class="sxs-lookup"><span data-stu-id="d0450-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d0450-403">Em **Condição**, selecione **Maior que**.</span><span class="sxs-lookup"><span data-stu-id="d0450-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="d0450-404">Em **Limite**, insira **2**.</span><span class="sxs-lookup"><span data-stu-id="d0450-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d0450-405">Em **Período**, selecione **Nos últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="d0450-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d0450-406">Em **Notificar via**:</span><span class="sxs-lookup"><span data-stu-id="d0450-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="d0450-407">Marque a caixa de seleção **Proprietários, colaboradores e leitores de email**.</span><span class="sxs-lookup"><span data-stu-id="d0450-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d0450-408">Insira seu endereço de email em **Emails adicionais do administrador**.</span><span class="sxs-lookup"><span data-stu-id="d0450-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d0450-409">Na barra de menu, selecione **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="d0450-410">Criar o alerta de redução</span><span class="sxs-lookup"><span data-stu-id="d0450-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="d0450-411">Em **CONFIGURAR**, selecione **Alertas (clássico)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d0450-412">Selecione **adicionar alerta de métrica (clássico)** .</span><span class="sxs-lookup"><span data-stu-id="d0450-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d0450-413">Em **Adicionar regra**, defina as seguintes configurações:</span><span class="sxs-lookup"><span data-stu-id="d0450-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d0450-414">Em **Nome**, insira **Escalar de volta para o Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="d0450-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="d0450-415">A **Descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="d0450-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="d0450-416">Em **Origem** > **Alerta no**, selecione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="d0450-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d0450-417">Em **Critérios**, selecione sua assinatura, o grupo de recursos do seu perfil do Gerenciador de Tráfego e o nome do perfil do Gerenciador de Tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="d0450-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d0450-418">Em **Métrica**, selecione **Taxa de Solicitação**.</span><span class="sxs-lookup"><span data-stu-id="d0450-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d0450-419">Em **Condição**, selecione **Menor que**.</span><span class="sxs-lookup"><span data-stu-id="d0450-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="d0450-420">Em **Limite**, insira **2**.</span><span class="sxs-lookup"><span data-stu-id="d0450-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d0450-421">Em **Período**, selecione **Nos últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="d0450-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d0450-422">Em **Notificar via**:</span><span class="sxs-lookup"><span data-stu-id="d0450-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="d0450-423">Marque a caixa de seleção **Proprietários, colaboradores e leitores de email**.</span><span class="sxs-lookup"><span data-stu-id="d0450-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d0450-424">Insira seu endereço de email em **Emails adicionais do administrador**.</span><span class="sxs-lookup"><span data-stu-id="d0450-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d0450-425">Na barra de menu, selecione **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="d0450-426">A captura de tela a seguir mostra os alertas de expansão e redução horizontal.</span><span class="sxs-lookup"><span data-stu-id="d0450-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alertas do Application Insights (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d0450-428">Redirecionar o tráfego entre o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d0450-429">Você pode configurar a alternância do tráfego de aplicativo Web para manual ou automática entre o Azure e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d0450-430">Configurar a alternância manual entre o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d0450-431">Quando o site atingir os limites que você configurar, você receberá um alerta.</span><span class="sxs-lookup"><span data-stu-id="d0450-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="d0450-432">Use as etapas a seguir para redirecionar manualmente o tráfego para o Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="d0450-433">No portal do Azure, selecione seu perfil do Gerenciador de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="d0450-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Pontos de extremidade do Gerenciador de Tráfego no portal do Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="d0450-435">Selecione **Pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="d0450-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="d0450-436">Selecione o **Ponto de extremidade do Azure**.</span><span class="sxs-lookup"><span data-stu-id="d0450-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="d0450-437">Em **Status**, selecione **Habilitado** e, em seguida, selecione **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Habilitar o ponto de extremidade do Azure no portal do Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="d0450-439">Em **Pontos de extremidade** para o perfil do Gerenciador de Tráfego, selecione **Ponto de extremidade externo**.</span><span class="sxs-lookup"><span data-stu-id="d0450-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="d0450-440">Em **Status**, selecione **Desabilitado** e, em seguida, selecione **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="d0450-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Desabilitar ponto de extremidade do Azure Stack Hub no portal do Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="d0450-442">Depois que os pontos de extremidade são configurados, o tráfego do aplicativo vai para o aplicativo Web de expansão do Azure em vez do aplicativo Web do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Pontos de extremidade alterados no tráfego do aplicativo Web do Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="d0450-444">Para reverter o fluxo de volta para o Azure Stack Hub, use as etapas anteriores para:</span><span class="sxs-lookup"><span data-stu-id="d0450-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="d0450-445">Habilitar o ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="d0450-446">Desabilitar o ponto de extremidade do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d0450-447">Configurar a alternância automática entre o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d0450-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d0450-448">Também é possível usar o monitoramento do Application Insights se seu aplicativo é executado em um ambiente [sem servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelo Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="d0450-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="d0450-449">Nesse cenário, você pode configurar o Application Insights para usar um webhook que chama um aplicativo de funções.</span><span class="sxs-lookup"><span data-stu-id="d0450-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="d0450-450">Esse aplicativo habilita ou desabilita automaticamente um ponto de extremidade em resposta a um alerta.</span><span class="sxs-lookup"><span data-stu-id="d0450-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="d0450-451">Use as etapas a seguir como um guia para configurar a alternância automática de tráfego.</span><span class="sxs-lookup"><span data-stu-id="d0450-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="d0450-452">Crie um aplicativo de funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="d0450-453">Crie uma função disparada por HTTP.</span><span class="sxs-lookup"><span data-stu-id="d0450-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="d0450-454">Importe os SDKs do Azure para o Resource Manager, aplicativos Web e Gerenciador de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="d0450-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="d0450-455">Desenvolva código para:</span><span class="sxs-lookup"><span data-stu-id="d0450-455">Develop code to:</span></span>

   - <span data-ttu-id="d0450-456">Autenticar-se na sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="d0450-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="d0450-457">Usar um parâmetro que alterna os pontos de extremidade do Gerenciador de Tráfego para direcionar o tráfego para o Azure ou o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d0450-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="d0450-458">Salvar seu código e adicionar a URL do aplicativo de funções com os parâmetros apropriados à seção **Webhook** das configurações da regra de alerta do Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d0450-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="d0450-459">O tráfego é redirecionado automaticamente quando um alerta do Application Insights é disparado.</span><span class="sxs-lookup"><span data-stu-id="d0450-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d0450-460">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="d0450-460">Next steps</span></span>

- <span data-ttu-id="d0450-461">Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="d0450-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
