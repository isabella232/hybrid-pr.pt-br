---
title: Padrão de dimensionamento entre nuvens (dados locais) no Azure Stack Hub
description: Saiba como criar um aplicativo escalonável entre nuvens que usa dados locais no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281237"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Padrão de dimensionamento entre nuvens (dados locais)

Saiba como criar um aplicativo híbrido que abrange o Azure e o Azure Stack Hub. Este padrão também mostra como usar uma única fonte de dados local para fins de conformidade.

## <a name="context-and-problem"></a>Contexto e problema

Muitas empresas coletam e armazenam quantidades enormes de dados confidenciais dos clientes, mas são frequentemente impedidas de armazená-los na nuvem pública devido a regulamentos corporativos ou a políticas governamentais. No entanto, elas também querem aproveitar a escalabilidade fornecida por ela. Como a nuvem pública é capaz de lidar com picos sazonais de tráfego, os clientes pagam exatamente pelo hardware necessário, quando precisam dele.

## <a name="solution"></a>Solução

A solução combina os benefícios de conformidade da nuvem privada com a escalabilidade da nuvem pública. A nuvem híbrida do Azure e do Azure Stack Hub oferece uma experiência consistente aos desenvolvedores. Com isso, eles conseguem aplicar suas habilidades tanto a ambientes locais quanto de nuvem pública.

Com o guia de implantação da solução, você pode implantar um aplicativo Web idêntico em uma nuvem pública e em uma privada. Também é possível acessar uma rede roteável fora da Internet que esteja hospedada na nuvem privada. Os aplicativos Web são monitorados com relação à carga. Quando há um aumento significativo no tráfego, um programa manipula os registros DNS para redirecioná-lo para a nuvem pública. Quando ele diminui, os registros DNS são atualizados para direcioná-lo novamente para a nuvem privada.

[![Padrão de dimensionamento entre nuvens usando dados locais](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Componentes

Esta solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Azure | Serviço de aplicativo do Azure | O [Serviço de Aplicativo do Azure](/azure/app-service/) permite criar e hospedar aplicativos Web, aplicativos de API RESTful e o Azure Functions. Para isso, você escolhe sua linguagem de programação preferencial, sem precisar gerenciar a infraestrutura. |
| | Rede Virtual do Azure| A [VNet (Rede Virtual) do Azure](/azure/virtual-network/virtual-networks-overview) é o bloco de construção fundamental das redes privadas no Azure. Ela permite a comunicação segura de vários tipos de recursos do Azure, como as VMs (máquinas virtuais), tanto entre si quanto com a Internet e com as redes locais. A solução também demonstra o uso de componentes de rede adicionais:<br>– Sub-redes de aplicativo e gateway.<br>– Um gateway de rede local.<br>– Um gateway de rede virtual, que atua como uma conexão de gateway de VPN site a site.<br>– Um endereço IP público.<br>– Uma conexão VPN ponto a site.<br>– DNS do Azure para hospedar domínios DNS e fornecer resolução de nomes. |
| | Gerenciador de Tráfego do Azure | O [Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) é um balanceador de carga de tráfego baseado em DNS. Ele permite controlar a distribuição do tráfego do usuário para pontos de extremidade de serviço em diferentes datacenters. |
| | Azure Application Insights | O [Application Insights](/azure/azure-monitor/app/app-insights-overview) é um serviço extensível de gerenciamento de desempenho de aplicativo para desenvolvedores da Web que criam e gerenciam aplicativos em várias plataformas.|
| | Funções do Azure | O [Azure Functions](/azure/azure-functions/) permite executar seu código em um ambiente sem servidor, e você não precisa criar uma VM ou publicar um aplicativo Web primeiro. |
| | Dimensionamento automático do Azure | O [dimensionamento automático](/azure/azure-monitor/platform/autoscale-overview) é um recurso integrado dos Serviços de Nuvem, das VMs e dos aplicativos Web. Ele permite o melhor desempenho dos aplicativos quando há alterações na demanda. Os aplicativos são ajustados para picos de tráfego, você recebe uma notificação quando as métricas mudam e o dimensionamento deles ocorre conforme necessário. |
| Azure Stack Hub | Computação de IaaS | O Azure Stack Hub permite o uso do mesmo modelo de aplicativo, do mesmo portal de autoatendimento e das mesmas APIs que são usados com o Azure. A IaaS do Azure Stack Hub dá suporte a uma ampla variedade de tecnologias de código aberto para implantações consistentes de nuvem híbrida. A solução de exemplo usa uma VM Windows Server para SQL Server, por exemplo.|
| | Serviço de aplicativo do Azure | Assim como o aplicativo Web do Azure, a solução usa o [Serviço de Aplicativo do Azure no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) para hospedar o aplicativo Web. |
| | Rede | A Rede Virtual do Azure Stack Hub funciona exatamente como a Rede Virtual do Azure. Ela usa muitos dos mesmos componentes de rede, como nomes de host personalizados.
| Azure DevOps Services | Inscrição | Configure rapidamente a integração contínua para criação, testes e implantação. Para saber mais, veja [Inscrever-se e entrar no Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Use o [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) para integração/entrega contínua. Com ele, você gerencia definições e agentes hospedados de build e versão. |
| | Repositório de códigos | Utilize vários repositórios de código para simplificar o pipeline de desenvolvimento, como os repositórios de código existentes no GitHub, no Bitbucket, no Dropbox, no OneDrive e no Azure Repos. |

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

O Azure e o Azure Stack Hub são exclusivos e ideais para as necessidades do mundo atual de negócios distribuídos globalmente.

#### <a name="hybrid-cloud-without-the-hassle"></a>Nuvem híbrida sem complicações

A Microsoft oferece uma integração incomparável de ativos locais com o Azure Stack Hub e o Azure em uma solução unificada. Essa integração simplifica o gerenciamento de várias soluções de ponto e de vários provedores de nuvem. Com o dimensionamento entre nuvens, você tem todo o poder do Azure em suas mãos. Basta conectar seu Azure Stack Hub ao Azure com o bursting de nuvem e seus dados e aplicativos estarão disponíveis no Azure sempre que você precisar deles.

- Você não precisa mais criar e manter um site secundário de DR.
- Economize tempo e dinheiro eliminando o backup em fita e hospede até 99 anos de dados de backup no Azure.
- Migre facilmente as cargas de trabalho físicas (em versão prévia), do Hyper-V e do VMware (em versão prévia) para o Azure e aproveite a economia e a elasticidade da nuvem.
- Execute relatórios ou análises de computação intensiva em uma cópia replicada do seu ativo local no Azure sem afetar as cargas de trabalho de produção.
- Faça o bursting de nuvem e execute as cargas de trabalho locais no Azure com modelos de computação maiores, quando for preciso. A nuvem híbrida pode fornecer o poder necessário sempre que você precisar dele.
- Crie ambientes de desenvolvimento de várias camadas no Azure com apenas alguns cliques. Você pode até mesmo replicar dados reais de produção no ambiente de desenvolvimento/teste para manter a sincronização deles quase em tempo real.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economia do dimensionamento entre nuvens com o Azure Stack Hub

A principal vantagem do bursting de nuvem é a economia. Você só paga pelos recursos adicionais quando há uma demanda por eles. Agora você não precisa mais gastar com uma capacidade adicional desnecessária ou tentar prever os picos e as flutuações de demanda.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Reduzir cargas de alta demanda na nuvem

Use o dimensionamento entre nuvens para lidar com as cargas de processamento. Com a distribuição dessas cargas, os aplicativos básicos são movidos para a nuvem pública e os recursos locais são liberados para os aplicativos críticos aos negócios. Um aplicativo designado à nuvem privada pode ser enviado por bursting para a nuvem pública somente quando necessário para atender às demandas.

### <a name="availability"></a>Disponibilidade

A implantação global apresenta desafios, como conectividade variável e regulamentos governamentais diferentes de acordo com a região. Os desenvolvedores podem criar um único aplicativo e implantá-lo em regiões variadas com diferentes requisitos. Implante seu aplicativo na nuvem pública do Azure e, em seguida, faça a implantação de instâncias ou componentes adicionais no local. Você pode gerenciar o tráfego entre todas as instâncias com o Azure.

### <a name="manageability"></a>Capacidade de gerenciamento

#### <a name="a-single-consistent-development-approach"></a>Uma abordagem de desenvolvimento única e consistente

O Azure e o Azure Stack Hub permitem o uso de um conjunto consistente de ferramentas de desenvolvimento em toda a empresa. Com isso, é mais fácil implementar uma prática de CI/CD (integração contínua e desenvolvimento contínuo). Muitos aplicativos e serviços implantados no Azure ou no Azure Stack Hub são intercambiáveis e podem ser executados em qualquer local sem problemas.

Um pipeline híbrido de CI/CD pode ajudar você a:

- Iniciar uma nova criação com base em confirmações de código em seu repositório de código.
- Implantar automaticamente um código recém-criado no Azure para o teste de aceitação do usuário.
- Fazer a implantação automática no Azure Stack Hub depois que o código passa no teste.

### <a name="a-single-consistent-identity-management-solution"></a>Uma solução de gerenciamento de identidades única e consistente

O Azure Stack Hub funciona com o Azure AD (Azure Active Directory) e os Serviços de Federação do Active Directory (AD FS). Com o Azure AD, ele funciona em cenários conectados. Com os AD FS como uma solução desconectada, ele funciona para ambientes sem conectividade. As entidades de serviço são usadas para permitir acesso a aplicativos que implantarão ou configurarão recursos por meio do Azure Resource Manager.

### <a name="security"></a>Segurança

#### <a name="ensure-compliance-and-data-sovereignty"></a>Garantir a conformidade e a soberania de dados

Com o Azure Stack Hub, você executa o mesmo serviço em vários países, como se estivesse usando uma nuvem pública. Ao implantar o mesmo aplicativo nos datacenters de cada país, você atende aos requisitos de soberania de dados. Essa funcionalidade garante que os dados pessoais permaneçam dentro das fronteiras de cada país.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub – Postura de segurança

Toda postura de segurança precisa de um processo sólido de manutenção contínua. Por isso, a Microsoft investiu em um mecanismo de orquestração que aplica patches e atualizações continuamente em toda a infraestrutura.

Graças aos parceiros OEM do Azure Stack Hub, a Microsoft estende a mesma postura de segurança para os componentes específicos de OEM, como o host de ciclo de vida de hardware e o software em execução nele. Essa parceria garante ao Azure Stack Hub uma postura de segurança uniforme e sólida em toda a infraestrutura. Como resultado, os clientes podem criar e proteger cargas de trabalho de aplicativo.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Uso de entidades de serviço por meio do PowerShell, da CLI e do portal do Azure

Para conceder a um script ou aplicativo o acesso a recursos, configure uma identidade para ele e o autentique com as próprias credenciais. Essa identidade, conhecida como entidade de serviço, permite:

- Atribuir à identidade do aplicativo permissões diferentes de suas próprias e restritas precisamente às necessidades dele.
- Use um certificado para a autenticação ao executar um script autônomo.

Para saber mais sobre a criação da entidade de serviço e o uso de um certificado para credenciais, confira [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Minha empresa está usando uma abordagem de DevOps ou planeja adotar uma em breve.
- Quero adotar práticas de CI/CD em minha implementação do Azure Stack Hub e na nuvem pública.
- Quero consolidar o pipeline de CI/CD em ambientes locais e de nuvem.
- Quero desenvolver aplicativos com facilidade usando serviços locais ou em nuvem.
- Quero aproveitar as habilidades consistentes do desenvolvedor em aplicativos locais e na nuvem.
- Estou usando o Azure, mas tenho desenvolvedores que estão trabalhando em uma nuvem local do Azure Stack Hub.
- Meus aplicativos locais passam por picos de demanda durante flutuações sazonais, cíclicas ou imprevisíveis.
- Tenho componentes locais e quero usar a nuvem para dimensioná-los com facilidade.
- Quero aproveitar a escalabilidade da nuvem, mas prefiro que meu aplicativo seja executado no local o máximo possível.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Para ter uma visão geral sobre o uso desse padrão, assista a [Dimensionar aplicativos de maneira dinâmica entre datacenters e a nuvem pública](https://www.youtube.com/watch?v=2lw8zOpJTn0).
- Confira as [considerações sobre o design de aplicativos híbridos](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas a outras perguntas.
- Esse padrão usa a família de produtos Azure Stack, incluindo o Azure Stack Hub. Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar a solução de exemplo, siga o [guia de implantação da solução de dimensionamento entre nuvens (dados locais)](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes.