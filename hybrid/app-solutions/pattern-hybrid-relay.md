---
title: Padrão de retransmissão híbrida no Azure e no Hub de Azure Stack
description: Use o padrão de retransmissão híbrida no Azure e Azure Stack Hub para se conectar aos recursos de borda protegidos por firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909823"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="960db-103">Padrão de retransmissão híbrida</span><span class="sxs-lookup"><span data-stu-id="960db-103">Hybrid relay pattern</span></span>

<span data-ttu-id="960db-104">Saiba como se conectar a recursos de borda ou dispositivos protegidos por firewalls usando o padrão de retransmissão híbrida e a retransmissão do Azure.</span><span class="sxs-lookup"><span data-stu-id="960db-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="960db-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="960db-105">Context and problem</span></span>

<span data-ttu-id="960db-106">Dispositivos de borda geralmente estão protegidos por um firewall corporativo ou dispositivo NAT.</span><span class="sxs-lookup"><span data-stu-id="960db-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="960db-107">Embora eles sejam seguros, eles podem não conseguir se comunicar com os dispositivos de nuvem pública ou de borda em outras redes corporativas.</span><span class="sxs-lookup"><span data-stu-id="960db-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="960db-108">Pode ser necessário expor determinadas portas e funcionalidades para os usuários na nuvem pública de maneira segura.</span><span class="sxs-lookup"><span data-stu-id="960db-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="960db-109">Solução</span><span class="sxs-lookup"><span data-stu-id="960db-109">Solution</span></span>

<span data-ttu-id="960db-110">O padrão de retransmissão híbrida usa a retransmissão do Azure para estabelecer um túnel WebSocket entre dois pontos de extremidade que não podem se comunicar diretamente.</span><span class="sxs-lookup"><span data-stu-id="960db-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="960db-111">Dispositivos que não são locais, mas precisam se conectar a um ponto de extremidade local se conectarão a um ponto de extremidade na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="960db-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="960db-112">Esse ponto de extremidade redirecionará o tráfego em rotas predefinidas por um canal seguro.</span><span class="sxs-lookup"><span data-stu-id="960db-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="960db-113">Um ponto de extremidade dentro do ambiente local recebe o tráfego e o roteia para o destino correto.</span><span class="sxs-lookup"><span data-stu-id="960db-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![arquitetura da solução de padrão de retransmissão híbrida](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="960db-115">Veja como funciona o padrão de retransmissão híbrida:</span><span class="sxs-lookup"><span data-stu-id="960db-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="960db-116">Um dispositivo se conecta à VM (máquina virtual) no Azure, em uma porta predefinida.</span><span class="sxs-lookup"><span data-stu-id="960db-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="960db-117">O tráfego é encaminhado para a retransmissão do Azure no Azure.</span><span class="sxs-lookup"><span data-stu-id="960db-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="960db-118">A VM no Hub Azure Stack, que já estabeleceu uma conexão de vida longa para a retransmissão do Azure, recebe o tráfego e o encaminha para o destino.</span><span class="sxs-lookup"><span data-stu-id="960db-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="960db-119">O serviço ou ponto de extremidade local processa a solicitação.</span><span class="sxs-lookup"><span data-stu-id="960db-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="960db-120">Componentes</span><span class="sxs-lookup"><span data-stu-id="960db-120">Components</span></span>

<span data-ttu-id="960db-121">Essa solução usa os seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="960db-121">This solution uses the following components:</span></span>

| <span data-ttu-id="960db-122">Camada</span><span class="sxs-lookup"><span data-stu-id="960db-122">Layer</span></span> | <span data-ttu-id="960db-123">Componente</span><span class="sxs-lookup"><span data-stu-id="960db-123">Component</span></span> | <span data-ttu-id="960db-124">Descrição</span><span class="sxs-lookup"><span data-stu-id="960db-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="960db-125">Azure</span><span class="sxs-lookup"><span data-stu-id="960db-125">Azure</span></span> | <span data-ttu-id="960db-126">VM do Azure</span><span class="sxs-lookup"><span data-stu-id="960db-126">Azure VM</span></span> | <span data-ttu-id="960db-127">Uma VM do Azure fornece um ponto de extremidade publicamente acessível para o recurso local.</span><span class="sxs-lookup"><span data-stu-id="960db-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="960db-128">Retransmissão do Azure</span><span class="sxs-lookup"><span data-stu-id="960db-128">Azure Relay</span></span> | <span data-ttu-id="960db-129">Uma [retransmissão do Azure](/azure/azure-relay/) fornece a infraestrutura para manter o túnel e a conexão entre a VM do Azure e a VM do Hub de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="960db-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="960db-130">Hub de Azure Stack</span><span class="sxs-lookup"><span data-stu-id="960db-130">Azure Stack Hub</span></span> | <span data-ttu-id="960db-131">Computação</span><span class="sxs-lookup"><span data-stu-id="960db-131">Compute</span></span> | <span data-ttu-id="960db-132">Uma VM de Hub de Azure Stack fornece o lado do servidor do túnel de retransmissão híbrida.</span><span class="sxs-lookup"><span data-stu-id="960db-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="960db-133">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="960db-133">Storage</span></span> | <span data-ttu-id="960db-134">O cluster do mecanismo do AKS implantado no Hub Azure Stack fornece um mecanismo escalonável e resiliente para executar o contêiner API de Detecção Facial.</span><span class="sxs-lookup"><span data-stu-id="960db-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="960db-135">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="960db-135">Issues and considerations</span></span>

<span data-ttu-id="960db-136">Considere os seguintes pontos ao decidir como implementar essa solução:</span><span class="sxs-lookup"><span data-stu-id="960db-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="960db-137">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="960db-137">Scalability</span></span>

<span data-ttu-id="960db-138">Esse padrão só permite mapeamentos de porta 1:1 no cliente e no servidor.</span><span class="sxs-lookup"><span data-stu-id="960db-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="960db-139">Por exemplo, se a porta 80 for encapsulada para um serviço no ponto de extremidade do Azure, ela não poderá ser usada para outro serviço.</span><span class="sxs-lookup"><span data-stu-id="960db-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="960db-140">Os mapeamentos de porta devem ser planejados de acordo.</span><span class="sxs-lookup"><span data-stu-id="960db-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="960db-141">A retransmissão do Azure e as VMs devem ser adequadamente dimensionadas para lidar com o tráfego.</span><span class="sxs-lookup"><span data-stu-id="960db-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="960db-142">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="960db-142">Availability</span></span>

<span data-ttu-id="960db-143">Esses túneis e conexões não são redundantes.</span><span class="sxs-lookup"><span data-stu-id="960db-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="960db-144">Para garantir a alta disponibilidade, talvez você queira implementar o código de verificação de erro.</span><span class="sxs-lookup"><span data-stu-id="960db-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="960db-145">Outra opção é ter um pool de VMs conectadas à retransmissão do Azure atrás de um balanceador de carga.</span><span class="sxs-lookup"><span data-stu-id="960db-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="960db-146">Capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="960db-146">Manageability</span></span>

<span data-ttu-id="960db-147">Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado.</span><span class="sxs-lookup"><span data-stu-id="960db-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="960db-148">Os serviços de IoT do Azure podem automaticamente colocar novos locais e dispositivos online e mantê-los atualizados.</span><span class="sxs-lookup"><span data-stu-id="960db-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="960db-149">Segurança</span><span class="sxs-lookup"><span data-stu-id="960db-149">Security</span></span>

<span data-ttu-id="960db-150">Esse padrão, conforme mostrado, permite o acesso irrestrito a uma porta em um dispositivo interno da borda.</span><span class="sxs-lookup"><span data-stu-id="960db-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="960db-151">Considere adicionar um mecanismo de autenticação ao serviço no dispositivo interno ou na frente do ponto de extremidade de retransmissão híbrida.</span><span class="sxs-lookup"><span data-stu-id="960db-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="960db-152">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="960db-152">Next steps</span></span>

<span data-ttu-id="960db-153">Para saber mais sobre os tópicos apresentados neste artigo:</span><span class="sxs-lookup"><span data-stu-id="960db-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="960db-154">Esse padrão usa a retransmissão do Azure.</span><span class="sxs-lookup"><span data-stu-id="960db-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="960db-155">Para obter mais informações, consulte a [documentação de retransmissão do Azure](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="960db-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="960db-156">Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.</span><span class="sxs-lookup"><span data-stu-id="960db-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="960db-157">Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="960db-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="960db-158">Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de retransmissão híbrida](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="960db-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="960db-159">O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.</span><span class="sxs-lookup"><span data-stu-id="960db-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>