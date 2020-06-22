---
title: Padrão de dimensionamento entre nuvens (dados locais) no Hub Azure Stack
description: Saiba como criar um aplicativo de nuvem cruzada escalonável que usa dados locais no Azure e no Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909820"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Padrão de dimensionamento entre nuvens (dados locais)

Saiba como criar um aplicativo híbrido que abranja o Azure e o Hub de Azure Stack. Esse padrão também mostra como usar uma única fonte de dados local para fins de conformidade.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações coletam e armazenam grandes quantidades de dados confidenciais do cliente. Frequentemente, eles são impedidos de armazenar dados confidenciais na nuvem pública devido às normas corporativas ou à política governamental. Essas organizações também querem aproveitar a escalabilidade da nuvem pública. A nuvem pública pode lidar com picos sazonais de tráfego, permitindo que os clientes paguem exatamente o hardware de que precisam, quando precisam.

## <a name="solution"></a>Solução

A solução aproveita os benefícios de conformidade da nuvem privada, combinando-os com a escalabilidade da nuvem pública. A nuvem híbrida do Hub do Azure e do Azure Stack fornece uma experiência consistente para os desenvolvedores. Essa consistência permite aplicar suas habilidades a ambientes de nuvem pública e locais.

O guia de implantação da solução permite que você implante um aplicativo Web idêntico a uma nuvem pública e privada. Você também pode acessar uma rede roteável que não seja da Internet hospedada na nuvem privada. Os aplicativos Web são monitorados para carregamento. Após um aumento significativo no tráfego, um programa manipula os registros DNS para redirecionar o tráfego para a nuvem pública. Quando o tráfego não é mais significativo, os registros DNS são atualizados para direcionar o tráfego de volta para a nuvem privada.

[![Dimensionamento entre nuvem com padrão de dados local](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Componentes

Essa solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Azure | Serviço de aplicativo do Azure | [Azure app serviço](/azure/app-service/) permite que você crie e hospede aplicativos Web, aplicativos de API RESTful e Azure functions. Tudo na linguagem de programação de sua escolha, sem gerenciamento de infraestrutura. |
| | Rede Virtual do Azure| A [VNet (rede virtual) do Azure](/azure/virtual-network/virtual-networks-overview) é o bloco de construção fundamental para redes privadas no Azure. A VNet permite que vários tipos de recursos do Azure, como máquinas virtuais (VM), comuniquem-se com segurança entre si, Internet e redes locais. A solução também demonstra o uso de componentes de rede adicionais:<br>-sub-redes de gateway e de aplicativo.<br>-um gateway de rede local.<br>-um gateway de rede virtual, que atua como uma conexão de gateway de VPN site a site.<br>-um endereço IP público.<br>-uma conexão VPN ponto a site.<br>-DNS do Azure para hospedar domínios DNS e fornecer resolução de nomes. |
| | Gerenciador de Tráfego do Azure | O [Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) é um balanceador de carga de tráfego baseado em DNS. Ele permite que você controle a distribuição do tráfego do usuário para pontos de extremidade de serviço em diferentes data centers. |
| | Azure Application Insights | O [Application insights](/azure/azure-monitor/app/app-insights-overview) é um serviço de gerenciamento de desempenho de aplicativos extensível para desenvolvedores da Web que criam e gerenciam aplicativos em várias plataformas.|
| | Funções do Azure | O [Azure Functions](/azure/azure-functions/) permite que você execute seu código em um ambiente sem servidor que não precise primeiro criar uma VM ou publicar um aplicativo Web. |
| | Dimensionamento automático do Azure | O [dimensionamento automático](/azure/azure-monitor/platform/autoscale-overview) é um recurso interno de serviços de nuvem, VMS e aplicativos Web. O recurso permite que os aplicativos realizem seu melhor desempenho quando mudam de demanda. Os aplicativos serão ajustados para picos de tráfego, notificando você quando as métricas mudam e dimensionando conforme necessário. |
| Hub de Azure Stack | Computação IaaS | Azure Stack Hub permite que você use o mesmo modelo de aplicativo, portal de autoatendimento e APIs habilitados pelo Azure. O Azure Stack Hub IaaS permite uma ampla gama de tecnologias de software livre para implantações de nuvem híbrida consistentes. O exemplo de solução usa uma VM do Windows Server para SQL Server, por exemplo.|
| | Serviço de aplicativo do Azure | Assim como o aplicativo Web do Azure, a solução usa [Azure app serviço no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) para hospedar o aplicativo Web. |
| | Rede | A rede virtual do hub de Azure Stack funciona exatamente como a rede virtual do Azure. Ele usa muitos dos mesmos componentes de rede, incluindo nomes de host personalizados.
| Azure DevOps Services | Inscrição | Configure rapidamente a integração contínua para compilação, teste e implantação. Para obter mais informações, consulte [inscrever-se, entrar no Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Use [Azure pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) para integração contínua/entrega contínua. Azure Pipelines permite que você gerencie a compilação hospedada e os agentes e as definições de versão. |
| | Repositório de códigos | Aproveite vários repositórios de código para simplificar seu pipeline de desenvolvimento. Use repositórios de código existentes no GitHub, bitbucket, Dropbox, OneDrive e Azure Repos. |

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

O Azure e o Hub de Azure Stack são exclusivamente adequados para dar suporte às necessidades do negócio globalmente distribuído de hoje.

#### <a name="hybrid-cloud-without-the-hassle"></a>Nuvem híbrida sem complicações

A Microsoft oferece uma integração inigualável de ativos locais com o Azure Stack Hub e o Azure em uma solução unificada. Essa integração elimina a complicação de gerenciar soluções de vários pontos e uma combinação de provedores de nuvem. Com o dimensionamento entre nuvens, o poder do Azure é apenas alguns cliques. Basta conectar seu hub de Azure Stack ao Azure com a intermitência de nuvem e seus dados e aplicativos estarão disponíveis no Azure quando necessário.

- Elimine a necessidade de criar e manter um site de DR secundário.
- Economize tempo e dinheiro eliminando o backup em fita e alojar até 99 anos de dados de backup no Azure.
- Migre facilmente a execução de cargas de trabalho Hyper-V, físicas (em versão prévia) e VMware (em versão prévia) para o Azure para aproveitar a economia e a elasticidade da nuvem.
- Execute relatórios ou análises com computação intensiva em uma cópia replicada do seu ativo local no Azure sem afetar as cargas de trabalho de produção.
- Estoure na nuvem e execute cargas de trabalho locais no Azure, com modelos de computação maiores quando necessário. O híbrido oferece o poder que você precisa, quando necessário.
- Crie ambientes de desenvolvimento de várias camadas no Azure com alguns cliques – até mesmo Replique dados de produção ao vivo para seu ambiente de desenvolvimento/teste para mantê-lo em sincronização quase em tempo real.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economia de dimensionamento entre nuvem com o Hub Azure Stack

A principal vantagem da nuvem de intermitência é a economia econômica. Você só paga pelos recursos adicionais quando há uma demanda por esses recursos. Não há mais gastos com capacidade extra desnecessária ou tentativa de prever picos e flutuações de demanda.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Reduzir cargas de alta demanda na nuvem

O dimensionamento entre nuvens pode ser usado para encargos de processamento. A carga é distribuída movendo aplicativos básicos para a nuvem pública, liberando recursos locais para aplicativos críticos para os negócios. Um aplicativo pode ser aplicado à nuvem privada e, em seguida, intermitente para a nuvem pública somente quando necessário para atender às demandas.

### <a name="availability"></a>Disponibilidade

A implantação global tem seus próprios desafios, como conectividade variável e diferentes regulamentos governamentais por região. Os desenvolvedores podem desenvolver apenas um aplicativo e, em seguida, implantá-lo por diferentes motivos com requisitos diferentes. Implante seu aplicativo na nuvem pública do Azure e implante outras instâncias ou componentes localmente. Você pode gerenciar o tráfego entre todas as instâncias usando o Azure.

### <a name="manageability"></a>Capacidade de gerenciamento

#### <a name="a-single-consistent-development-approach"></a>Uma abordagem de desenvolvimento única e consistente

O Azure e o Hub de Azure Stack permitem que você use um conjunto consistente de ferramentas de desenvolvimento em toda a organização. Essa consistência facilita a implementação de uma prática da integração contínua e do desenvolvimento contínuo (CI/CD). Muitos aplicativos e serviços implantados no Azure ou Azure Stack Hub são intercambiáveis e podem ser executados em qualquer local de forma direta.

Um pipeline de CI/CD híbrido pode ajudá-lo a:

- Inicie uma nova compilação com base em confirmações de código para seu repositório de código.
- Implante automaticamente seu código recém-criado no Azure para o teste de aceitação do usuário.
- Depois que o código passar no teste, implante-o automaticamente no Hub Azure Stack.

### <a name="a-single-consistent-identity-management-solution"></a>Uma solução de gerenciamento de identidade única e consistente

Azure Stack Hub funciona com o Azure Active Directory (AD do Azure) e o Serviços de Federação do Active Directory (AD FS) (ADFS). Azure Stack Hub funciona com o Azure AD em cenários conectados. Para ambientes que não têm conectividade, você pode usar o ADFS como uma solução desconectada. As entidades de serviço são usadas para conceder acesso a aplicativos, permitindo que eles implantem ou configurem recursos por meio de Azure Resource Manager.

### <a name="security"></a>Segurança

#### <a name="ensure-compliance-and-data-sovereignty"></a>Garantir a conformidade e a soberania de dados

Azure Stack Hub permite que você execute o mesmo serviço em vários países como faria usando uma nuvem pública. Implantar o mesmo aplicativo em data centers em cada país permite que os requisitos de soberania de dados sejam atendidos. Esse recurso garante que os dados pessoais sejam mantidos dentro das bordas de cada país.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack o Hub-postura de segurança

Não há nenhuma postura de segurança sem um processo de manutenção estável e contínuo. Por esse motivo, a Microsoft investiu em um mecanismo de orquestração que aplica patches e atualizações diretamente em toda a infraestrutura.

Graças às parcerias com parceiros OEM de Hub Azure Stack, a Microsoft estende a mesma postura de segurança para componentes específicos de OEM, como o host de ciclo de vida de hardware e o software em execução sobre ele. Essa parceria garante que Azure Stack Hub tenha uma postura de segurança sólida e uniforme em toda a infraestrutura. Por sua vez, os clientes podem criar e proteger suas cargas de trabalho de aplicativo.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Uso de entidades de serviço por meio do PowerShell, da CLI e do portal do Azure

Para fornecer acesso de recurso a um script ou aplicativo, configure uma identidade para seu aplicativo e autentique o aplicativo com suas próprias credenciais. Essa identidade é conhecida como uma entidade de serviço e permite que você:

- Atribua permissões à identidade do aplicativo que sejam diferentes das suas próprias permissões e sejam restritas precisamente às necessidades do aplicativo.
- Use um certificado para a autenticação ao executar um script autônomo.

Para obter mais informações sobre a criação da entidade de serviço e o uso de um certificado para credenciais, consulte [usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Minha organização está usando uma abordagem DevOps ou tem uma planejada para o futuro próximo.
- Quero implementar as práticas de CI/CD em minha implementação do hub de Azure Stack e na nuvem pública.
- Quero consolidar o pipeline de CI/CD em ambientes de nuvem e locais.
- Quero a capacidade de desenvolver aplicativos diretamente usando serviços de nuvem ou locais.
- Quero aproveitar habilidades de desenvolvedor consistentes em aplicativos locais e na nuvem.
- Estou usando o Azure, mas tenho desenvolvedores que trabalham em uma nuvem de Hub de Azure Stack local.
- Meus aplicativos locais apresentam picos de demanda durante flutuações sazonais, cíclicas ou imprevisíveis.
- Tenho componentes locais e quero usar a nuvem para dimensioná-los sem problemas.
- Quero a escalabilidade da nuvem, mas quero que meu aplicativo seja executado no local o máximo possível.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Assista aos [aplicativos de escala dinâmica entre data centers e nuvem pública](https://www.youtube.com/watch?v=2lw8zOpJTn0) para obter uma visão geral de como esse padrão é usado.
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e para responder a perguntas adicionais que você possa ter.
- Esse padrão usa a família de produtos Azure Stack, incluindo Azure Stack Hub. Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação de solução de dimensionamento de nuvem cruzada (dados locais)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.
