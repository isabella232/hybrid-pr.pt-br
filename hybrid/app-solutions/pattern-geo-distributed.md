---
title: Padrão de aplicativo distribuído geograficamente no Hub de Azure Stack
description: Saiba mais sobre o padrão de aplicativo distribuído geograficamente para a borda inteligente usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909785"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="f1ea5-103">Padrão de aplicativo distribuído geograficamente</span><span class="sxs-lookup"><span data-stu-id="f1ea5-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="f1ea5-104">Saiba como fornecer pontos de extremidade de aplicativo em várias regiões e rotear o tráfego do usuário com base nas necessidades de local e conformidade.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="f1ea5-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="f1ea5-105">Context and problem</span></span>

<span data-ttu-id="f1ea5-106">As organizações com geografias de longo alcance se esforçam para distribuir e proteger com segurança e precisão o acesso aos dados e, ao mesmo tempo, garantem os níveis necessários de segurança, conformidade e desempenho por usuário, local e dispositivo entre bordas.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="f1ea5-107">Solução</span><span class="sxs-lookup"><span data-stu-id="f1ea5-107">Solution</span></span>

<span data-ttu-id="f1ea5-108">O padrão de roteamento de tráfego geográfico do hub de Azure Stack ou aplicativos distribuídos geograficamente permite que o tráfego seja direcionado para pontos de extremidade específicos com base em várias métricas.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="f1ea5-109">Criar um Gerenciador de tráfego com roteamento baseado em geográfico e configuração de ponto de extremidade roteia o tráfego para os pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e necessidades de dados.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Padrão distribuído geograficamente](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="f1ea5-111">Componentes</span><span class="sxs-lookup"><span data-stu-id="f1ea5-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="f1ea5-112">Fora da nuvem</span><span class="sxs-lookup"><span data-stu-id="f1ea5-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="f1ea5-113">Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="f1ea5-113">Traffic Manager</span></span>

<span data-ttu-id="f1ea5-114">No diagrama, o Gerenciador de tráfego está localizado fora da nuvem pública, mas ele precisa ser capaz de coordenar o tráfego no datacenter local e na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="f1ea5-115">O balanceador roteia o tráfego para localizações geográficas.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="f1ea5-116">DNS (Sistema de Nomes de Domínio)</span><span class="sxs-lookup"><span data-stu-id="f1ea5-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="f1ea5-117">O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="f1ea5-118">Nuvem pública</span><span class="sxs-lookup"><span data-stu-id="f1ea5-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="f1ea5-119">Ponto de extremidade de nuvem</span><span class="sxs-lookup"><span data-stu-id="f1ea5-119">Cloud Endpoint</span></span>

<span data-ttu-id="f1ea5-120">Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="f1ea5-121">Nuvens locais</span><span class="sxs-lookup"><span data-stu-id="f1ea5-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="f1ea5-122">Ponto de extremidade local</span><span class="sxs-lookup"><span data-stu-id="f1ea5-122">Local endpoint</span></span>

<span data-ttu-id="f1ea5-123">Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="f1ea5-124">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="f1ea5-124">Issues and considerations</span></span>

<span data-ttu-id="f1ea5-125">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="f1ea5-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="f1ea5-126">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="f1ea5-126">Scalability</span></span>

<span data-ttu-id="f1ea5-127">O padrão manipula o roteamento de tráfego geográfico em vez de dimensionar para atender a aumentos no tráfego.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="f1ea5-128">No entanto, você pode combinar esse padrão com outras soluções do Azure e locais.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="f1ea5-129">Por exemplo, esse padrão pode ser usado com o padrão de dimensionamento entre nuvens.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="f1ea5-130">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="f1ea5-130">Availability</span></span>

<span data-ttu-id="f1ea5-131">Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="f1ea5-132">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="f1ea5-132">Manageability</span></span>

<span data-ttu-id="f1ea5-133">O padrão garante o gerenciamento contínuo e a interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="f1ea5-134">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="f1ea5-134">When to use this pattern</span></span>

- <span data-ttu-id="f1ea5-135">Minha organização tem ramificações internacionais que exigem políticas regionais personalizadas de segurança e distribuição.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="f1ea5-136">Cada um dos escritórios da minha organização recebe dados de funcionários, de negócios e de instalações, exigindo a atividade de relatórios por regulamentos locais e fuso horário.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="f1ea5-137">Os requisitos de alta escala podem ser atendidos por meio da expansão horizontal de aplicativos, com várias implantações de aplicativo sendo feitas em uma única região e entre regiões para lidar com requisitos de carga extremo.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="f1ea5-138">Os aplicativos devem ser altamente disponíveis e responsivos às solicitações do cliente, mesmo em interrupções de região única.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f1ea5-139">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="f1ea5-139">Next steps</span></span>

<span data-ttu-id="f1ea5-140">Para saber mais sobre os tópicos apresentados neste artigo:</span><span class="sxs-lookup"><span data-stu-id="f1ea5-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="f1ea5-141">Consulte a [visão geral do Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como esse balanceador de carga de tráfego baseado em DNS funciona.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="f1ea5-142">Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="f1ea5-143">Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="f1ea5-144">Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de aplicativo distribuído geograficamente](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="f1ea5-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="f1ea5-145">O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="f1ea5-146">Você aprende a direcionar o tráfego para pontos de extremidade específicos, com base em várias métricas usando o padrão de aplicativo distribuído geograficamente.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="f1ea5-147">Criar um perfil do Gerenciador de tráfego com roteamento baseado em geográfico e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.</span><span class="sxs-lookup"><span data-stu-id="f1ea5-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
