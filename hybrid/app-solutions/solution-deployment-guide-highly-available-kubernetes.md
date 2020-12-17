---
title: Implantar o cluster do Kubernetes de alta disponibilidade no Azure Stack Hub
description: Saiba como implantar uma solução de cluster do Kubernetes para alta disponibilidade usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911917"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="53afc-103">Implantar um cluster do Kubernetes de alta disponibilidade no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="53afc-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="53afc-104">Este artigo vai mostrar como criar um ambiente de cluster do Kubernetes de alta disponibilidade, implantado em várias instâncias do Azure Stack Hub, em diferentes locais físicos.</span><span class="sxs-lookup"><span data-stu-id="53afc-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="53afc-105">Neste guia de implantação de solução, você aprenderá a:</span><span class="sxs-lookup"><span data-stu-id="53afc-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="53afc-106">Baixar e preparar o mecanismo do AKS</span><span class="sxs-lookup"><span data-stu-id="53afc-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="53afc-107">Conectar-se à VM auxiliar do mecanismo do AKS</span><span class="sxs-lookup"><span data-stu-id="53afc-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="53afc-108">Implantar um cluster do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="53afc-109">Conectar-se ao cluster do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="53afc-110">Conectar o Azure Pipelines ao cluster do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="53afc-111">Configurar monitoramento</span><span class="sxs-lookup"><span data-stu-id="53afc-111">Configure monitoring</span></span>
> - <span data-ttu-id="53afc-112">Implantar um aplicativo</span><span class="sxs-lookup"><span data-stu-id="53afc-112">Deploy application</span></span>
> - <span data-ttu-id="53afc-113">Aplicativo de dimensionamento automático</span><span class="sxs-lookup"><span data-stu-id="53afc-113">Autoscale application</span></span>
> - <span data-ttu-id="53afc-114">Configurar o Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="53afc-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="53afc-115">Atualizar o Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="53afc-116">Escalar Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="53afc-117">![Pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="53afc-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="53afc-118">O Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="53afc-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="53afc-119">O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="53afc-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="53afc-120">O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos.</span><span class="sxs-lookup"><span data-stu-id="53afc-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="53afc-121">As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="53afc-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="53afc-122">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="53afc-122">Prerequisites</span></span>

<span data-ttu-id="53afc-123">Antes de começar a usar este guia de implantação, você deve:</span><span class="sxs-lookup"><span data-stu-id="53afc-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="53afc-124">Examinar o artigo [Padrão de cluster do Kubernetes de alta disponibilidade](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="53afc-125">Examinar o conteúdo do [Repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), que contém ativos adicionais referenciados neste artigo.</span><span class="sxs-lookup"><span data-stu-id="53afc-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="53afc-126">Ter uma conta que possa acessar o [Portal do usuário do Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), com pelo menos [permissões de "colaborador"](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="53afc-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="53afc-127">Baixar e preparar o mecanismo do AKS</span><span class="sxs-lookup"><span data-stu-id="53afc-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="53afc-128">O mecanismo do AKS é um binário que pode ser usado de qualquer host Windows ou Linux que possa alcançar os pontos de extremidade do Azure Resource Manager do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="53afc-129">Este guia descreve a implantação de uma nova VM do Linux (ou do Windows) no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="53afc-130">Ela será usada posteriormente, quando o mecanismo do AKS implantar os clusters do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="53afc-131">Você também pode usar uma VM existente do Windows ou do Linux para implantar um cluster do Kubernetes no Azure Stack Hub usando o mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="53afc-132">O processo passo a passo e os requisitos para o mecanismo do AKS estão documentados aqui:</span><span class="sxs-lookup"><span data-stu-id="53afc-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="53afc-133">[Instalar o mecanismo do AKS no Linux no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou usando o [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="53afc-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="53afc-134">O mecanismo do AKS é uma ferramenta auxiliar para implantar e operar clusters do Kubernetes (não gerenciados) no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="53afc-135">Os detalhes e as diferenças do mecanismo do AKS no Azure Stack Hub são descritos aqui:</span><span class="sxs-lookup"><span data-stu-id="53afc-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="53afc-136">O que é o mecanismo do AKS no Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="53afc-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="53afc-137">[Mecanismo de AKS no Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (no GitHub)</span><span class="sxs-lookup"><span data-stu-id="53afc-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="53afc-138">O ambiente de exemplo usará o Terraform para automatizar a implantação da VM do mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="53afc-139">Você pode encontrar os [detalhes e o código no repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="53afc-140">O resultado dessa etapa é um novo grupo de recursos no Azure Stack Hub que contém a VM auxiliar do mecanismo do AKS e os recursos relacionados:</span><span class="sxs-lookup"><span data-stu-id="53afc-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Recursos de VM do mecanismo do AKS no Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="53afc-142">Se você precisar implantar o mecanismo do AKS em um ambiente air-gapped desconectado, examine as [instâncias desconectadas do Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) para saber mais.</span><span class="sxs-lookup"><span data-stu-id="53afc-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="53afc-143">Na próxima etapa, usaremos a VM do mecanismo do AKS implantada recentemente a fim de implantar um cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="53afc-144">Conectar-se à VM auxiliar do mecanismo do AKS</span><span class="sxs-lookup"><span data-stu-id="53afc-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="53afc-145">Primeiro, você deve se conectar à VM auxiliar do mecanismo AKS criada anteriormente.</span><span class="sxs-lookup"><span data-stu-id="53afc-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="53afc-146">A VM deve ter um Endereço IP Público e deve ser acessível via SSH (porta 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="53afc-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Página de visão geral da VM do mecanismo do AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="53afc-148">Você pode usar uma ferramenta de sua escolha, como o MobaXterm, o puTTY ou o PowerShell no Windows 10 para se conectar a uma VM do Linux usando SSH.</span><span class="sxs-lookup"><span data-stu-id="53afc-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="53afc-149">Após a conexão, execute o comando `aks-engine`.</span><span class="sxs-lookup"><span data-stu-id="53afc-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="53afc-150">Acesse [Versões compatíveis do mecanismo do AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para saber mais sobre as versões do mecanismo do AKS e do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Exemplo de linha de comando do mecanismo do AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="53afc-152">Implantar um cluster do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="53afc-153">A própria VM auxiliar do mecanismo do AKS ainda não criou um cluster do Kubernetes no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="53afc-154">A criação do cluster é a primeira ação a ser tomada na VM auxiliar do mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="53afc-155">O processo passo a passo está documentado aqui:</span><span class="sxs-lookup"><span data-stu-id="53afc-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="53afc-156">Implantar um cluster do Kubernetes com o mecanismo do AKS no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="53afc-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="53afc-157">O resultado final do comando `aks-engine deploy` e das preparações nas etapas anteriores é um cluster do Kubernetes com recursos completos, implantado no espaço do locatário da primeira instância do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="53afc-158">O próprio cluster consiste em componentes de IaaS do Azure, como VMs, balanceadores de carga, VNets, discos e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="53afc-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Componentes de IaaS do cluster no portal do Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="53afc-160">Balanceador de carga do Azure (Ponto de Extremidade de API do K8s)</span><span class="sxs-lookup"><span data-stu-id="53afc-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="53afc-161">Nós de trabalho (pool de agentes)</span><span class="sxs-lookup"><span data-stu-id="53afc-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="53afc-162">Nós mestres</span><span class="sxs-lookup"><span data-stu-id="53afc-162">Master Nodes</span></span>

<span data-ttu-id="53afc-163">Agora o cluster está em execução e na próxima etapa faremos a conexão com ele.</span><span class="sxs-lookup"><span data-stu-id="53afc-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="53afc-164">Conectar-se ao cluster do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="53afc-165">Agora você pode se conectar ao cluster do Kubernetes criado anteriormente, seja por meio de SSH (usando a chave SSH especificada como parte da implantação) ou por meio do `kubectl` (recomendado).</span><span class="sxs-lookup"><span data-stu-id="53afc-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="53afc-166">A ferramenta de linha de comando do Kubernetes `kubectl` está disponível para Windows, Linux e macOS [aqui](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="53afc-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="53afc-167">Ela já está pré-instalada e configurada nos nós mestres do cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Executar kubectl no nó mestre](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="53afc-169">Não é recomendável usar o nó mestre como um Jumpbox para tarefas administrativas.</span><span class="sxs-lookup"><span data-stu-id="53afc-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="53afc-170">A configuração `kubectl` é armazenada em `.kube/config` nos nós mestres, bem como na VM do mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="53afc-171">Você pode copiar a configuração para um computador de administração com conectividade para o cluster do Kubernetes e usar o comando `kubectl` lá.</span><span class="sxs-lookup"><span data-stu-id="53afc-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="53afc-172">O arquivo `.kube/config` também é usado posteriormente para configurar uma conexão de serviço no Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="53afc-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="53afc-173">Mantenha esses arquivos seguros porque eles contêm as credenciais para o cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="53afc-174">Um invasor com acesso ao arquivo tem informações suficientes para obter acesso de administrador a ele.</span><span class="sxs-lookup"><span data-stu-id="53afc-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="53afc-175">Todas as ações feitas usando o arquivo `.kube/config` inicial são feitas por meio de uma conta do administrador do cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="53afc-176">Agora você pode tentar vários comandos usando `kubectl` para verificar o status do cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="53afc-177">O Kubernetes tem seu próprio modelo _ *RBAC (controle de acesso baseado em função)* \* que lhe permite criar definições e associações de função bem refinadas.</span><span class="sxs-lookup"><span data-stu-id="53afc-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="53afc-178">Essa é a maneira preferível de controlar o acesso ao cluster em vez de distribuir as permissões de administrador de cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="53afc-179">Conectar o Azure Pipelines a clusters do Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="53afc-180">Para conectar o Azure Pipelines a clusters do Kubernetes implantado recentemente, precisamos do arquivo de configuração de kube (`.kube/config`), conforme explicado na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="53afc-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="53afc-181">Conecte-se a um dos nós mestres do cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="53afc-182">Copie o conteúdo do arquivo `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="53afc-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="53afc-183">Vá para Azure DevOps > Configurações do projeto > Conexões de serviço para criar uma conexão de serviço "Kubernetes" (usar o KubeConfig como método de autenticação)</span><span class="sxs-lookup"><span data-stu-id="53afc-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="53afc-184">O Azure Pipelines (ou seus agentes de build) devem ter acesso à API do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="53afc-185">Se houver uma conexão com a Internet do Azure Pipelines para o cluster do Kubernetes do Azure Stack Hub, você precisará implantar um agente de build do Azure Pipelines auto-hospedado.</span><span class="sxs-lookup"><span data-stu-id="53afc-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="53afc-186">Ao implantar agentes auto-hospedados para o Azure Pipelines, você pode implantá-los no Azure Stack Hub ou em um computador com conectividade de rede para todos os pontos de extremidade de gerenciamento necessários.</span><span class="sxs-lookup"><span data-stu-id="53afc-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="53afc-187">Conferir os detalhes aqui:</span><span class="sxs-lookup"><span data-stu-id="53afc-187">See the details here:</span></span>

* <span data-ttu-id="53afc-188">[Agentes do Azure Pipelines](/azure/devops/pipelines/agents/agents) no [Windows](/azure/devops/pipelines/agents/v2-windows) ou no [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="53afc-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="53afc-189">A seção do padrão [Considerações sobre implantação (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contém um fluxo de decisão que ajuda a entender se devem ser usados agentes hospedados pela Microsoft ou agentes auto-hospedados:</span><span class="sxs-lookup"><span data-stu-id="53afc-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="53afc-190">[![Agentes auto-hospedados do fluxo de decisão](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="53afc-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="53afc-191">Nesta solução de exemplo, a topologia inclui um agente de build auto-hospedado em cada instância do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="53afc-192">O agente pode acessar os pontos de extremidade de gerenciamento do Azure Stack Hub e os pontos de extremidade da API do cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="53afc-193">[![Somente tráfego de saída](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="53afc-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="53afc-194">Esse design atende a um requisito normativo comum, que é ter apenas conexões de saída da solução do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="53afc-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="53afc-195">Configurar monitoramento</span><span class="sxs-lookup"><span data-stu-id="53afc-195">Configure monitoring</span></span>

<span data-ttu-id="53afc-196">Você pode usar o [Azure Monitor](/azure/azure-monitor/) para que os contêineres monitorem os contêineres da solução.</span><span class="sxs-lookup"><span data-stu-id="53afc-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="53afc-197">Isso aponta o Azure Monitor para o cluster do Kubernetes implantado pelo mecanismo do AKS no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="53afc-198">Há duas maneiras de habilitar o Azure Monitor no cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="53afc-199">Ambas as maneiras exigem que você configure um workspace do Log Analytics no Azure.</span><span class="sxs-lookup"><span data-stu-id="53afc-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="53afc-200">O [Método um](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa um gráfico do Helm</span><span class="sxs-lookup"><span data-stu-id="53afc-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="53afc-201">O [Método dois](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) faz parte da especificação de cluster do mecanismo do AKS</span><span class="sxs-lookup"><span data-stu-id="53afc-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="53afc-202">A topologia de exemplo usa o "Método um", o que permite a automação do processo e as atualizações podem ser instaladas com mais facilidade.</span><span class="sxs-lookup"><span data-stu-id="53afc-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="53afc-203">Para a próxima etapa, você precisa de um workspace do Log Analytics no Azure (ID e chave), `Helm` (versão 3) e `kubectl` em seu computador.</span><span class="sxs-lookup"><span data-stu-id="53afc-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="53afc-204">O Helm é um gerenciador de pacotes do Kubernetes, disponível como um binário que é executado no macOS, no Windows e no Linux.</span><span class="sxs-lookup"><span data-stu-id="53afc-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="53afc-205">Ele pode ser baixado aqui: [helm.sh](https://helm.sh/docs/intro/quickstart/) O Helm se baseia no arquivo de configuração do Kubernetes usado para o comando `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="53afc-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="53afc-206">Este comando instalará o agente do Azure Monitor no cluster do Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="53afc-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="53afc-207">O agente do OMS (Operations Management Suite) no cluster do Kubernetes enviará dados de monitoramento para o workspace do Log Analytics no Azure (usando HTTPS de saída).</span><span class="sxs-lookup"><span data-stu-id="53afc-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="53afc-208">Agora você pode usar o Azure Monitor para obter insights mais aprofundados sobre seus clusters do Kubernetes no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="53afc-209">Esse design é uma forma eficiente de demonstrar o poder da análise que pode ser implantada automaticamente com os clusters do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="53afc-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="53afc-210">[![Clusters do Azure Stack Hub no Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="53afc-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="53afc-211">[![Detalhes de cluster do Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="53afc-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="53afc-212">Se o Azure Monitor não mostrar nenhum dado do Azure Stack Hub, certifique-se de ter seguido as instruções em [Como adicionar uma solução AzureMonitor-Containers a um workspace do Log Analytics do Azure](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) com cuidado.</span><span class="sxs-lookup"><span data-stu-id="53afc-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="53afc-213">Implantar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="53afc-213">Deploy the application</span></span>

<span data-ttu-id="53afc-214">Antes de instalar o aplicativo de exemplo, há outra etapa para configurar o controlador de entrada baseado em NGINX no cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="53afc-215">O controlador de entrada é usado como um balanceador de carga de camada 7 para rotear o tráfego no cluster com base no host, no caminho ou no protocolo.</span><span class="sxs-lookup"><span data-stu-id="53afc-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="53afc-216">O ingress do NGINX está disponível como um gráfico do Helm.</span><span class="sxs-lookup"><span data-stu-id="53afc-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="53afc-217">Para obter instruções detalhadas, confira o [Repositório do GitHub do gráfico do Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="53afc-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="53afc-218">O aplicativo de exemplo também é empacotado como um gráfico do Helm, como o [Agente de Monitoramento do Azure](#configure-monitoring) na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="53afc-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="53afc-219">Assim sendo, é simples implantar o aplicativo no cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53afc-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="53afc-220">Você pode encontrar os [Arquivos de gráfico do Helm no repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="53afc-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="53afc-221">O aplicativo de exemplo é um aplicativo de três camadas, implantado em um cluster do Kubernetes em cada uma das duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="53afc-222">O aplicativo usa um banco de dados do MongoDB.</span><span class="sxs-lookup"><span data-stu-id="53afc-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="53afc-223">Você pode aprender mais sobre como obter os dados replicados em várias instâncias no padrão [Considerações sobre dados e armazenamento](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="53afc-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="53afc-224">Depois de implantar o gráfico do Helm para o aplicativo, você verá todas as três camadas do aplicativo representadas como implantações e conjuntos com estado (para o banco de dados) com apenas um pod:</span><span class="sxs-lookup"><span data-stu-id="53afc-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="53afc-225">No lado dos serviços, você vai encontrar o controlador de entrada baseado em NGINX e o endereço IP público dele:</span><span class="sxs-lookup"><span data-stu-id="53afc-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="53afc-226">O endereço "IP Externo" é o "ponto de extremidade do aplicativo".</span><span class="sxs-lookup"><span data-stu-id="53afc-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="53afc-227">É assim que os usuários se conectarão para abrir o aplicativo e também serão usados como o ponto de extremidade para a próxima etapa, [Configurar o Gerenciador de Tráfego](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="53afc-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="53afc-228">Dimensionamento automático do aplicativo</span><span class="sxs-lookup"><span data-stu-id="53afc-228">Autoscale the application</span></span>
<span data-ttu-id="53afc-229">Opcionalmente, você pode configurar o [Dimensionador Automático de Pod Horizontal](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar ou reduzir verticalmente com base em determinadas métricas, como a utilização da CPU.</span><span class="sxs-lookup"><span data-stu-id="53afc-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="53afc-230">O comando a seguir criará um Dimensionador Automático de Pod Horizontal que mantém de 1 a 10 réplicas dos Pods controladas pela implantação de classificações Web.</span><span class="sxs-lookup"><span data-stu-id="53afc-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="53afc-231">O HPA aumentará e diminuirá o número de réplicas (por meio da implantação) para manter uma utilização média da CPU em todos os pods na faixa de 80%.</span><span class="sxs-lookup"><span data-stu-id="53afc-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="53afc-232">Você pode verificar o status atual do dimensionador automático executando-o:</span><span class="sxs-lookup"><span data-stu-id="53afc-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="53afc-233">Configurar o Gerenciador de Tráfego</span><span class="sxs-lookup"><span data-stu-id="53afc-233">Configure Traffic Manager</span></span>

<span data-ttu-id="53afc-234">Para distribuir o tráfego entre duas (ou mais) implantações do aplicativo, usaremos o [Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="53afc-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="53afc-235">O Gerenciador de tráfego do Azure é um balanceador de carga de tráfego baseado em DNS no Azure.</span><span class="sxs-lookup"><span data-stu-id="53afc-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="53afc-236">O Gerenciador de Tráfego usa o DNS para direcionar as solicitações do cliente ao ponto de extremidade de serviço mais apropriado com base em um método de roteamento de tráfego e na integridade dos pontos de extremidade.</span><span class="sxs-lookup"><span data-stu-id="53afc-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="53afc-237">Em vez de usar o Gerenciador de Tráfego do Azure, você também pode usar outras soluções de balanceamento de carga global hospedadas no local.</span><span class="sxs-lookup"><span data-stu-id="53afc-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="53afc-238">No cenário de exemplo, usaremos o Gerenciador de Tráfego do Azure para distribuir o tráfego entre duas instâncias do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="53afc-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="53afc-239">Eles podem ser executados em instâncias do Azure Stack Hub no mesmo local ou em locais diferentes:</span><span class="sxs-lookup"><span data-stu-id="53afc-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Gerenciador de tráfego local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="53afc-241">No Azure, configuramos o Gerenciador de Tráfego para apontar para as duas instâncias diferentes de aplicativo:</span><span class="sxs-lookup"><span data-stu-id="53afc-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="53afc-242">[![Perfil do ponto de extremidade do TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="53afc-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="53afc-243">Como você pode ver, os dois pontos de extremidade apontam para as duas instâncias do aplicativo implantado da [seção anterior](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="53afc-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="53afc-244">Neste ponto:</span><span class="sxs-lookup"><span data-stu-id="53afc-244">At this point:</span></span>
- <span data-ttu-id="53afc-245">A infraestrutura do Kubernetes já foi criada, incluindo um controlador de entrada.</span><span class="sxs-lookup"><span data-stu-id="53afc-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="53afc-246">Os clusters já foram implantados em duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="53afc-247">O monitoramento já foi configurado.</span><span class="sxs-lookup"><span data-stu-id="53afc-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="53afc-248">O Gerenciador de Tráfego do Azure balanceará a carga do tráfego entre as duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="53afc-249">Além dessa infraestrutura, o aplicativo de três camadas de exemplo já foi implantado de maneira automatizada usando os gráficos do Helm.</span><span class="sxs-lookup"><span data-stu-id="53afc-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="53afc-250">A solução agora deve estar ativa e acessível aos usuários.</span><span class="sxs-lookup"><span data-stu-id="53afc-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="53afc-251">Há também algumas considerações operacionais referentes ao pós-implantação que vale a pena discutir, as quais são abordadas nas próximas duas seções.</span><span class="sxs-lookup"><span data-stu-id="53afc-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="53afc-252">Atualizar o Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="53afc-253">Considere os seguintes tópicos ao atualizar o cluster do Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="53afc-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="53afc-254">A atualização de um cluster do Kubernetes é uma operação complexa do segundo dia que pode ser realizada usando o mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="53afc-255">Para obter mais informações, confira [Atualizar um cluster do Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="53afc-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="53afc-256">O mecanismo do AKS permite que você atualize os clusters para versões mais recentes do Kubernetes e da imagem do sistema operacional de base.</span><span class="sxs-lookup"><span data-stu-id="53afc-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="53afc-257">Para obter mais informações, confira [Etapas para atualizar para uma versão mais recente do Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="53afc-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="53afc-258">Você também pode atualizar apenas os nós subjacentes para versões mais recentes da imagem do sistema operacional de base.</span><span class="sxs-lookup"><span data-stu-id="53afc-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="53afc-259">Para obter mais informações, confira [Etapas para atualizar apenas a imagem do sistema operacional](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="53afc-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="53afc-260">As imagens mais recentes do sistema operacional de base contêm atualizações de segurança e do kernel.</span><span class="sxs-lookup"><span data-stu-id="53afc-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="53afc-261">É responsabilidade do operador do cluster monitorar a disponibilidade de versões mais recentes do Kubernetes e das imagens do sistema operacional.</span><span class="sxs-lookup"><span data-stu-id="53afc-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="53afc-262">O operador deve planejar e executar essas atualizações usando o mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="53afc-263">As imagens do sistema operacional base devem ser baixadas do Marketplace do Azure Stack Hub pelo operador do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="53afc-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="53afc-264">Escalar Kubernetes</span><span class="sxs-lookup"><span data-stu-id="53afc-264">Scale Kubernetes</span></span>

<span data-ttu-id="53afc-265">A escala é outra operação do segundo dia que pode ser orquestrada usando o mecanismo do AKS.</span><span class="sxs-lookup"><span data-stu-id="53afc-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="53afc-266">O comando de escala reutiliza o arquivo de configuração do cluster (apimodel.json) no diretório de saída, como entrada para uma nova implantação do Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="53afc-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="53afc-267">O mecanismo do AKS executa a operação de escala em um pool de agentes específico.</span><span class="sxs-lookup"><span data-stu-id="53afc-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="53afc-268">Quando a operação de escala for concluída, o mecanismo do AKS atualizará a definição do cluster no mesmo arquivo apimodel.json.</span><span class="sxs-lookup"><span data-stu-id="53afc-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="53afc-269">A definição do cluster reflete a nova contagem de nós para refletir a configuração atualizada do cluster.</span><span class="sxs-lookup"><span data-stu-id="53afc-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="53afc-270">Escalar um cluster do Kubernetes no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="53afc-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="53afc-271">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="53afc-271">Next steps</span></span>

- <span data-ttu-id="53afc-272">Saiba mais sobre as [Considerações de design do aplicativo híbrido](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="53afc-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="53afc-273">Revise e proponha melhorias para [o código desse exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="53afc-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>