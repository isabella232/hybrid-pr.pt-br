---
title: Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub
description: Saiba como direcionar o tráfego a pontos de extremidade específicos com uma solução de aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895424"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="2bbc0-103">Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="2bbc0-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="2bbc0-104">Saiba como direcionar o tráfego a pontos de extremidade específicos com base em várias métricas usando o padrão de aplicativos distribuídos geograficamente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="2bbc0-105">Criar um perfil do Gerenciador de Tráfego com o roteamento baseado em geografia e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="2bbc0-106">Nesta solução, você criará um ambiente de exemplo para:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2bbc0-107">Criar um aplicativo distribuído geograficamente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="2bbc0-108">Usar o Gerenciador de Tráfego para direcionar seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="2bbc0-109">Usar o padrão de aplicativos distribuídos geograficamente</span><span class="sxs-lookup"><span data-stu-id="2bbc0-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="2bbc0-110">Com o padrão distribuído geograficamente, seu aplicativo abrangerá regiões.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="2bbc0-111">Você pode usar o envio à nuvem pública como padrão, mas alguns de seus usuários podem requerer que seus dados permaneçam na região.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="2bbc0-112">Com base nos requisitos dos usuários, você pode direcioná-los para a nuvem mais adequada.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="2bbc0-113">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="2bbc0-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="2bbc0-114">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="2bbc0-114">Scalability considerations</span></span>

<span data-ttu-id="2bbc0-115">A solução que você desenvolverá com esse artigo não é a de acomodar a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="2bbc0-116">No entanto, se usada junto com outras soluções locais e do Azure, você poderá acomodar os requisitos de escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="2bbc0-117">Para obter informações sobre como criar uma solução híbrida com dimensionamento automático por meio do Gerenciador de Tráfego, confira [Criar soluções de dimensionamento entre nuvens com o Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="2bbc0-118">Considerações sobre disponibilidade</span><span class="sxs-lookup"><span data-stu-id="2bbc0-118">Availability considerations</span></span>

<span data-ttu-id="2bbc0-119">Assim como nas considerações de escalabilidade, essa solução não aborda diretamente a disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="2bbc0-120">No entanto, as soluções locais e do Azure podem ser implementadas nessa solução para garantir a alta disponibilidade para todos os componentes envolvidos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="2bbc0-121">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="2bbc0-121">When to use this pattern</span></span>

- <span data-ttu-id="2bbc0-122">Sua organização tem ramificações internacionais que requerem políticas regionais personalizadas de segurança e distribuição.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="2bbc0-123">Cada escritório da sua organização recebe dados de funcionários, de negócios e de instalações, o que requer relatar as atividades conforme os regulamentos locais e fusos horários.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="2bbc0-124">Os requisitos de alta escala são atendidos pela escala horizontal de aplicativos, com várias implantações de aplicativo dentro de uma única região e entre regiões para lidar com requisitos de carga extrema.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="2bbc0-125">Planejamento da topologia</span><span class="sxs-lookup"><span data-stu-id="2bbc0-125">Planning the topology</span></span>

<span data-ttu-id="2bbc0-126">Antes de desenvolver o volume do aplicativo distribuído, é importante saber o seguinte:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="2bbc0-127">**Domínio personalizado para o aplicativo:** Qual é o nome de domínio personalizado que os clientes usarão para acessar o aplicativo?</span><span class="sxs-lookup"><span data-stu-id="2bbc0-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="2bbc0-128">Para o aplicativo de exemplo, o nome de domínio personalizado é *www\.scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="2bbc0-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="2bbc0-129">**Domínio do Gerenciador de Tráfego:** Um nome de domínio é escolhido quando criamos um [perfil do Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="2bbc0-130">Esse nome é combinado com o sufixo *trafficmanager.net* para registrar uma entrada de domínio controlada pelo Gerenciador de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="2bbc0-131">Para o aplicativo de exemplo, o nome escolhido é *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="2bbc0-132">Assim, o nome de domínio completo que é gerenciado pelo Gerenciador de Tráfego é *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="2bbc0-133">**Estratégia para dimensionar o volume do aplicativo:** Decida se o volume do aplicativo será distribuído em vários ambientes do Serviço de Aplicativo em uma única região, em várias regiões ou em um mix das duas abordagens.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="2bbc0-134">A decisão deve se basear nas expectativas de origem de tráfego do cliente e em quão bem o restante da infraestrutura de back-end de suporte de um aplicativo pode ser escalonado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="2bbc0-135">Por exemplo, com um aplicativo 100% sem monitoração de estado, ele pode ser altamente dimensionado usando uma combinação de vários ambientes do Serviço de Aplicativo por região do Azure e multiplicado pelos ambientes do Serviço de Aplicativo implantados em várias regiões do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="2bbc0-136">Com mais de 15 regiões do Azure global disponíveis para escolha, os clientes podem realmente desenvolver um volume de aplicativo de hiperescala mundial.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="2bbc0-137">Para o aplicativo de exemplo usado aqui, três ambientes do Serviço de Aplicativo foram criados em uma única região do Azure (Centro-Sul dos EUA).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="2bbc0-138">**Convenção de nomenclatura para os ambientes do Serviço de Aplicativo:** Cada ambiente do Serviço de Aplicativo requer um nome exclusivo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="2bbc0-139">Além de um ou dois ambientes do Serviço de Aplicativo, é útil ter uma convenção de nomenclatura para ajudar a identificar cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="2bbc0-140">Para o aplicativo de exemplo deste artigo, foi usada uma convenção de nomenclatura simples.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="2bbc0-141">Os nomes dos três ambientes do Serviço de Aplicativo são *fe1ase*, *fe2ase* e *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="2bbc0-142">**Convenção de nomenclatura para os aplicativos:** como várias instâncias do aplicativo serão implantadas, é necessário um nome para cada instância do aplicativo implantado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="2bbc0-143">Com o ambiente do Serviço de Aplicativo do Power Apps, o mesmo nome de aplicativo pode ser usado em vários ambientes.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="2bbc0-144">Como cada ambiente do Serviço de Aplicativo tem um sufixo de domínio exclusivo, os desenvolvedores podem optar por usar novamente o mesmo nome de aplicativo em cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="2bbc0-145">Por exemplo, um desenvolvedor poderia ter aplicativos nomeados da seguinte forma: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="2bbc0-146">Em relação ao aplicativo usado aqui, cada instância de aplicativo tem um nome exclusivo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="2bbc0-147">Os nomes de instância de aplicativo usados são *webfrontend1*, *webfrontend2* e *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="2bbc0-148">![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="2bbc0-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="2bbc0-149">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="2bbc0-150">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="2bbc0-151">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="2bbc0-152">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="2bbc0-153">Parte 1: Criar um aplicativo distribuído geograficamente</span><span class="sxs-lookup"><span data-stu-id="2bbc0-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="2bbc0-154">Nessa parte, você criará um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2bbc0-155">Criar aplicativos Web e publicá-los.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="2bbc0-156">Adicionar códigos ao Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="2bbc0-157">Direcionar a compilação do aplicativo para vários destinos de nuvem.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="2bbc0-158">Gerenciar e configurar o processo de CD.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2bbc0-159">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="2bbc0-159">Prerequisites</span></span>

<span data-ttu-id="2bbc0-160">São necessárias uma assinatura do Azure e a instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="2bbc0-161">Etapas do aplicativo distribuído geograficamente</span><span class="sxs-lookup"><span data-stu-id="2bbc0-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="2bbc0-162">Obter um domínio personalizado e configurar o DNS</span><span class="sxs-lookup"><span data-stu-id="2bbc0-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="2bbc0-163">Atualize o arquivo de zona DNS do domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="2bbc0-164">Assim, o Azure AD poderá verificar a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="2bbc0-165">Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Microsoft 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="2bbc0-166">Registre um domínio personalizado com um registrador público.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="2bbc0-167">Entre no registrador de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="2bbc0-168">Um administrador aprovado pode ser necessário para fazer as atualizações de DNS.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="2bbc0-169">Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="2bbc0-170">A entrada DNS não altera comportamentos, como o roteamento de e-mails ou a hospedagem na Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="2bbc0-171">Criar aplicativos Web e publicá-los</span><span class="sxs-lookup"><span data-stu-id="2bbc0-171">Create web apps and publish</span></span>

<span data-ttu-id="2bbc0-172">Configure a integração contínua/entrega contínua (CI/CD) híbrida para implantar aplicativos Web no Azure e no Azure Stack Hub e para enviar alterações por push a ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-173">São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="2bbc0-174">Para obter mais informações, confira [Pré-requisitos para implantar o Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="2bbc0-175">Adicionar código ao Azure Repos</span><span class="sxs-lookup"><span data-stu-id="2bbc0-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="2bbc0-176">Entre no Visual Studio com uma **conta que tenha direitos de criação de projeto** no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="2bbc0-177">A CI/CD pode ser aplicada tanto ao código do aplicativo quanto ao código de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="2bbc0-178">Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privado e hospedado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conectar-se a um projeto no Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="2bbc0-180">**Clone o repositório** criando e abrindo o aplicativo Web padrão.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonar repositório no Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="2bbc0-182">Criar implantações de aplicativo Web em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="2bbc0-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="2bbc0-183">Edite o arquivo **WebApplication.csproj**: Selecione `Runtimeidentifier` e adicione `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="2bbc0-184">(Confira a documentação de [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)</span><span class="sxs-lookup"><span data-stu-id="2bbc0-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar arquivo de projeto de aplicativo Web no Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="2bbc0-186">**Faça check-in do código no Azure Repos** usando o Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="2bbc0-187">Confirme se o **código do aplicativo** foi inserido no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="2bbc0-188">Criar a definição de build</span><span class="sxs-lookup"><span data-stu-id="2bbc0-188">Create the build definition</span></span>

1. <span data-ttu-id="2bbc0-189">**Entre no Azure Pipelines** para confirmar a capacidade de criar definições de build.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="2bbc0-190">Adicione o código `-r win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="2bbc0-191">Essa adição é necessária para disparar uma implantação independente com o .NET Core.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Adicionar código à definição de build no Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="2bbc0-193">**Execute o build**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-193">**Run the build**.</span></span> <span data-ttu-id="2bbc0-194">O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="2bbc0-195">Usar um agente hospedado do Azure</span><span class="sxs-lookup"><span data-stu-id="2bbc0-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="2bbc0-196">Usar um agente hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="2bbc0-197">A manutenção e as atualizações são executadas automaticamente pelo Microsoft Azure, o que permite desenvolvimentos ininterruptos, testes e implantações.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="2bbc0-198">Gerenciar e configurar o processo de CD</span><span class="sxs-lookup"><span data-stu-id="2bbc0-198">Manage and configure the CD process</span></span>

<span data-ttu-id="2bbc0-199">O Azure DevOps Services fornece um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade e ambientes de produção, incluindo a necessidade de aprovações em estágios específicos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="2bbc0-200">Criar definição de versão</span><span class="sxs-lookup"><span data-stu-id="2bbc0-200">Create release definition</span></span>

1. <span data-ttu-id="2bbc0-201">Selecione o botão **mais** para adicionar uma nova versão na guia **Versões** na seção **Build e Versão** do Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Criar uma definição de versão no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="2bbc0-203">Aplique o modelo de implantação do Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-203">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar o modelo de implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="2bbc0-205">Em **Adicionar artefato**, adicione o artefato para o aplicativo de build no Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Adicionar artefato ao build da nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="2bbc0-207">Na guia Pipeline, selecione o link **Fase, Tarefa** do ambiente e defina os valores do ambiente de nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir os valores de ambiente de nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="2bbc0-209">Defina o **nome do ambiente** e selecione a **assinatura do Azure** do ponto de extremidade da Nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecionar a assinatura do Azure para o ponto de extremidade da Nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="2bbc0-211">Em **Nome do serviço de aplicativo**, defina o nome do serviço de aplicativo necessário do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir o nome do serviço de aplicativo Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="2bbc0-213">Insira “Hosted VS2017” em **Fila de agentes** para o ambiente hospedado na nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Definir a Fila de agentes para o ambiente hospedado na nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="2bbc0-215">No menu Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="2bbc0-216">Selecione **OK** para a **localização da pasta**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-216">Select **OK** to **folder location**.</span></span>
  
      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Caixa de diálogo do seletor de pasta 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="2bbc0-219">Salve todas as alterações e volte para o **pipeline de lançamento**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Salvar alterações no pipeline de lançamento no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="2bbc0-221">Adicione um novo artefato selecionando o build do aplicativo Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Adicionar novo artefato do aplicativo Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="2bbc0-223">Adicione mais um ambiente aplicando a Implantação do Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Adicionar ambiente à Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="2bbc0-225">Nomeie o novo ambiente como Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-225">Name the new environment Azure Stack Hub.</span></span>

    ![Nomear o ambiente na Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="2bbc0-227">Localize o ambiente do Azure Stack Hub na guia **Tarefa**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Ambiente do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="2bbc0-229">Selecione a assinatura do ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selecionar a assinatura do ponto de extremidade do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="2bbc0-231">Defina o nome do aplicativo Web Azure Stack Hub como o nome do serviço de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Definir o nome do aplicativo Web do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="2bbc0-233">Selecione o agente do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-233">Select the Azure Stack Hub agent.</span></span>

    ![Selecionar o agente do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="2bbc0-235">Na seção Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="2bbc0-236">Selecione **OK** para a localização da pasta.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-236">Select **OK** to folder location.</span></span>

    ![Selecionar a pasta da Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Caixa de diálogo do seletor de pasta 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="2bbc0-239">Na guia Variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, defina seu valor como **true** e defina o escopo no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Adicionar variável à Implantação do Aplicativo Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="2bbc0-241">Selecione o ícone de gatilho de implantação **Contínuo** em ambos os artefatos e habilite o gatilho de implantação **Continuar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selecionar o gatilho de implantação contínua no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="2bbc0-243">Selecione o ícone de condições de **Pré-implantação** no ambiente do Azure Stack Hub e defina o gatilho com **Após o lançamento.**</span><span class="sxs-lookup"><span data-stu-id="2bbc0-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Selecionar as condições de pré-implantação no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="2bbc0-245">Salvar todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-246">Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) na criação de uma definição da versão a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="2bbc0-247">Essas configurações não podem ser modificadas nas configurações da tarefa. Em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="2bbc0-248">Parte 2: Atualizar as opções do aplicativo Web</span><span class="sxs-lookup"><span data-stu-id="2bbc0-248">Part 2: Update web app options</span></span>

<span data-ttu-id="2bbc0-249">O [Serviço de Aplicativo do Azure](/azure/app-service/overview) fornece um serviço de hospedagem na Web altamente escalonável e com aplicação automática de patches.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Serviço de aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="2bbc0-251">Mapear um nome DNS personalizado existente para aplicativos Web do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="2bbc0-252">Use um **registro CNAME** e um **registro A** para mapear um nome DNS personalizado para o Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="2bbc0-253">Mapear um nome DNS personalizado existente para aplicativos Web do Azure</span><span class="sxs-lookup"><span data-stu-id="2bbc0-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-254">Use um CNAME para todos os nomes DNS personalizados, exceto um domínio raiz (por exemplo, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="2bbc0-255">Para migrar um site ativo e seu nome de domínio DNS para o Serviço de Aplicativo, consulte [Migrar um nome DNS ativo para o Serviço de Aplicativo do Azure](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2bbc0-256">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="2bbc0-256">Prerequisites</span></span>

<span data-ttu-id="2bbc0-257">Para concluir essa solução:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-257">To complete this solution:</span></span>

- <span data-ttu-id="2bbc0-258">[Crie um aplicativo do Serviço de Aplicativo](/azure/app-service/) ou use um aplicativo criado para outra solução.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="2bbc0-259">Compre um nome de domínio e garanta o acesso ao registro de DNS do provedor de domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="2bbc0-260">Atualize o arquivo de zona DNS do domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="2bbc0-261">O Azure AD verificará a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="2bbc0-262">Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Microsoft 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="2bbc0-263">Registre um domínio personalizado com um registrador público.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="2bbc0-264">Entre no registrador de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="2bbc0-265">(Um administrador aprovado pode ser necessário para fazer atualizações de DNS.)</span><span class="sxs-lookup"><span data-stu-id="2bbc0-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="2bbc0-266">Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="2bbc0-267">Por exemplo, para adicionar entradas DNS para northwindcloud.com e www\.northwindcloud.com, defina as configurações do DNS para o domínio raiz northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-268">Um nome de domínio pode ser comprado usando o [portal do Azure](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="2bbc0-269">Para mapear um nome DNS personalizado para um aplicativo Web, o [plano do Serviço de Aplicativo](https://azure.microsoft.com/pricing/details/app-service/) do aplicativo Web deve ser uma camada paga (**Compartilhado**, **Básico**, **Standard** ou **Premium**).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="2bbc0-270">Criar e mapear registros CNAME e A</span><span class="sxs-lookup"><span data-stu-id="2bbc0-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="2bbc0-271">Acessar registros DNS com o provedor de domínio</span><span class="sxs-lookup"><span data-stu-id="2bbc0-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="2bbc0-272">Use o DNS do Azure para configurar um nome DNS personalizado para Aplicativos Web do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="2bbc0-273">Para obter mais informações, consulte [Usar o DNS do Azure para fornecer as configurações de domínio personalizadas para um serviço do Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="2bbc0-274">Entre no site do provedor principal.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="2bbc0-275">Localize a página para gerenciamento de registros DNS.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="2bbc0-276">Todo provedor de domínio tem sua própria interface de registros DNS.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="2bbc0-277">Procure áreas do site rotuladas como **Nome de Domínio**, **DNS** ou **Gerenciamento de Servidor de Nomes**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="2bbc0-278">A página com os registros DNS pode ser exibida em **Meus domínios**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="2bbc0-279">Encontre o link denominado **Arquivo de zona**, **Registros DNS** ou **Configuração avançada**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="2bbc0-280">A captura de tela a seguir é um exemplo de uma página de registros DNS:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-280">The following screenshot is an example of a DNS records page:</span></span>

![Exemplo de página de registros DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="2bbc0-282">Em Registrador de Nome de Domínio, selecione **Adicionar ou Criar** para criar um registro.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="2bbc0-283">Alguns provedores têm links diferentes para adicionar tipos de registro diferentes.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="2bbc0-284">Confira a documentação do provedor.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="2bbc0-285">Adicione um registro CNAME para mapear um subdomínio para o nome do host padrão do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="2bbc0-286">Para o exemplo do domínio www\.northwindcloud.com, adicione um registro CNAME que mapeie o nome para `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="2bbc0-287">Depois da inclusão do CNAME, a página de registros DNS ficará parecida com o seguinte exemplo:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navegação no Portal para o aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="2bbc0-289">Habilitar o mapeamento de registro CNAME no Azure</span><span class="sxs-lookup"><span data-stu-id="2bbc0-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="2bbc0-290">Em uma nova guia, entre no portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="2bbc0-291">Acesse Serviços de Aplicativos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-291">Go to App Services.</span></span>

3. <span data-ttu-id="2bbc0-292">Selecione aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-292">Select web app.</span></span>

4. <span data-ttu-id="2bbc0-293">No painel de navegação à esquerda da página do aplicativo no portal do Azure, selecione **Domínios personalizados**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="2bbc0-294">Selecione o ícone **+** ao lado de **Adicionar nome do host**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="2bbc0-295">Digite o nome de domínio totalmente qualificado, como `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="2bbc0-296">Selecione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-296">Select **Validate**.</span></span>

8. <span data-ttu-id="2bbc0-297">Caso seja indicado, adicione registros de outros tipos (`A` ou `TXT`) aos registros DNS dos registradores de nome de domínio.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="2bbc0-298">O Azure fornecerá os valores e os tipos desses registros:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="2bbc0-299">a.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-299">a.</span></span>  <span data-ttu-id="2bbc0-300">Um registro **A** a ser mapeado para o endereço IP do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="2bbc0-301">b.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-301">b.</span></span>  <span data-ttu-id="2bbc0-302">Um registro **TXT** a ser mapeado para o nome do host padrão do aplicativo `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="2bbc0-303">O Serviço de Aplicativo usa esse registro somente em tempo de configuração para verificar a propriedade de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="2bbc0-304">Após a verificação, exclua o registro TXT.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="2bbc0-305">Conclua essa tarefa na guia do registrador de domínio e revalide até que o botão **Adicionar nome de host** seja ativado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="2bbc0-306">Verifique se **Tipo de registro de nome do host** está definido como **CNAME** (www.example.com ou qualquer subdomínio).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="2bbc0-307">Selecione **Adicionar nome do host**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="2bbc0-308">Digite o nome de domínio totalmente qualificado, como `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="2bbc0-309">Selecione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-309">Select **Validate**.</span></span> <span data-ttu-id="2bbc0-310">A opção **Adicionar** está ativada.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="2bbc0-311">Verifique se **Tipo de registro de nome de host** está definido como **Registro A** (example.com).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="2bbc0-312">**Adicionar nome do host**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-312">**Add hostname**.</span></span>

    <span data-ttu-id="2bbc0-313">Pode levar algum tempo para que os novos nomes do host sejam refletidos na página **Domínios personalizados** do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="2bbc0-314">Tente atualizar o navegador para atualizar os dados.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-314">Try refreshing the browser to update the data.</span></span>
  
    ![Domínios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="2bbc0-316">Caso haja um erro, uma notificação de erro de verificação será exibida na parte inferior da página.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Erro de verificação de domínio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="2bbc0-318">As etapas acima podem ser repetidas para mapear um domínio curinga (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="2bbc0-319">Isso permite que quaisquer subdomínios sejam adicionados a esse serviço de aplicativo sem a necessidade de criar um registro CNAME separado para cada um.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="2bbc0-320">Siga as instruções do registrador para definir essa configuração.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="2bbc0-321">Testar em um navegador</span><span class="sxs-lookup"><span data-stu-id="2bbc0-321">Test in a browser</span></span>

<span data-ttu-id="2bbc0-322">Navegue até os nomes DNS configurados anteriormente (por exemplo, `northwindcloud.com` ou `www.northwindcloud.com`).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="2bbc0-323">Parte 3: Associar um certificado SSL personalizado</span><span class="sxs-lookup"><span data-stu-id="2bbc0-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="2bbc0-324">Nesta parte, vamos:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2bbc0-325">Associar o certificado SSL personalizado ao Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="2bbc0-326">Impor o HTTPS ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="2bbc0-327">Automatizar a associação de certificado SSL com scripts.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-328">Caso seja necessário, obtenha um certificado SSL de cliente no portal do Azure e associe-o ao aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="2bbc0-329">Para obter mais informações, confira o [Tutorial de Certificados do Serviço de Aplicativo](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2bbc0-330">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="2bbc0-330">Prerequisites</span></span>

<span data-ttu-id="2bbc0-331">Para concluir essa solução:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-331">To complete this  solution:</span></span>

- [<span data-ttu-id="2bbc0-332">Crie um aplicativo do Serviço de Aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="2bbc0-333">Mapeie um nome DNS personalizado para o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="2bbc0-334">Adquira um certificado SSL de uma autoridade de certificado confiável e use a chave para assinar a solicitação.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="2bbc0-335">Requisitos para o certificado SSL</span><span class="sxs-lookup"><span data-stu-id="2bbc0-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="2bbc0-336">Para usar o certificado no Serviço de Aplicativo, o certificado deve atender a todos os seguintes requisitos:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="2bbc0-337">Ser assinado por uma autoridade de certificado confiável.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="2bbc0-338">Ser exportado como um arquivo PFX protegido por senha.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="2bbc0-339">Conter chave privada com pelo menos 2.048 bits de extensão.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="2bbc0-340">Conter todos os certificados intermediários na cadeia de certificados.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="2bbc0-341">Os **certificados ECC (Criptografia de Curva Elíptica)** funcionam com o Serviço de Aplicativo, mas não estão incluídos neste guia.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="2bbc0-342">Confira uma autoridade de certificação para obter assistência quanto à criação de certificados ECC.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="2bbc0-343">Preparar o aplicativo Web</span><span class="sxs-lookup"><span data-stu-id="2bbc0-343">Prepare the web app</span></span>

<span data-ttu-id="2bbc0-344">Para a associação de um certificado SSL personalizado ao aplicativo Web, o [plano do Serviço de Aplicativo](https://azure.microsoft.com/pricing/details/app-service/) deve estar no nível **Básico**, **Standard** ou **Premium**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="2bbc0-345">Entrar no Azure</span><span class="sxs-lookup"><span data-stu-id="2bbc0-345">Sign in to Azure</span></span>

1. <span data-ttu-id="2bbc0-346">Abra o [portal do Azure](https://portal.azure.com/) e vá até o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="2bbc0-347">No menu à esquerda, selecione **Serviços de Aplicativos** e, em seguida, selecione o nome do aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Selecionar o aplicativo Web no portal do Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="2bbc0-349">Verifique o tipo de preço</span><span class="sxs-lookup"><span data-stu-id="2bbc0-349">Check the pricing tier</span></span>

1. <span data-ttu-id="2bbc0-350">No painel de navegação à esquerda da página do aplicativo Web, role até a seção **Configurações** e selecione **Escalar verticalmente (plano do Serviço de Aplicativo)** .</span><span class="sxs-lookup"><span data-stu-id="2bbc0-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu da escala vertical no aplicativo Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="2bbc0-352">Garanta que o aplicativo Web não está no nível **Gratuito** ou **Compartilhado**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="2bbc0-353">O tipo de preço atual do aplicativo Web está realçado por uma caixa azul escuro.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Verificar o tipo de preço do aplicativo Web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="2bbc0-355">Não há suporte para o SSL personalizado nos tipos de preço **Gratuito** ou **Compartilhado**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="2bbc0-356">Para aumentar a escala, siga as etapas da próxima seção na página **Escolher o tipo de preço** e vá para [Carregar e associar o certificado SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="2bbc0-357">Escalar verticalmente seu Plano do Serviço de Aplicativo</span><span class="sxs-lookup"><span data-stu-id="2bbc0-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="2bbc0-358">Selecione uma das camadas **Básico**, **Standard** ou **Premium**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="2bbc0-359">Selecione **Selecionar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-359">Select **Select**.</span></span>

![Escolher o tipo de preço do seu aplicativo Web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="2bbc0-361">A operação de escala estará concluída quando a notificação for exibida.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-361">The scale operation is complete when notification is displayed.</span></span>

![Escalar verticalmente a notificação](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="2bbc0-363">Associar o certificado SSL e mesclar certificados intermediários</span><span class="sxs-lookup"><span data-stu-id="2bbc0-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="2bbc0-364">Mescle múltiplos certificados na cadeia.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="2bbc0-365">**Abra cada certificado** recebido em um editor de texto.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="2bbc0-366">Crie um arquivo para o certificado mesclado com o nome de *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="2bbc0-367">Em um editor de texto, copie o conteúdo de cada certificado para esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="2bbc0-368">A ordem de seus certificados deve seguir a ordem na cadeia de certificados, começando com o seu certificado e terminando com o certificado raiz.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="2bbc0-369">Ela se parece com o seguinte exemplo:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="2bbc0-370">Exportar o certificado para PFX</span><span class="sxs-lookup"><span data-stu-id="2bbc0-370">Export certificate to PFX</span></span>

<span data-ttu-id="2bbc0-371">Exporte o certificado SSL mesclado com a chave privada gerada pelo certificado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="2bbc0-372">Um arquivo de chave privada será criado via OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="2bbc0-373">Para exportar o certificado para o PFX, execute o seguinte comando e substitua os espaços reservados `<private-key-file>` e `<merged-certificate-file>` pelo caminho da chave privada e o arquivo de certificado mesclado:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="2bbc0-374">Quando solicitado, defina uma senha de exportação para carregar seu certificado SSL para o Serviço de Aplicativo posteriormente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="2bbc0-375">Quando o IIS ou o **Certreq.exe** forem usados para gerar a solicitação de certificado, instale o certificado em um computador local e, em seguida, [exporte o certificado para PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="2bbc0-376">Carregar o certificado SSL</span><span class="sxs-lookup"><span data-stu-id="2bbc0-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="2bbc0-377">No painel de navegação esquerdo do aplicativo Web, selecione **configurações de SSL**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="2bbc0-378">Selecione **Carregar Certificado**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="2bbc0-379">Em **Arquivo de Certificado PFX**, selecione arquivo PFX.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="2bbc0-380">Em **Senha do certificado**, digite a senha criada ao exportar o arquivo PFX.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="2bbc0-381">Escolha **Carregar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-381">Select **Upload**.</span></span>

    ![Carregar o certificado SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="2bbc0-383">Quando o Serviço de Aplicativo terminar de carregar o certificado, ele aparecerá na página **Configurações de SSL**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Configurações de SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="2bbc0-385">Associar o certificado SSL</span><span class="sxs-lookup"><span data-stu-id="2bbc0-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="2bbc0-386">Na seção **Associações SSL**, selecione **Adicionar associação**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="2bbc0-387">Caso o certificado tenha sido atualizado, mas não apareça nos nomes de domínio no menu suspenso **Nome do host**, tente atualizar a página do navegador.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="2bbc0-388">Na página **Adicionar Associação SSL**, use os menus suspensos para selecionar o nome de domínio a ser protegido e o certificado a ser usado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="2bbc0-389">Em **Tipo SSL**, selecione se deseja usar [**SNI (Indicação de Nome de Servidor)**](https://en.wikipedia.org/wiki/Server_Name_Indication) ou SSL baseado em IP.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="2bbc0-390">**SSL com base em SNI**: Várias associações SSL baseadas em SNI podem ser adicionadas.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="2bbc0-391">Esta opção permite que vários certificados SSL protejam vários domínios no mesmo endereço IP.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="2bbc0-392">Navegadores mais modernos (incluindo Internet Explorer, Chrome, Firefox e Opera) dão suporte ao SNI (encontre informações de suporte ao navegador mais abrangentes em [Indicação de Nome de Servidor](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="2bbc0-393">**SSL baseado em IP**: Apenas uma associação SSL baseada em IP pode ser adicionada.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="2bbc0-394">Esta opção permite apenas um certificado SSL para proteger um endereço IP público dedicado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="2bbc0-395">Para proteger vários domínios, proteja todos usando o mesmo certificado SSL.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="2bbc0-396">O SSL baseado em IP é a opção tradicional para a associação de SSL.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="2bbc0-397">Selecione **Adicionar Associação**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-397">Select **Add Binding**.</span></span>

    ![Adicionar associação SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="2bbc0-399">Quando o Serviço de Aplicativo terminar de carregar o certificado, ele aparecerá nas seções de **associações SSL**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Atualização concluída das associações SSL](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="2bbc0-401">Remapear um registro para IP SSL</span><span class="sxs-lookup"><span data-stu-id="2bbc0-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="2bbc0-402">Se o SSL baseado em IP não for usado no aplicativo Web, pule para [Testar o HTTPS para seu domínio personalizado](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="2bbc0-403">Por padrão, o aplicativo Web usa um endereço IP público compartilhado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="2bbc0-404">Quando o certificado estiver associado ao SSL baseado em IP, o Serviço de Aplicativo criará um novo endereço IP dedicado para o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="2bbc0-405">Quando um registro A for mapeado para o aplicativo Web, o registro de domínio deve ser atualizado com o endereço IP dedicado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="2bbc0-406">A página **Domínio personalizado** é atualizada com o novo endereço IP dedicado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="2bbc0-407">Copie esse [endereço IP](/azure/app-service/app-service-web-tutorial-custom-domain) e, em seguida, remapeie o [registro A](/azure/app-service/app-service-web-tutorial-custom-domain) para esse novo endereço IP.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="2bbc0-408">Testar HTTPS</span><span class="sxs-lookup"><span data-stu-id="2bbc0-408">Test HTTPS</span></span>

<span data-ttu-id="2bbc0-409">Em navegadores diferentes, acesse `https://<your.custom.domain>` para garantir que o aplicativo Web seja atendido.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Navegar até o aplicativo Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="2bbc0-411">Erros de validação de certificado podem ocorrer devido aos seguintes motivos: um certificado autoassinado; ou certificados intermediários podem ter sido deixados para ser desativados na exportação para o arquivo PFX.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="2bbc0-412">Impor HTTPS</span><span class="sxs-lookup"><span data-stu-id="2bbc0-412">Enforce HTTPS</span></span>

<span data-ttu-id="2bbc0-413">Por padrão, todos podem acessar o aplicativo Web usando o HTTP.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="2bbc0-414">Todas as solicitações HTTP à porta HTTPS podem ser redirecionadas.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="2bbc0-415">Na página do aplicativo Web, selecione **Configurações do SL**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="2bbc0-416">Depois, em **HTTPS somente**, selecione **Ligado**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Impor HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="2bbc0-418">Quando a operação for concluída, vá até alguma das URLs HTTP que aponte para seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="2bbc0-419">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-419">For example:</span></span>

- <span data-ttu-id="2bbc0-420"> https://<app_name>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="2bbc0-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="2bbc0-421">Impor o TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="2bbc0-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="2bbc0-422">O aplicativo permite o protocolo [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 por padrão, o que não é mais considerado seguro pelos padrões do setor (como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="2bbc0-423">Para impor versões superiores do TLS, execute estas etapas:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="2bbc0-424">Na página do aplicativo Web, na navegação à esquerda, selecione **Configurações SSL**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="2bbc0-425">Em **Versão do TLS**, selecione a versão mínima do TLS.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Impor o TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="2bbc0-427">Criar um perfil do Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="2bbc0-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="2bbc0-428">Escolha **Criar um recurso** > **Rede** > **Perfil do Gerenciador de Tráfego** > **Criar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="2bbc0-429">Em **Criar perfil do Gerenciador de Tráfego**, preencha os seguintes campos:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="2bbc0-430">Em **Nome**, forneça um nome para o perfil.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="2bbc0-431">Esse nome deve ser exclusivo na zona de tráfego manager.net e resultará no nome DNS trafficmanager.net, que é usado para acessar o perfil do Gerenciador de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="2bbc0-432">Em **Método de roteamento**, selecione o **Método de roteamento geográfico**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="2bbc0-433">Em **Assinatura**, selecione a assinatura sob a qual o perfil deve ser criado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="2bbc0-434">Em **Grupo de Recursos**, crie um novo grupo de recursos no qual colocar esse perfil.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="2bbc0-435">Em **Local do grupo de recursos**, selecione o local do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="2bbc0-436">Essa configuração refere-se ao local do grupo de recursos e não tem impacto no perfil do Gerenciador de Tráfego implantado globalmente.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="2bbc0-437">Selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-437">Select **Create**.</span></span>

    7. <span data-ttu-id="2bbc0-438">Quando a implantação global do perfil do Gerenciador de Tráfego estiver concluída, ela será listada no respectivo grupo de recursos como um dos recursos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Grupos de recursos na criação de perfil do Gerenciador de Tráfego](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="2bbc0-440">Adicionar pontos de extremidade do Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="2bbc0-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="2bbc0-441">Na barra de pesquisa do portal, pesquise o nome do **Perfil do Gerenciador de Tráfego** criado na seção anterior e selecione o perfil do gerenciador de tráfego nos resultados exibidos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="2bbc0-442">Em **Perfil do Gerenciador de Tráfego**, na seção **Configurações**, selecione **Pontos de extremidade**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="2bbc0-443">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-443">Select **Add**.</span></span>

4. <span data-ttu-id="2bbc0-444">Adicione o ponto de extremidade do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="2bbc0-445">Para **Tipo**, selecione **Ponto de extremidade externo**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="2bbc0-446">Forneça um **Nome** para esse ponto de extremidade, idealmente o nome do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="2bbc0-447">Para o nome de domínio totalmente qualificado (**FQDN**), use a URL externa para o aplicativo Web do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="2bbc0-448">Em Mapeamento geográfico, selecione uma região/continente em que o recurso esteja localizado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="2bbc0-449">Por exemplo, **Europa**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="2bbc0-450">No menu suspenso País/Região exibido, selecione o país que se aplica a esse ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="2bbc0-451">Por exemplo, **Alemanha**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="2bbc0-452">Mantenha a opção **Adicionar como desabilitado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="2bbc0-453">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-453">Select **OK**.</span></span>

12. <span data-ttu-id="2bbc0-454">Para adicionar o Ponto de Extremidade do Azure:</span><span class="sxs-lookup"><span data-stu-id="2bbc0-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="2bbc0-455">Em **Tipo**, selecione **Ponto de extremidade do Azure**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="2bbc0-456">Forneça um **Nome** para o ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="2bbc0-457">Para **Tipo de recurso de destino**, selecione **Serviço de Aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="2bbc0-458">Para **Recurso de destino**, selecione **Escolher um serviço de aplicativo** para mostrar a lista dos Aplicativos Web sob a mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="2bbc0-459">Em **Recursos**, escolha o Serviço de aplicativo usado como o primeiro ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="2bbc0-460">Em Mapeamento geográfico, selecione uma região/continente em que o recurso esteja localizado.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="2bbc0-461">Por exemplo, **América do Norte/América Central/Caribe.**</span><span class="sxs-lookup"><span data-stu-id="2bbc0-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="2bbc0-462">No menu suspenso País/Região que é exibido, deixe esse ponto em branco para selecionar todo o agrupamento regional acima.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="2bbc0-463">Mantenha a opção **Adicionar como desabilitado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="2bbc0-464">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="2bbc0-465">Crie pelo menos um ponto de extremidade com um escopo geográfico de Todos (Mundo) para servir como o ponto de extremidade padrão do recurso.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="2bbc0-466">Quando a adição de ambos os pontos de extremidade estiver concluída, eles serão exibidos no **Perfil do Gerenciador de Tráfego**, juntamente com seu status de monitoramento como **Online**.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Status do ponto de extremidade do perfil do Gerenciador de Tráfego](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="2bbc0-468">As empresas globais contam com os recursos de distribuição geográfica do Azure</span><span class="sxs-lookup"><span data-stu-id="2bbc0-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="2bbc0-469">Direcionar o tráfego de dados por meio do Gerenciador de Tráfego do Azure e pontos de extremidade específicos de geografia permite que as empresas globais sigam as normas regionais e mantenham os dados em conformidade e seguros, o que é crucial para o sucesso de regiões de negócios locais e remotos.</span><span class="sxs-lookup"><span data-stu-id="2bbc0-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2bbc0-470">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="2bbc0-470">Next steps</span></span>

- <span data-ttu-id="2bbc0-471">Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="2bbc0-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
