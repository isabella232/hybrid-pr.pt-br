---
title: Padrão do Kubernetes de alta disponibilidade usando o Azure e o Azure Stack Hub
description: Saiba como uma solução de cluster do Kubernetes fornece alta disponibilidade usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281305"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Padrão de cluster do Kubernetes de alta disponibilidade

Este artigo descreve como arquitetar e operar uma infraestrutura altamente disponível baseada no Kubernetes usando o Mecanismo do AKS (Serviço de Kubernetes do Azure) no Azure Stack Hub. Esse cenário é comum para as organizações com cargas de trabalho críticas em ambientes altamente restritos e regulamentados. Organizações em domínios como finanças, defesa e governo.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações estão desenvolvendo soluções nativas de nuvem que aproveitam serviços e tecnologias de ponta como o Kubernetes. Embora o Azure forneça datacenters na maioria das regiões do mundo, às vezes, há casos de uso de borda e cenários em que aplicativos comercialmente críticos precisam ser executados em uma localização específica. Entre as considerações estão:

- Confidencialidade de localização
- Latência entre o aplicativo e os sistemas locais
- Conservação da largura de banda
- Conectividade
- Requisitos regulatórios ou legais

O Azure, em combinação com o Azure Stack Hub, resolve a maioria dessas preocupações. Um amplo conjunto de opções, decisões e considerações para uma implementação bem-sucedida do Kubernetes em execução no Azure Stack Hub é descrito abaixo.

## <a name="solution"></a>Solução

Esse padrão pressupõe que precisamos lidar com um conjunto estrito de restrições. O aplicativo precisa ser executado no local e todos os dados pessoais não devem alcançar serviços de nuvem pública. O monitoramento e outros dados não PII podem ser enviados para o Azure e ser processados nele. Serviços externos, como um Registro de Contêiner público ou outros, podem ser acessados, mas podem ser filtrados por meio de um firewall ou um servidor proxy.

O aplicativo de exemplo mostrado aqui (com base no [Workshop do Serviço de Kubernetes do Azure](/learn/modules/aks-workshop/)) foi projetado para usar soluções nativas do Kubernetes sempre que possível. Esse design evita o bloqueio de fornecedor, em vez de usar serviços nativos de plataforma. Por exemplo, o aplicativo usa um back-end do banco de dados MongoDB auto-hospedado, em vez de um serviço de PaaS ou um serviço de banco de dados externo.

[![Padrão de aplicativo híbrido](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

O diagrama anterior ilustra a arquitetura do aplicativo de exemplo em execução no Kubernetes no Azure Stack Hub. O aplicativo consiste em vários componentes, incluindo:

 1) Um Mecanismo do AKS baseado em um cluster do Kubernetes no Azure Stack Hub.
 2) O [cert-manager](https://www.jetstack.io/cert-manager/), que fornece um pacote de ferramentas para o gerenciamento de certificados no Kubernetes, usado para solicitar automaticamente certificados do Let's Encrypt.
 3) Um namespace do Kubernetes que contém os componentes do aplicativo para o front-end (ratings-web), a API (ratings-api) e o banco de dados (ratings-mongodb).
 4) O controlador de entrada que roteia o tráfego HTTP/HTTPS para pontos de extremidade no cluster do Kubernetes.

O aplicativo de exemplo é usado para ilustrar a arquitetura do aplicativo. Todos os componentes são exemplos. A arquitetura contém apenas uma implantação de aplicativo. Para obter HA (alta disponibilidade), executaremos a implantação, pelo menos, duas vezes em duas instâncias diferentes do Azure Stack Hub: elas podem ser executadas na mesma localização ou em dois (ou mais) sites diferentes:

![Arquitetura de infraestrutura](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Serviços como o Registro de Contêiner do Azure, o Azure Monitor e outros são hospedados fora do Azure Stack Hub no Azure ou no local. Esse design híbrido protege a solução contra a interrupção de uma instância individual do Azure Stack Hub.

## <a name="components"></a>Componentes

A arquitetura geral é formada pelos seguintes componentes:

O **Azure Stack Hub** é uma extensão do Azure que pode executar cargas de trabalho em um ambiente local fornecendo serviços do Azure no seu datacenter. Acesse [Visão geral do Azure Stack Hub](/azure-stack/operator/azure-stack-overview) para saber mais.

O **Mecanismo do AKS (Serviço de Kubernetes do Azure)** é o mecanismo por trás da oferta de serviço Kubernetes gerenciado, o AKS (Serviço de Kubernetes do Azure), que está disponível hoje no Azure. Para o Azure Stack Hub, o Mecanismo do AKS nos permite implantar, escalar e atualizar clusters do Kubernetes autogerenciados completos usando as funcionalidades de IaaS do Azure Stack Hub. Acesse [Visão geral do Mecanismo do AKS](https://github.com/Azure/aks-engine) para saber mais.

Acesse [Problemas conhecidos e limitações](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) para saber mais sobre as diferenças entre o Mecanismo do AKS no Azure e o Mecanismo do AKS no Azure Stack Hub.

A **VNet (Rede Virtual) do Azure** é usada para fornecer a infraestrutura de rede em cada Azure Stack Hub para as VMs (Máquinas Virtuais) que hospedam a infraestrutura de cluster do Kubernetes.

O **Azure Load Balancer** é usado para o Ponto de Extremidade de API do Kubernetes e o controlador de entrada NGINX. O balanceador de carga roteia o tráfego externo (por exemplo, Internet) para os nós e as VMs que oferecem um serviço específico.

O **ACR (Registro de Contêiner do Azure)** é usado para armazenar imagens privadas do Docker e gráficos do Helm, que são implantados no cluster. O Mecanismo do AKS pode se autenticar no Registro de Contêiner usando uma identidade do Azure AD. O Kubernetes não exige o ACR. Você pode usar outros registros de contêiner, como o Hub do Docker.

O **Azure Repos** é um conjunto de ferramentas de controle de versão que você pode usar para gerenciar seu código. Use também o GitHub ou outros repositórios baseados no Git. Acesse [Visão geral do Azure Repos](/azure/devops/repos/get-started/what-is-repos) para saber mais.

O **Azure Pipelines** faz parte do Azure DevOps Services e executa builds, implantações e testes automatizados. Você também pode usar soluções de CI/CD de terceiros, como o Jenkins. Acesse [Visão geral do Azure Pipeline](/azure/devops/pipelines/get-started/what-is-azure-pipelines) para saber mais.

O **Azure Monitor** coleta e armazena métricas e logs, incluindo a métrica de plataforma para os serviços do Azure na telemetria do aplicativo e da solução. Use esses dados para monitorar o aplicativo, configurar alertas e painéis e executar a análise da causa raiz de falhas. O Azure Monitor integra-se ao Kubernetes para coletar métricas de controladores, nós e contêineres, bem como logs do contêiner e logs do nó mestre. Acesse [Visão geral do Azure Monitor](/azure/azure-monitor/overview) para saber mais.

O **Gerenciador de Tráfego do Azure** é um balanceador de carga de tráfego baseado em DNS que permite distribuir o tráfego de maneira ideal para serviços em diferentes regiões do Azure ou implantações do Azure Stack Hub. O Gerenciador de Tráfego também fornece alta disponibilidade e capacidade de resposta. Os pontos de extremidade do aplicativo precisam ser acessíveis externamente. Também há outras soluções locais disponíveis.

O **controlador de entrada do Kubernetes** expõe as rotas HTTP(S) aos serviços em um cluster do Kubernetes. O NGINX ou qualquer controlador de entrada adequado pode ser usado para essa finalidade.

O **Helm** é um gerenciador de pacotes para a implantação do Kubernetes, fornecendo uma forma de agrupar diferentes objetos do Kubernetes, como implantações, serviços, segredos, em um só "gráfico". Você pode publicar, implantar e atualizar um objeto de gráfico, além de controlar o gerenciamento de versão dele. O Registro de Contêiner do Azure pode ser usado como um repositório para armazenar gráficos empacotados do Helm.

## <a name="design-considerations"></a>Considerações sobre o design

Esse padrão segue algumas considerações de alto nível explicadas mais detalhadamente nas próximas seções deste artigo:

- O aplicativo usa soluções nativas do Kubernetes para evitar o bloqueio de fornecedor.
- Ele usa uma arquitetura de microsserviços.
- O Azure Stack Hub não precisa de entrada, mas permite a conectividade de saída com a Internet.

Essas práticas recomendadas também serão aplicadas a cargas de trabalho e cenários do mundo real.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

A escalabilidade é importante para fornecer aos usuários acesso consistente, confiável e de bom desempenho ao aplicativo.

O cenário de exemplo aborda a escalabilidade em várias camadas da pilha de aplicativos. Esta é uma visão geral de alto nível das diferentes camadas:

| Nível de arquitetura | Afeta | Como posso fazer isso? |
| --- | --- | ---
| Aplicativo | Aplicativo | Escala horizontal com base no número de pods/réplicas/Instâncias de Contêiner* |
| Cluster | Cluster do Kubernetes | Número de nós (entre 1 e 50), tamanhos de SKU da VM e pools de nós (atualmente, o Mecanismo do AKS no Azure Stack Hub só dá suporte a um pool de nós); uso do comando scale do Mecanismo do AKS (manual) |
| Infraestrutura | Azure Stack Hub | Número de nós, capacidade e unidades de escala em uma implantação do Azure Stack Hub |

\* Uso do HPA (Dimensionador Automático de Pod Horizontal) do Kubernetes; escala automatizada baseada em métrica ou escala vertical pelo dimensionamento das instâncias de contêiner (CPU/memória).

**Azure Stack Hub (nível de infraestrutura)**

A infraestrutura do Azure Stack Hub é a base dessa implementação, porque o Azure Stack Hub é executado em um hardware físico em um datacenter. Ao selecionar o hardware do Hub, você precisa fazer escolhas para CPU, densidade de memória, configuração de armazenamento e número de servidores. Para saber mais sobre a escalabilidade do Azure Stack Hub, confira os seguintes recursos:

- [Visão geral do planejamento da capacidade para o Azure Stack Hub](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Adicionar mais nós de unidade de escala no Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Cluster do Kubernetes (nível de cluster)**

O próprio cluster do Kubernetes é formado por componentes de IaaS (pilha) do Azure, incluindo recursos de computação, armazenamento e rede, e é criado com base neles. As soluções do Kubernetes envolvem nós mestres e de trabalho, que são implantados como VMs no Azure (e no Azure Stack Hub).

- Os [nós do painel de controle](/azure/aks/concepts-clusters-workloads#control-plane) (mestre) fornecem os serviços principais do Kubernetes e a orquestração de cargas de trabalho do aplicativo.
- Os [nós de trabalho](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (trabalho) executam as cargas de trabalhos do aplicativo.

Ao selecionar tamanhos de VM para a implantação inicial, há várias considerações:  

- **Custo**: ao planejar seus nós de trabalho, tenha em mente o custo geral por VM que você pagará. Por exemplo, se as cargas de trabalho do aplicativo exigirem recursos limitados, você deverá planejar a implantação de VMs menores. O Azure Stack Hub, como o Azure, normalmente é cobrado de acordo com o consumo. Portanto, o dimensionamento adequado das VMs para as funções do Kubernetes é crucial para otimizar os custos de consumo. 

- **Escalabilidade**: a escalabilidade do cluster é obtida pela escala e pela redução do número de nós mestres e de trabalho ou pela adição de pools de nós extras (não disponíveis atualmente no Azure Stack Hub). A escala do cluster pode ser feita com base nos dados de desempenho, coletados com o uso de Insights de Contêiner (Azure Monitor + Log Analytics). 

    Se o seu aplicativo precisar de mais (ou menos) recursos, escale ou reduza horizontalmente os nós atuais (entre 1 e 50 nós). Caso precise de mais de 50 nós, crie um cluster adicional em uma assinatura separada. Você não poderá escalar verticalmente as VMs reais para outro tamanho de VM sem reimplantar o cluster.

    A escala é feita manualmente usando a VM auxiliar do Mecanismo do AKS que foi usada para implantar inicialmente o cluster do Kubernetes. Para obter mais informações, confira [Como escalar clusters do Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Cotas**: considere as [cotas](/azure-stack/operator/azure-stack-quota-types) que você configurou ao planejar uma implantação do AKS no Azure Stack Hub. Verifique se cada [assinatura](/azure-stack/operator/service-plan-offer-subscription-overview) tem as cotas e os planos adequados configurados. A assinatura precisará acomodar a quantidade de computação, armazenamento e outros serviços necessários para seus clusters à medida que eles escalarem horizontalmente.

- **Cargas de trabalho do aplicativo**: veja os [conceitos de clusters e cargas de trabalho](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) no documento Principais conceitos do Kubernetes para o Serviço de Kubernetes do Azure. Este artigo ajudará você a definir o escopo do tamanho adequado da VM com base nas necessidades de computação e memória do seu aplicativo.  

**Aplicativo (nível de aplicativo)**

Na camada de aplicativo, usamos o [HPA (Dimensionador Automático de Pod Horizontal)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) do Kubernetes. O HPA pode aumentar ou diminuir o número de réplicas (pod/Instâncias de Contêiner) em nossa implantação com base em diferentes métricas (como utilização da CPU).

Outra opção é escalar as instâncias de contêiner verticalmente. Faça isso alterando a quantidade de CPU e memória solicitada e disponível para uma implantação específica. Confira [Como gerenciar recursos para contêineres](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) em kubernetes.io para saber mais.

## <a name="networking-and-connectivity-considerations"></a>Considerações sobre rede e conectividade

A rede e a conectividade também afetam as três camadas mencionadas anteriormente para o Kubernetes no Azure Stack Hub. A seguinte tabela mostra as camadas e os serviços que elas contêm:

| Camada | Afeta | O quê? |
| --- | --- | ---
| Aplicativo | Aplicativo | Como o aplicativo fica acessível? Ele será exposto à Internet? |
| Cluster | Cluster do Kubernetes | API do Kubernetes, VM do Mecanismo do AKS, pull de imagens de contêiner (saída), envio de dados de monitoramento e telemetria (saída) |
| Infraestrutura | Azure Stack Hub | Acessibilidade dos pontos de extremidade de gerenciamento do Azure Stack Hub como os pontos de extremidade do Azure Resource Manager e do portal. |

**Aplicativo**

Para a camada de aplicativo, a consideração mais importante é se o aplicativo é exposto e acessível pela Internet. Da perspectiva do Kubernetes, a acessibilidade da Internet significa expor uma implantação ou um pod usando um Serviço de Kubernetes ou um controlador de entrada.

> [!NOTE]
> Recomendamos o uso de controladores de entrada para expor os Serviços de Kubernetes, pois o número de IPs públicos de front-end no Azure Stack Hub é limitado a cinco. Esse design também limita o número de Serviços de Kubernetes (com o tipo LoadBalancer) a 5, que será pequeno demais para muitas implantações. Acesse a [documentação do Mecanismo do AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) para saber mais.

Expor um aplicativo usando um IP público por meio de um balanceador de carga ou um controlador de entrada não significa necessariamente que o aplicativo agora é acessível pela Internet. É possível que o Azure Stack Hub tenha um endereço IP público que só esteja visível na intranet local, nem todos os IPs públicos são verdadeiramente voltados para a Internet.

O bloco anterior considera o tráfego de entrada para o aplicativo. Outro tópico que precisa ser considerado para uma implantação bem-sucedida do Kubernetes é o tráfego de saída. Estes são alguns casos de uso que exigem o tráfego de saída:

- Pull de imagens de contêiner armazenadas no Docker Hub ou no Registro de Contêiner do Azure
- Recuperação de gráficos do Helm
- Emissão de dados do Application Insights (ou outros dados de monitoramento)

Alguns ambientes corporativos podem exigir o uso de servidores proxy _transparentes_ ou _não transparentes_. Esses servidores exigem uma configuração específica em vários componentes do nosso cluster. A documentação do Mecanismo do AKS contém vários detalhes sobre como acomodar os proxies de rede. Para obter mais detalhes, confira [Mecanismo do AKS e servidores proxy](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Por fim, o tráfego entre clusters precisa fluir entre as instâncias do Azure Stack Hub. A implantação de exemplo consiste em clusters individuais do Kubernetes em execução em instâncias individuais do Azure Stack Hub. O tráfego entre eles, como o tráfego de replicação entre dois bancos de dados, é o "tráfego externo". O tráfego externo precisa ser roteado por meio de um VPN site a site ou endereços IP públicos do Azure Stack Hub para conectar o Kubernetes em duas instâncias do Azure Stack Hub:

![tráfego entre clusters e dentro do cluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

O cluster do Kubernetes não precisa necessariamente ser acessível pela Internet. A parte relevante é a API do Kubernetes usada para operar um cluster, por exemplo, usando `kubectl`. O ponto de extremidade de API do Kubernetes precisa estar acessível a todos que operam o cluster ou que implantam aplicativos e serviços com base nele. Este tópico é abordado mais detalhadamente da perspectiva de DevOps na seção [Considerações sobre implantação (CI/CD)](#deployment-cicd-considerations) abaixo.

No nível do cluster, também há algumas considerações sobre o tráfego de saída:

- Atualizações de nó (para o Ubuntu)
- Dados de monitoramento (enviados para o Azure Log Analytics)
- Outros agentes que exigem o tráfego de saída (específico de cada ambiente do implantador)

Antes de implantar o cluster do Kubernetes usando o Mecanismo do AKS, planeje o design final da rede. Em vez de criar uma Rede Virtual dedicada, talvez seja mais eficiente implantar um cluster em uma rede existente. Por exemplo, você pode aproveitar uma conexão VPN site a site existente já configurada no seu ambiente do Azure Stack Hub.

**Infraestrutura**  

A infraestrutura refere-se ao acesso aos pontos de extremidade de gerenciamento do Azure Stack Hub. Os pontos de extremidade incluem os portais do locatário e do administrador e os pontos de extremidade do administrador e do locatário do Azure Resource Manager. Esses pontos de extremidade são necessários para operar o Azure Stack Hub e os serviços principais dele.

## <a name="data-and-storage-considerations"></a>Considerações sobre dados e armazenamento

Duas instâncias do nosso aplicativo serão implantadas em dois clusters individuais do Kubernetes, em duas instâncias do Azure Stack Hub. Esse design exigirá que você considere como replicar e sincronizar dados entre elas.

Com o Azure, temos a capacidade interna de replicar o armazenamento em várias regiões e zonas na nuvem. Atualmente, com o Azure Stack Hub, não há maneiras nativas de replicar o armazenamento em duas instâncias diferentes do Azure Stack Hub: elas formam duas nuvens independentes sem uma forma abrangente de gerenciá-las como um conjunto. O planejamento da resiliência dos aplicativos em execução no Azure Stack Hub força você a considerar essa independência no design e nas implantações do seu aplicativo.

Na maioria dos casos, a replicação de armazenamento não será necessária para um aplicativo resiliente e altamente disponível implantado no AKS. Porém, você deve considerar o armazenamento independente por instância do Azure Stack Hub no design do aplicativo. Se esse design for uma preocupação ou um obstáculo para implantar a solução no Azure Stack Hub, haverá soluções possíveis de parceiros da Microsoft que fornecem anexos de armazenamento. Os anexos de armazenamento fornecem uma solução de replicação de armazenamento entre vários Azure Stack Hubs e o Azure. Para obter mais informações, confira [Soluções de parceiros](#partner-solutions).

Em nossa arquitetura, estas camadas foram consideradas:

**Configuration**

A configuração inclui a configuração do Azure Stack Hub, do Mecanismo do AKS e do próprio cluster do Kubernetes. Ela deve ser automatizada o máximo possível e armazenada como uma infraestrutura como código em um sistema de controle de versão baseado no Git, como o Azure DevOps ou o GitHub. Essas configurações não podem ser sincronizadas com facilidade em várias implantações. Portanto, recomendamos armazenar e aplicar a configuração externamente e usando o pipeline do DevOps.

**Aplicativo**

O aplicativo deve ser armazenado em um repositório baseado no Git. Sempre que houver uma nova implantação, alterações no aplicativo ou uma recuperação de desastre, elas poderão ser implantadas com facilidade usando o Azure Pipelines.

**Dados**

Os dados são a consideração mais importante na maioria dos designs de aplicativos. Os dados do aplicativo precisam permanecer em sincronia entre as diferentes instâncias do aplicativo. Os dados também precisam ter uma estratégia de backup e recuperação de desastre em caso de interrupção.

Alcançar esse design depende muito das opções de tecnologia. Estes são alguns exemplos de soluções para implementar um banco de dados de modo altamente disponível no Azure Stack Hub:

- [Implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

As considerações a serem feitas ao trabalhar com os dados em várias localizações são ainda mais complexas para uma solução altamente disponível e resiliente. Considere:

- Latência e conectividade de rede entre Azure Stack Hubs.
- Disponibilidade de identidades para serviços e permissões. Cada instância do Azure Stack Hub se integra a um diretório externo. Durante a implantação, escolha se deseja usar o Azure AD (Azure Active Directory) ou os ADFS (Serviços de Federação do Active Directory). Assim, é possível usar uma só identidade que pode interagir com várias instâncias independentes do Azure Stack Hub.

## <a name="business-continuity-and-disaster-recovery"></a>Continuidade dos negócios e recuperação de desastres

A BCDR (continuidade dos negócios e recuperação de desastres) é um tópico importante no Azure Stack Hub e no Azure. A principal diferença é que, no Azure Stack Hub, o operador precisa gerenciar todo o processo de BCDR. No Azure, algumas partes da BCDR são gerenciadas automaticamente pela Microsoft.

A BCDR afeta as mesmas áreas mencionadas na seção anterior [Considerações sobre dados e armazenamento](#data-and-storage-considerations):

- Infraestrutura/configuração
- Disponibilidade do aplicativo
- Dados de aplicativos

Além disso, conforme mencionado na seção anterior, essas áreas são de responsabilidade do operador do Azure Stack Hub e podem variar entre as organizações. Planeje a BCDR de acordo com as ferramentas e os processos disponíveis.

**Infraestrutura e configuração**

Esta seção aborda a infraestrutura física e lógica e a configuração do Azure Stack Hub. Ela aborda ações nos espaços do administrador e do locatário.

O operador (ou o administrador) do Azure Stack Hub é responsável pela manutenção das instâncias do Azure Stack Hub. Incluindo componentes como rede, armazenamento, identidade e outros tópicos que estão fora do escopo deste artigo. Para saber mais sobre as especificações das operações do Azure Stack Hub, confira os seguintes recursos:

- [Recuperar dados no Azure Stack Hub com o serviço Backup de Infraestrutura](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Habilitar o backup para o Azure Stack Hub pelo portal do administrador](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Recuperação da perda de dados catastrófica](/azure-stack/operator/azure-stack-backup-recover-data)
- [Práticas recomendadas do serviço de Backup de Infraestrutura](/azure-stack/operator/azure-stack-backup-best-practices)

O Azure Stack Hub é a plataforma e a malha em que os aplicativos do Kubernetes serão implantados. O proprietário do aplicativo do Kubernetes será um usuário do Azure Stack Hub, com acesso permitido para implantar a infraestrutura de aplicativo necessária para a solução. A infraestrutura do aplicativo, nesse caso, significa o cluster do Kubernetes, implantado por meio do Mecanismo do AKS, e os serviços relacionados. Esses componentes serão implantados no Azure Stack Hub, restringidos por uma oferta do Azure Stack Hub. Verifique se a oferta aceita pelo proprietário do aplicativo do Kubernetes tem capacidade suficiente (expressa em cotas do Azure Stack Hub) para implantar a solução inteira. Conforme recomendado na seção anterior, a implantação do aplicativo deve ser automatizada por meio de pipelines de infraestrutura como código e de implantação, como o Azure Pipelines do Azure DevOps.

Para obter mais informações sobre as ofertas e as cotas do Azure Stack Hub, confira [Visão geral dos serviços, dos planos, das ofertas e das assinaturas do Azure Stack Hub](/azure-stack/operator/service-plan-offer-subscription-overview)

É importante salvar e armazenar com segurança a configuração do Mecanismo do AKS, incluindo as respectivas saídas. Esses arquivos contêm informações confidenciais usadas para acessar o cluster do Kubernetes. Portanto, elas precisam ser protegidas contra exposição a não administradores.

**Disponibilidade do aplicativo**

O aplicativo não deve depender de backups de uma instância implantada. Como uma prática padrão, reimplante o aplicativo por completo seguindo os padrões da infraestrutura como código. Por exemplo, reimplante-o usando o Azure Pipelines do Azure DevOps. O procedimento da BCDR deve envolver a reimplantação do aplicativo no mesmo ou em outro cluster do Kubernetes.

**Dados do aplicativo**

Os dados do aplicativo são a parte crítica necessária para minimizar a perda de dados. Na seção anterior, foram descritas as técnicas para replicar e sincronizar dados entre duas (ou mais) instâncias do aplicativo. Dependendo da infraestrutura de banco de dados (MySQL, MongoDB, MSSQL ou outras) usada para armazenar os dados, haverá técnicas diferentes de backup e disponibilidade de banco de dados disponíveis para sua escolha.

As maneiras recomendadas de alcançar a integridade são:
- Uma solução de backup nativa para o banco de dados específico.
- Uma solução de backup que oficialmente dê suporte ao backup e à recuperação do tipo de banco de dados usado pelo aplicativo.

> [!IMPORTANT]
> Não armazene seus dados de backup na mesma instância do Azure Stack Hub em que residem os dados do aplicativo. Uma interrupção completa da instância do Azure Stack Hub também comprometerá os backups.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

O Kubernetes no Azure Stack Hub implantado por meio do Mecanismo do AKS não é um serviço gerenciado. É uma implantação e configuração automatizadas de um cluster do Kubernetes usando a IaaS (infraestrutura como serviço) do Azure. Assim, ele fornece a mesma disponibilidade da infraestrutura subjacente.

A infraestrutura do Azure Stack Hub já é resiliente a falhas e oferece funcionalidades como conjuntos de disponibilidade para distribuir componentes entre vários [domínios de falha e atualização](/azure-stack/user/azure-stack-vm-considerations#high-availability). Porém, a tecnologia subjacente (clustering de failover) ainda gera algum tempo de inatividade para as VMs em um servidor físico afetado, em caso de falha de hardware.

É uma boa prática implantar o cluster do Kubernetes de produção, bem como a carga de trabalho, em dois (ou mais) clusters. Esses clusters devem ser hospedados diferentes localizações ou datacenters e usar tecnologias como o Gerenciador de Tráfego do Azure para encaminhar os usuários com base no tempo de resposta do cluster ou na geografia.

![Como usar o Gerenciador de Tráfego para controlar fluxos de tráfego](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Os clientes que têm um só cluster do Kubernetes normalmente se conectam ao nome DNS ou ao IP de serviço de determinado aplicativo. Em uma implantação de vários clusters, os clientes devem se conectar a um nome DNS do Gerenciador de Tráfego que aponte para os serviços/o ingress em cada cluster do Kubernetes.

![Como usar o Gerenciador de Tráfego para encaminhamento para o cluster local](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Esse padrão também é uma [melhor prática para clusters (gerenciados) do AKS no Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

O próprio cluster do Kubernetes, implantado por meio do Mecanismo do AKS, deve consistir em, pelo menos, três nós mestres e dois nós de trabalho.

## <a name="identity-and-security-considerations"></a>Considerações sobre identidade e segurança

Identidade e segurança são tópicos importantes. Especialmente quando a solução abrange instâncias independentes do Azure Stack Hub. O Kubernetes e o Azure (incluindo o Azure Stack Hub) têm mecanismos distintos para o RBAC (controle de acesso baseado em função):

- O RBAC do Azure controla o acesso aos recursos no Azure (e no Azure Stack Hub), incluindo a capacidade de criar recursos do Azure. As permissões podem ser atribuídas a usuários, grupos ou entidades de serviço. (Uma entidade de serviço é uma identidade de segurança usada pelos aplicativos.)
- O RBAC do Kubernetes controla permissões para a API do Kubernetes. Por exemplo, a criação e a listagem de pods são ações que podem ser autorizadas (ou negadas) a um usuário por meio de RBAC. Para atribuir permissões de Kubernetes aos usuários, crie funções e associações de função.

**Identidade e RBAC do Azure Stack Hub**

O Azure Stack Hub fornece duas opções de provedor de identidade. O provedor usado depende do ambiente e se ele está em execução em um ambiente conectado ou desconectado:

- Azure AD: só pode ser usado em um ambiente conectado.
- ADFS para uma floresta tradicional do Active Directory: pode ser usado em um ambiente conectado ou desconectado.

O provedor de identidade gerencia usuários e grupos, incluindo autenticação e autorização para acessar os recursos. O acesso pode ser permitido aos recursos do Azure Stack Hub, como assinaturas, grupos de recursos e recursos individuais, como VMs ou balanceadores de carga. Para ter um modelo de acesso consistente, você deve considerar o uso dos mesmos grupos (diretos ou aninhados) para todos os Azure Stack Hubs. Veja um exemplo de configuração:

![grupos do AAD aninhados com o Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

O exemplo contém um grupo dedicado (usando o AAD ou o ADFS) para uma finalidade específica. Por exemplo, para fornecer permissões de Colaborador ao grupo de recursos que contém nossa infraestrutura de cluster do Kubernetes em uma instância específica do Azure Stack Hub (aqui, "Colaborador do Cluster do K8s Seattle"). Depois, esses grupos são aninhados em um grupo geral que contém os "subgrupos" de cada Azure Stack Hub.

Agora, nosso usuário de exemplo terá permissões de "Colaborador" nos grupos de recursos que contêm todo o conjunto de recursos da infraestrutura do Kubernetes. O usuário terá acesso aos recursos nas duas instâncias do Azure Stack Hub, porque elas compartilham o provedor de identidade.

> [!IMPORTANT]
> Essas permissões afetam apenas o Azure Stack Hub e alguns dos recursos implantados nele. Um usuário que tenha esse nível de acesso pode causar muitos danos, mas não pode acessar as VMs IaaS do Kubernetes nem a API do Kubernetes sem ter o acesso adicional à implantação do Kubernetes.

**Identidade e RBAC do Kubernetes**

Por padrão, um cluster do Kubernetes não usa o mesmo provedor de identidade do Azure Stack Hub subjacente. As VMs que hospedam o cluster do Kubernetes, o mestre e os nós de trabalho usam a chave SSH especificada durante a implantação do cluster. Essa chave SSH é necessária para se conectar a esses nós usando o SSH.

A API do Kubernetes (por exemplo, acessada por meio do `kubectl`) também é protegida por contas de serviço, incluindo uma conta de serviço padrão do "administrador do cluster". As credenciais dessa conta de serviço são inicialmente armazenadas no arquivo `.kube/config` nos nós mestres do Kubernetes.

**Credenciais de aplicativo e gerenciamento de segredos**

Para armazenar segredos como cadeias de conexão ou credenciais de banco de dados, há várias opções, incluindo:

- Cofre de Chave do Azure
- Segredos do Kubernetes
- Soluções de terceiros como o HashiCorp Vault (em execução no Kubernetes)

Não armazene segredos nem credenciais em um texto não criptografado nos arquivos de configuração, no código do aplicativo ou em scripts. Não os armazene também em um sistema de controle de versão. Em vez disso, a automação da implantação deve recuperar os segredos conforme necessário.

## <a name="patch-and-update"></a>Patch e atualização

O processo de **PNU (Patch e Atualização)** do Serviço de Kubernetes do Azure é parcialmente automatizado. As atualizações de versão do Kubernetes são disparadas manualmente, enquanto as atualizações de segurança são aplicadas automaticamente. Essas atualizações podem incluir correções de segurança do sistema operacional ou atualizações de kernel. O AKS não reinicializa automaticamente esses nós do Linux para concluir o processo de atualização. 

O processo de PNU para um cluster do Kubernetes implantado por meio do Mecanismo do AKS no Azure Stack Hub não é gerenciado e é responsabilidade do operador de cluster. 

O Mecanismo do AKS ajuda com as duas tarefas mais importantes:

- [Atualização para uma versão mais recente da imagem base do sistema operacional e do Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Atualização somente da imagem base do sistema operacional](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

As imagens base mais recentes do sistema operacional contêm as últimas correções de segurança do sistema operacional e atualizações de kernel. 

O mecanismo de [atualização autônoma](https://wiki.debian.org/UnattendedUpgrades) instala automaticamente as atualizações de segurança que são lançadas antes de uma nova versão de imagem base do sistema operacional estar disponível no Marketplace do Azure Stack Hub. A atualização autônoma está habilitada por padrão e instala as atualizações de segurança automaticamente, mas não reinicializa os nós de cluster do Kubernetes. A reinicialização dos nós pode ser automatizada por meio do [**kured (K** Ubernetes **RE** boot **D** aemon)](/azure/aks/node-updates-kured) de software livre. O kured inspeciona os nós do Linux que exigem uma reinicialização e cuida automaticamente do reagendamento dos pods em execução e do processo de reinicialização dos nós.

## <a name="deployment-cicd-considerations"></a>Considerações de implantação (CI/CD)

O Azure e o Azure Stack Hub expõem as mesmas APIs REST do Azure Resource Manager. Essas APIs são tratadas como qualquer outra nuvem do Azure (Azure, Azure China 21Vianet e Azure Government). Pode haver diferenças nas versões de API entre as nuvens e o Azure Stack Hub fornece apenas um subconjunto de serviços. O URI do ponto de extremidade de gerenciamento também é diferente para cada nuvem e cada instância do Azure Stack Hub.

Além das diferenças sutis mencionadas, as APIs REST do Azure Resource Manager oferecem uma forma consistente de interagir com o Azure e o Azure Stack Hub. O mesmo conjunto de ferramentas pode ser usado aqui como é usado com qualquer outra nuvem do Azure. Você pode usar o Azure DevOps, ferramentas como o Jenkins ou o PowerShell para implantar e orquestrar serviços no Azure Stack Hub.

**Considerações**

Uma das principais diferenças quando se trata das implantações do Azure Stack Hub é a questão da acessibilidade da Internet. A acessibilidade da Internet determina se um agente de build auto-hospedado ou hospedado pela Microsoft deve ser escolhido para os trabalhos de CI/CD.

Um agente auto-hospedado pode ser executado no Azure Stack Hub (como uma VM IaaS) ou em uma sub-rede da rede que possa acessar o Azure Stack Hub. Acesse [Agentes do Azure Pipelines](/azure/devops/pipelines/agents/agents) para saber mais sobre as diferenças.

A seguinte imagem ajuda você a decidir se precisará de um agente de build auto-hospedado ou hospedado pela Microsoft:

![Agentes de build auto-hospedados Sim ou não](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Os pontos de extremidade de gerenciamento do Azure Stack Hub são acessíveis pela Internet?
  - Sim: Podemos usar o Azure Pipelines com agentes hospedados pela Microsoft para se conectar ao Azure Stack Hub.
  - Não: Precisamos de agentes auto-hospedados que possam se conectar aos pontos de extremidade de gerenciamento do Azure Stack Hub.
- Nosso cluster do Kubernetes é acessível pela Internet?
  - Sim: Podemos usar o Azure Pipelines com agentes hospedados pela Microsoft para interagir diretamente com o ponto de extremidade de API do Kubernetes.
  - Não: Precisamos de agentes auto-hospedados que possam se conectar ao ponto de extremidade da API do cluster do Kubernetes.

Nos cenários em que os pontos de extremidade de gerenciamento do Azure Stack Hub e a API do Kubernetes são acessíveis pela Internet, a implantação pode usar um agente hospedado pela Microsoft. Essa implantação resultará em uma arquitetura de aplicativo parecida com esta:

[![Visão geral da arquitetura pública](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Se os pontos de extremidade do Azure Resource Manager, a API do Kubernetes ou ambos não estiverem diretamente acessíveis pela Internet, poderemos aproveitar um agente de build auto-hospedado para executar as etapas do pipeline. Esse design precisa de menos conectividade e pode ser implantado só com a conectividade de rede local com os pontos de extremidade do Azure Resource Manager e a API do Kubernetes:

[![Visão geral da arquitetura local](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **E quanto aos cenários desconectados?** Nos cenários em que o Azure Stack Hub, o Kubernetes ou ambos não têm pontos de extremidade de gerenciamento para a Internet, ainda é possível usar o Azure DevOps para as implantações. Use um pool de agentes auto-hospedados (um agente de DevOps em execução no local ou no próprio Azure Stack Hub) ou um Azure DevOps Server totalmente auto-hospedado no local. O agente auto-hospedado precisa apenas de conectividade HTTPS (TCP/443) de saída com a Internet.

O padrão pode usar um cluster do Kubernetes (implantado e orquestrado com o Mecanismo do AKS) em cada instância do Azure Stack Hub. Ele inclui um aplicativo que consiste em serviços de front-end, de camada intermediária e de back-end (por exemplo, o MongoDB) e um controlador de entrada baseado no NGINX. Em vez de usar um banco de dados hospedado no cluster do K8s, você pode aproveitar "armazenamentos de dados externos". As opções de banco de dados incluem o MySQL, o SQL Server ou qualquer tipo de banco de dados hospedado fora do Azure Stack Hub ou na IaaS. Configurações como essa não estão no escopo deste artigo.

## <a name="partner-solutions"></a>Soluções de parceiros

Há soluções de Parceiros da Microsoft que podem estender as funcionalidades do Azure Stack Hub. Essas soluções foram consideradas úteis nas implantações de aplicativos em execução nos clusters do Kubernetes.  

## <a name="storage-and-data-solutions"></a>Soluções de armazenamento e dados

Conforme descrito em [Considerações sobre dados e armazenamento](#data-and-storage-considerations), atualmente, o Azure Stack Hub não tem uma solução nativa para replicar o armazenamento em várias instâncias. Ao contrário do Azure, a capacidade de replicar o armazenamento em várias regiões não existe. No Azure Stack Hub, cada instância é a própria nuvem distinta. No entanto, as soluções estão disponíveis por meio de Parceiros da Microsoft que habilitam a replicação de armazenamento entre os Azure Stack Hubs e o Azure. 

**SCALITY**

O [Scality](https://www.scality.com/) fornece armazenamento em escala da Web que capacita empresas digitais desde 2009. O Scality RING, nosso armazenamento definido pelo software, transforma os servidores x86 de mercadoria em um pool de armazenamento ilimitado para qualquer tipo de dados (arquivo e objeto) em escala de petabytes.

**CLOUDIAN**

O [Cloudian](https://www.cloudian.com/) simplifica o armazenamento corporativo com armazenamento escalonável ilimitado que consolida conjuntos de dados em massa em um só ambiente facilmente gerenciado.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os conceitos apresentados neste artigo:

- [Escala entre nuvens](pattern-cross-cloud-scale.md) e [padrões de aplicativo distribuídos geograficamente](pattern-geo-distributed.md) no Azure Stack Hub.
- [Arquitetura de microsserviços no AKS (Serviço de Kubernetes do Azure)](/azure/architecture/reference-architectures/microservices/aks).

Quando estiver pronto para testar o exemplo de solução, prossiga com o [Guia de implantação de cluster de alta disponibilidade do Kubernetes](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes.