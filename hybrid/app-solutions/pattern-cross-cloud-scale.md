---
title: Padrão de dimensionamento entre nuvens no Hub de Azure Stack
description: Saiba como criar um aplicativo escalonável entre nuvens no Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909821"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="fbd79-103">Padrão de dimensionamento entre nuvens</span><span class="sxs-lookup"><span data-stu-id="fbd79-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="fbd79-104">Adicione recursos automaticamente a um aplicativo existente para acomodar um aumento na carga.</span><span class="sxs-lookup"><span data-stu-id="fbd79-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="fbd79-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="fbd79-105">Context and problem</span></span>

<span data-ttu-id="fbd79-106">Seu aplicativo não pode aumentar a capacidade de atender a aumentos inesperados na demanda.</span><span class="sxs-lookup"><span data-stu-id="fbd79-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="fbd79-107">Essa falta de escalabilidade faz com que os usuários não atinjam o aplicativo durante os horários de pico de uso.</span><span class="sxs-lookup"><span data-stu-id="fbd79-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="fbd79-108">O aplicativo pode atender a um número fixo de usuários.</span><span class="sxs-lookup"><span data-stu-id="fbd79-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="fbd79-109">As empresas globais exigem aplicativos baseados em nuvem seguros, confiáveis e disponíveis.</span><span class="sxs-lookup"><span data-stu-id="fbd79-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="fbd79-110">A reunião aumenta na demanda e o uso da infra-estrutura certa para dar suporte a essa demanda é essencial.</span><span class="sxs-lookup"><span data-stu-id="fbd79-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="fbd79-111">As empresas lutam para balancear custos e manutenção com segurança de dados de negócios, armazenamento e disponibilidade em tempo real.</span><span class="sxs-lookup"><span data-stu-id="fbd79-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="fbd79-112">Talvez você não consiga executar seu aplicativo na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="fbd79-113">No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="fbd79-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="fbd79-114">Com esse padrão, você pode usar a elasticidade da nuvem pública com sua solução local.</span><span class="sxs-lookup"><span data-stu-id="fbd79-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="fbd79-115">Solução</span><span class="sxs-lookup"><span data-stu-id="fbd79-115">Solution</span></span>

<span data-ttu-id="fbd79-116">O padrão de dimensionamento entre nuvens estende um aplicativo localizado em uma nuvem local com recursos de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="fbd79-117">O padrão é disparado por um aumento ou diminuição na demanda e, respectivamente, adiciona ou remove recursos na nuvem.</span><span class="sxs-lookup"><span data-stu-id="fbd79-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="fbd79-118">Esses recursos fornecem redundância, disponibilidade rápida e roteamento de conformidade geográfica.</span><span class="sxs-lookup"><span data-stu-id="fbd79-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Padrão de dimensionamento entre nuvens](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="fbd79-120">Esse padrão aplica-se somente a componentes sem estado do seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="fbd79-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="fbd79-121">Componentes</span><span class="sxs-lookup"><span data-stu-id="fbd79-121">Components</span></span>

<span data-ttu-id="fbd79-122">O padrão de dimensionamento entre nuvens consiste nos seguintes componentes.</span><span class="sxs-lookup"><span data-stu-id="fbd79-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="fbd79-123">Fora da nuvem</span><span class="sxs-lookup"><span data-stu-id="fbd79-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="fbd79-124">Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="fbd79-124">Traffic Manager</span></span>

<span data-ttu-id="fbd79-125">No diagrama, isso está localizado fora do grupo de nuvem pública, mas ele precisaria coordenar o tráfego no datacenter local e na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="fbd79-126">O balanceador fornece alta disponibilidade para o aplicativo monitorando pontos de extremidade e fornecendo a redistribuição de failover quando necessário.</span><span class="sxs-lookup"><span data-stu-id="fbd79-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="fbd79-127">DNS (Sistema de Nomes de Domínio)</span><span class="sxs-lookup"><span data-stu-id="fbd79-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="fbd79-128">O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.</span><span class="sxs-lookup"><span data-stu-id="fbd79-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="fbd79-129">Nuvem</span><span class="sxs-lookup"><span data-stu-id="fbd79-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="fbd79-130">Servidor de compilação hospedado</span><span class="sxs-lookup"><span data-stu-id="fbd79-130">Hosted build server</span></span>

<span data-ttu-id="fbd79-131">Um ambiente para hospedar seu pipeline de compilação.</span><span class="sxs-lookup"><span data-stu-id="fbd79-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="fbd79-132">Recursos do aplicativo</span><span class="sxs-lookup"><span data-stu-id="fbd79-132">App resources</span></span>

<span data-ttu-id="fbd79-133">Os recursos do aplicativo precisam ser capazes de reduzir horizontalmente e escalar horizontalmente, como conjuntos de dimensionamento de máquinas virtuais e contêineres.</span><span class="sxs-lookup"><span data-stu-id="fbd79-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="fbd79-134">Nome de domínio personalizado</span><span class="sxs-lookup"><span data-stu-id="fbd79-134">Custom domain name</span></span>

<span data-ttu-id="fbd79-135">Use um nome de domínio personalizado para solicitações de roteamento glob.</span><span class="sxs-lookup"><span data-stu-id="fbd79-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="fbd79-136">Endereços IP públicos</span><span class="sxs-lookup"><span data-stu-id="fbd79-136">Public IP addresses</span></span>

<span data-ttu-id="fbd79-137">Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="fbd79-138">Nuvem local</span><span class="sxs-lookup"><span data-stu-id="fbd79-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="fbd79-139">Servidor de compilação hospedado</span><span class="sxs-lookup"><span data-stu-id="fbd79-139">Hosted build server</span></span>

<span data-ttu-id="fbd79-140">Um ambiente para hospedar seu pipeline de compilação.</span><span class="sxs-lookup"><span data-stu-id="fbd79-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="fbd79-141">Recursos do aplicativo</span><span class="sxs-lookup"><span data-stu-id="fbd79-141">App resources</span></span>

<span data-ttu-id="fbd79-142">Os recursos do aplicativo precisam da capacidade de reduzir e escalar horizontalmente, como conjuntos de dimensionamento de máquinas virtuais e contêineres.</span><span class="sxs-lookup"><span data-stu-id="fbd79-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="fbd79-143">Nome de domínio personalizado</span><span class="sxs-lookup"><span data-stu-id="fbd79-143">Custom domain name</span></span>

<span data-ttu-id="fbd79-144">Use um nome de domínio personalizado para solicitações de roteamento glob.</span><span class="sxs-lookup"><span data-stu-id="fbd79-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="fbd79-145">Endereços IP públicos</span><span class="sxs-lookup"><span data-stu-id="fbd79-145">Public IP addresses</span></span>

<span data-ttu-id="fbd79-146">Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="fbd79-147">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="fbd79-147">Issues and considerations</span></span>

<span data-ttu-id="fbd79-148">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="fbd79-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="fbd79-149">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="fbd79-149">Scalability</span></span>

<span data-ttu-id="fbd79-150">O principal componente do dimensionamento entre nuvem é a capacidade de fornecer dimensionamento sob demanda.</span><span class="sxs-lookup"><span data-stu-id="fbd79-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="fbd79-151">O dimensionamento deve ocorrer entre a infraestrutura de nuvem pública e a local e fornecer um serviço consistente e confiável por demanda.</span><span class="sxs-lookup"><span data-stu-id="fbd79-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="fbd79-152">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="fbd79-152">Availability</span></span>

<span data-ttu-id="fbd79-153">Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.</span><span class="sxs-lookup"><span data-stu-id="fbd79-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="fbd79-154">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="fbd79-154">Manageability</span></span>

<span data-ttu-id="fbd79-155">O padrão de nuvem cruzada garante o gerenciamento contínuo e a interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="fbd79-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="fbd79-156">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="fbd79-156">When to use this pattern</span></span>

<span data-ttu-id="fbd79-157">Use este padrão:</span><span class="sxs-lookup"><span data-stu-id="fbd79-157">Use this pattern:</span></span>

- <span data-ttu-id="fbd79-158">Quando você precisa aumentar a capacidade do aplicativo com demandas inesperadas ou demandas periódicas por demanda.</span><span class="sxs-lookup"><span data-stu-id="fbd79-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="fbd79-159">Quando você não quiser investir em recursos que serão usados somente durante picos.</span><span class="sxs-lookup"><span data-stu-id="fbd79-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="fbd79-160">Pague pelo que usar.</span><span class="sxs-lookup"><span data-stu-id="fbd79-160">Pay for what you use.</span></span>

<span data-ttu-id="fbd79-161">Esse padrão não é recomendado quando:</span><span class="sxs-lookup"><span data-stu-id="fbd79-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="fbd79-162">Sua solução requer que os usuários se conectem pela Internet.</span><span class="sxs-lookup"><span data-stu-id="fbd79-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="fbd79-163">Sua empresa tem regulamentos locais que exigem que a conexão de origem venha de uma chamada no local.</span><span class="sxs-lookup"><span data-stu-id="fbd79-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="fbd79-164">Sua rede experimenta afunilamentos regulares que restringem o desempenho do dimensionamento.</span><span class="sxs-lookup"><span data-stu-id="fbd79-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="fbd79-165">Seu ambiente está desconectado da Internet e não consegue acessar a nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="fbd79-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fbd79-166">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="fbd79-166">Next steps</span></span>

<span data-ttu-id="fbd79-167">Para saber mais sobre os tópicos apresentados neste artigo:</span><span class="sxs-lookup"><span data-stu-id="fbd79-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="fbd79-168">Consulte a [visão geral do Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como esse balanceador de carga de tráfego baseado em DNS funciona.</span><span class="sxs-lookup"><span data-stu-id="fbd79-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="fbd79-169">Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.</span><span class="sxs-lookup"><span data-stu-id="fbd79-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="fbd79-170">Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="fbd79-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="fbd79-171">Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de dimensionamento de nuvem cruzada](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="fbd79-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="fbd79-172">O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.</span><span class="sxs-lookup"><span data-stu-id="fbd79-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="fbd79-173">Você aprende a criar uma solução de nuvem cruzada para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado pelo Hub Azure Stack para um aplicativo Web hospedado do Azure.</span><span class="sxs-lookup"><span data-stu-id="fbd79-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="fbd79-174">Você também aprende a usar o dimensionamento automático por meio do Traffic Manager, garantindo um utilitário de nuvem flexível e escalonável quando sob carga.</span><span class="sxs-lookup"><span data-stu-id="fbd79-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
