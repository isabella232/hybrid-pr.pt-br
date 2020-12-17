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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Implantar um cluster do Kubernetes de alta disponibilidade no Azure Stack Hub

Este artigo vai mostrar como criar um ambiente de cluster do Kubernetes de alta disponibilidade, implantado em várias instâncias do Azure Stack Hub, em diferentes locais físicos.

Neste guia de implantação de solução, você aprenderá a:

> [!div class="checklist"]
> - Baixar e preparar o mecanismo do AKS
> - Conectar-se à VM auxiliar do mecanismo do AKS
> - Implantar um cluster do Kubernetes
> - Conectar-se ao cluster do Kubernetes
> - Conectar o Azure Pipelines ao cluster do Kubernetes
> - Configurar monitoramento
> - Implantar um aplicativo
> - Aplicativo de dimensionamento automático
> - Configurar o Gerenciador de Tráfego
> - Atualizar o Kubernetes
> - Escalar Kubernetes

> [!Tip]  
> ![Pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar a usar este guia de implantação, você deve:

- Examinar o artigo [Padrão de cluster do Kubernetes de alta disponibilidade](pattern-highly-available-kubernetes.md).
- Examinar o conteúdo do [Repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), que contém ativos adicionais referenciados neste artigo.
- Ter uma conta que possa acessar o [Portal do usuário do Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), com pelo menos [permissões de "colaborador"](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Baixar e preparar o mecanismo do AKS

O mecanismo do AKS é um binário que pode ser usado de qualquer host Windows ou Linux que possa alcançar os pontos de extremidade do Azure Resource Manager do Azure Stack Hub. Este guia descreve a implantação de uma nova VM do Linux (ou do Windows) no Azure Stack Hub. Ela será usada posteriormente, quando o mecanismo do AKS implantar os clusters do Kubernetes.

> [!NOTE]
> Você também pode usar uma VM existente do Windows ou do Linux para implantar um cluster do Kubernetes no Azure Stack Hub usando o mecanismo do AKS.

O processo passo a passo e os requisitos para o mecanismo do AKS estão documentados aqui:

* [Instalar o mecanismo do AKS no Linux no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou usando o [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

O mecanismo do AKS é uma ferramenta auxiliar para implantar e operar clusters do Kubernetes (não gerenciados) no Azure e no Azure Stack Hub.

Os detalhes e as diferenças do mecanismo do AKS no Azure Stack Hub são descritos aqui:

* [O que é o mecanismo do AKS no Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Mecanismo de AKS no Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (no GitHub)

O ambiente de exemplo usará o Terraform para automatizar a implantação da VM do mecanismo do AKS. Você pode encontrar os [detalhes e o código no repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

O resultado dessa etapa é um novo grupo de recursos no Azure Stack Hub que contém a VM auxiliar do mecanismo do AKS e os recursos relacionados:

![Recursos de VM do mecanismo do AKS no Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Se você precisar implantar o mecanismo do AKS em um ambiente air-gapped desconectado, examine as [instâncias desconectadas do Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) para saber mais.

Na próxima etapa, usaremos a VM do mecanismo do AKS implantada recentemente a fim de implantar um cluster do Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Conectar-se à VM auxiliar do mecanismo do AKS

Primeiro, você deve se conectar à VM auxiliar do mecanismo AKS criada anteriormente.

A VM deve ter um Endereço IP Público e deve ser acessível via SSH (porta 22/TCP).

![Página de visão geral da VM do mecanismo do AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Você pode usar uma ferramenta de sua escolha, como o MobaXterm, o puTTY ou o PowerShell no Windows 10 para se conectar a uma VM do Linux usando SSH.

```console
ssh <username>@<ipaddress>
```

Após a conexão, execute o comando `aks-engine`. Acesse [Versões compatíveis do mecanismo do AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para saber mais sobre as versões do mecanismo do AKS e do Kubernetes.

![Exemplo de linha de comando do mecanismo do AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Implantar um cluster do Kubernetes

A própria VM auxiliar do mecanismo do AKS ainda não criou um cluster do Kubernetes no Azure Stack Hub. A criação do cluster é a primeira ação a ser tomada na VM auxiliar do mecanismo do AKS.

O processo passo a passo está documentado aqui:

* [Implantar um cluster do Kubernetes com o mecanismo do AKS no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

O resultado final do comando `aks-engine deploy` e das preparações nas etapas anteriores é um cluster do Kubernetes com recursos completos, implantado no espaço do locatário da primeira instância do Azure Stack Hub. O próprio cluster consiste em componentes de IaaS do Azure, como VMs, balanceadores de carga, VNets, discos e assim por diante.

![Componentes de IaaS do cluster no portal do Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Balanceador de carga do Azure (Ponto de Extremidade de API do K8s)
2) Nós de trabalho (pool de agentes)
3) Nós mestres

Agora o cluster está em execução e na próxima etapa faremos a conexão com ele.

## <a name="connect-to-the-kubernetes-cluster"></a>Conectar-se ao cluster do Kubernetes

Agora você pode se conectar ao cluster do Kubernetes criado anteriormente, seja por meio de SSH (usando a chave SSH especificada como parte da implantação) ou por meio do `kubectl` (recomendado). A ferramenta de linha de comando do Kubernetes `kubectl` está disponível para Windows, Linux e macOS [aqui](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Ela já está pré-instalada e configurada nos nós mestres do cluster.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Executar kubectl no nó mestre](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Não é recomendável usar o nó mestre como um Jumpbox para tarefas administrativas. A configuração `kubectl` é armazenada em `.kube/config` nos nós mestres, bem como na VM do mecanismo do AKS. Você pode copiar a configuração para um computador de administração com conectividade para o cluster do Kubernetes e usar o comando `kubectl` lá. O arquivo `.kube/config` também é usado posteriormente para configurar uma conexão de serviço no Azure Pipelines.

> [!IMPORTANT]
> Mantenha esses arquivos seguros porque eles contêm as credenciais para o cluster do Kubernetes. Um invasor com acesso ao arquivo tem informações suficientes para obter acesso de administrador a ele. Todas as ações feitas usando o arquivo `.kube/config` inicial são feitas por meio de uma conta do administrador do cluster.

Agora você pode tentar vários comandos usando `kubectl` para verificar o status do cluster.

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
> O Kubernetes tem seu próprio modelo _ *RBAC (controle de acesso baseado em função)* * que lhe permite criar definições e associações de função bem refinadas. Essa é a maneira preferível de controlar o acesso ao cluster em vez de distribuir as permissões de administrador de cluster.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Conectar o Azure Pipelines a clusters do Kubernetes

Para conectar o Azure Pipelines a clusters do Kubernetes implantado recentemente, precisamos do arquivo de configuração de kube (`.kube/config`), conforme explicado na etapa anterior.

* Conecte-se a um dos nós mestres do cluster do Kubernetes.
* Copie o conteúdo do arquivo `.kube/config`.
* Vá para Azure DevOps > Configurações do projeto > Conexões de serviço para criar uma conexão de serviço "Kubernetes" (usar o KubeConfig como método de autenticação)

> [!IMPORTANT]
> O Azure Pipelines (ou seus agentes de build) devem ter acesso à API do Kubernetes. Se houver uma conexão com a Internet do Azure Pipelines para o cluster do Kubernetes do Azure Stack Hub, você precisará implantar um agente de build do Azure Pipelines auto-hospedado.

Ao implantar agentes auto-hospedados para o Azure Pipelines, você pode implantá-los no Azure Stack Hub ou em um computador com conectividade de rede para todos os pontos de extremidade de gerenciamento necessários. Conferir os detalhes aqui:

* [Agentes do Azure Pipelines](/azure/devops/pipelines/agents/agents) no [Windows](/azure/devops/pipelines/agents/v2-windows) ou no [Linux](/azure/devops/pipelines/agents/v2-linux)

A seção do padrão [Considerações sobre implantação (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contém um fluxo de decisão que ajuda a entender se devem ser usados agentes hospedados pela Microsoft ou agentes auto-hospedados:

[![Agentes auto-hospedados do fluxo de decisão](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

Nesta solução de exemplo, a topologia inclui um agente de build auto-hospedado em cada instância do Azure Stack Hub. O agente pode acessar os pontos de extremidade de gerenciamento do Azure Stack Hub e os pontos de extremidade da API do cluster do Kubernetes.

[![Somente tráfego de saída](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Esse design atende a um requisito normativo comum, que é ter apenas conexões de saída da solução do aplicativo.

## <a name="configure-monitoring"></a>Configurar monitoramento

Você pode usar o [Azure Monitor](/azure/azure-monitor/) para que os contêineres monitorem os contêineres da solução. Isso aponta o Azure Monitor para o cluster do Kubernetes implantado pelo mecanismo do AKS no Azure Stack Hub.

Há duas maneiras de habilitar o Azure Monitor no cluster. Ambas as maneiras exigem que você configure um workspace do Log Analytics no Azure.

* O [Método um](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa um gráfico do Helm
* O [Método dois](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) faz parte da especificação de cluster do mecanismo do AKS

A topologia de exemplo usa o "Método um", o que permite a automação do processo e as atualizações podem ser instaladas com mais facilidade.

Para a próxima etapa, você precisa de um workspace do Log Analytics no Azure (ID e chave), `Helm` (versão 3) e `kubectl` em seu computador.

O Helm é um gerenciador de pacotes do Kubernetes, disponível como um binário que é executado no macOS, no Windows e no Linux. Ele pode ser baixado aqui: [helm.sh](https://helm.sh/docs/intro/quickstart/) O Helm se baseia no arquivo de configuração do Kubernetes usado para o comando `kubectl`.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Este comando instalará o agente do Azure Monitor no cluster do Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

O agente do OMS (Operations Management Suite) no cluster do Kubernetes enviará dados de monitoramento para o workspace do Log Analytics no Azure (usando HTTPS de saída). Agora você pode usar o Azure Monitor para obter insights mais aprofundados sobre seus clusters do Kubernetes no Azure Stack Hub. Esse design é uma forma eficiente de demonstrar o poder da análise que pode ser implantada automaticamente com os clusters do aplicativo.

[![Clusters do Azure Stack Hub no Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Detalhes de cluster do Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Se o Azure Monitor não mostrar nenhum dado do Azure Stack Hub, certifique-se de ter seguido as instruções em [Como adicionar uma solução AzureMonitor-Containers a um workspace do Log Analytics do Azure](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) com cuidado.

## <a name="deploy-the-application"></a>Implantar o aplicativo

Antes de instalar o aplicativo de exemplo, há outra etapa para configurar o controlador de entrada baseado em NGINX no cluster do Kubernetes. O controlador de entrada é usado como um balanceador de carga de camada 7 para rotear o tráfego no cluster com base no host, no caminho ou no protocolo. O ingress do NGINX está disponível como um gráfico do Helm. Para obter instruções detalhadas, confira o [Repositório do GitHub do gráfico do Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

O aplicativo de exemplo também é empacotado como um gráfico do Helm, como o [Agente de Monitoramento do Azure](#configure-monitoring) na etapa anterior. Assim sendo, é simples implantar o aplicativo no cluster do Kubernetes. Você pode encontrar os [Arquivos de gráfico do Helm no repositório complementar do GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

O aplicativo de exemplo é um aplicativo de três camadas, implantado em um cluster do Kubernetes em cada uma das duas instâncias do Azure Stack Hub. O aplicativo usa um banco de dados do MongoDB. Você pode aprender mais sobre como obter os dados replicados em várias instâncias no padrão [Considerações sobre dados e armazenamento](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Depois de implantar o gráfico do Helm para o aplicativo, você verá todas as três camadas do aplicativo representadas como implantações e conjuntos com estado (para o banco de dados) com apenas um pod:

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

No lado dos serviços, você vai encontrar o controlador de entrada baseado em NGINX e o endereço IP público dele:

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

O endereço "IP Externo" é o "ponto de extremidade do aplicativo". É assim que os usuários se conectarão para abrir o aplicativo e também serão usados como o ponto de extremidade para a próxima etapa, [Configurar o Gerenciador de Tráfego](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Dimensionamento automático do aplicativo
Opcionalmente, você pode configurar o [Dimensionador Automático de Pod Horizontal](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar ou reduzir verticalmente com base em determinadas métricas, como a utilização da CPU. O comando a seguir criará um Dimensionador Automático de Pod Horizontal que mantém de 1 a 10 réplicas dos Pods controladas pela implantação de classificações Web. O HPA aumentará e diminuirá o número de réplicas (por meio da implantação) para manter uma utilização média da CPU em todos os pods na faixa de 80%.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Você pode verificar o status atual do dimensionador automático executando-o:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Configurar o Gerenciador de Tráfego

Para distribuir o tráfego entre duas (ou mais) implantações do aplicativo, usaremos o [Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-overview). O Gerenciador de tráfego do Azure é um balanceador de carga de tráfego baseado em DNS no Azure.

> [!NOTE]
> O Gerenciador de Tráfego usa o DNS para direcionar as solicitações do cliente ao ponto de extremidade de serviço mais apropriado com base em um método de roteamento de tráfego e na integridade dos pontos de extremidade.

Em vez de usar o Gerenciador de Tráfego do Azure, você também pode usar outras soluções de balanceamento de carga global hospedadas no local. No cenário de exemplo, usaremos o Gerenciador de Tráfego do Azure para distribuir o tráfego entre duas instâncias do aplicativo. Eles podem ser executados em instâncias do Azure Stack Hub no mesmo local ou em locais diferentes:

![Gerenciador de tráfego local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

No Azure, configuramos o Gerenciador de Tráfego para apontar para as duas instâncias diferentes de aplicativo:

[![Perfil do ponto de extremidade do TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Como você pode ver, os dois pontos de extremidade apontam para as duas instâncias do aplicativo implantado da [seção anterior](#deploy-the-application).

Neste ponto:
- A infraestrutura do Kubernetes já foi criada, incluindo um controlador de entrada.
- Os clusters já foram implantados em duas instâncias do Azure Stack Hub.
- O monitoramento já foi configurado.
- O Gerenciador de Tráfego do Azure balanceará a carga do tráfego entre as duas instâncias do Azure Stack Hub.
- Além dessa infraestrutura, o aplicativo de três camadas de exemplo já foi implantado de maneira automatizada usando os gráficos do Helm. 

A solução agora deve estar ativa e acessível aos usuários.

Há também algumas considerações operacionais referentes ao pós-implantação que vale a pena discutir, as quais são abordadas nas próximas duas seções.

## <a name="upgrade-kubernetes"></a>Atualizar o Kubernetes

Considere os seguintes tópicos ao atualizar o cluster do Kubernetes:

- A atualização de um cluster do Kubernetes é uma operação complexa do segundo dia que pode ser realizada usando o mecanismo do AKS. Para obter mais informações, confira [Atualizar um cluster do Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- O mecanismo do AKS permite que você atualize os clusters para versões mais recentes do Kubernetes e da imagem do sistema operacional de base. Para obter mais informações, confira [Etapas para atualizar para uma versão mais recente do Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Você também pode atualizar apenas os nós subjacentes para versões mais recentes da imagem do sistema operacional de base. Para obter mais informações, confira [Etapas para atualizar apenas a imagem do sistema operacional](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

As imagens mais recentes do sistema operacional de base contêm atualizações de segurança e do kernel. É responsabilidade do operador do cluster monitorar a disponibilidade de versões mais recentes do Kubernetes e das imagens do sistema operacional. O operador deve planejar e executar essas atualizações usando o mecanismo do AKS. As imagens do sistema operacional base devem ser baixadas do Marketplace do Azure Stack Hub pelo operador do Azure Stack Hub.

## <a name="scale-kubernetes"></a>Escalar Kubernetes

A escala é outra operação do segundo dia que pode ser orquestrada usando o mecanismo do AKS.

O comando de escala reutiliza o arquivo de configuração do cluster (apimodel.json) no diretório de saída, como entrada para uma nova implantação do Azure Resource Manager. O mecanismo do AKS executa a operação de escala em um pool de agentes específico. Quando a operação de escala for concluída, o mecanismo do AKS atualizará a definição do cluster no mesmo arquivo apimodel.json. A definição do cluster reflete a nova contagem de nós para refletir a configuração atualizada do cluster.

- [Escalar um cluster do Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre as [Considerações de design do aplicativo híbrido](overview-app-design-considerations.md)
- Revise e proponha melhorias para [o código desse exemplo no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).