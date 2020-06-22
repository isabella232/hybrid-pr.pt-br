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
# <a name="cross-cloud-scaling-pattern"></a>Padrão de dimensionamento entre nuvens

Adicione recursos automaticamente a um aplicativo existente para acomodar um aumento na carga.

## <a name="context-and-problem"></a>Contexto e problema

Seu aplicativo não pode aumentar a capacidade de atender a aumentos inesperados na demanda. Essa falta de escalabilidade faz com que os usuários não atinjam o aplicativo durante os horários de pico de uso. O aplicativo pode atender a um número fixo de usuários.

As empresas globais exigem aplicativos baseados em nuvem seguros, confiáveis e disponíveis. A reunião aumenta na demanda e o uso da infra-estrutura certa para dar suporte a essa demanda é essencial. As empresas lutam para balancear custos e manutenção com segurança de dados de negócios, armazenamento e disponibilidade em tempo real.

Talvez você não consiga executar seu aplicativo na nuvem pública. No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda para o aplicativo. Com esse padrão, você pode usar a elasticidade da nuvem pública com sua solução local.

## <a name="solution"></a>Solução

O padrão de dimensionamento entre nuvens estende um aplicativo localizado em uma nuvem local com recursos de nuvem pública. O padrão é disparado por um aumento ou diminuição na demanda e, respectivamente, adiciona ou remove recursos na nuvem. Esses recursos fornecem redundância, disponibilidade rápida e roteamento de conformidade geográfica.

![Padrão de dimensionamento entre nuvens](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Esse padrão aplica-se somente a componentes sem estado do seu aplicativo.

## <a name="components"></a>Componentes

O padrão de dimensionamento entre nuvens consiste nos seguintes componentes.

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gerenciador de Tráfego

No diagrama, isso está localizado fora do grupo de nuvem pública, mas ele precisaria coordenar o tráfego no datacenter local e na nuvem pública. O balanceador fornece alta disponibilidade para o aplicativo monitorando pontos de extremidade e fornecendo a redistribuição de failover quando necessário.

#### <a name="domain-name-system-dns"></a>DNS (Sistema de Nomes de Domínio)

O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.

### <a name="cloud"></a>Nuvem

#### <a name="hosted-build-server"></a>Servidor de compilação hospedado

Um ambiente para hospedar seu pipeline de compilação.

#### <a name="app-resources"></a>Recursos do aplicativo

Os recursos do aplicativo precisam ser capazes de reduzir horizontalmente e escalar horizontalmente, como conjuntos de dimensionamento de máquinas virtuais e contêineres.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Use um nome de domínio personalizado para solicitações de roteamento glob.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.  

### <a name="local-cloud"></a>Nuvem local

#### <a name="hosted-build-server"></a>Servidor de compilação hospedado

Um ambiente para hospedar seu pipeline de compilação.

#### <a name="app-resources"></a>Recursos do aplicativo

Os recursos do aplicativo precisam da capacidade de reduzir e escalar horizontalmente, como conjuntos de dimensionamento de máquinas virtuais e contêineres.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Use um nome de domínio personalizado para solicitações de roteamento glob.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

O principal componente do dimensionamento entre nuvem é a capacidade de fornecer dimensionamento sob demanda. O dimensionamento deve ocorrer entre a infraestrutura de nuvem pública e a local e fornecer um serviço consistente e confiável por demanda.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

O padrão de nuvem cruzada garante o gerenciamento contínuo e a interface familiar entre ambientes.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use este padrão:

- Quando você precisa aumentar a capacidade do aplicativo com demandas inesperadas ou demandas periódicas por demanda.
- Quando você não quiser investir em recursos que serão usados somente durante picos. Pague pelo que usar.

Esse padrão não é recomendado quando:

- Sua solução requer que os usuários se conectem pela Internet.
- Sua empresa tem regulamentos locais que exigem que a conexão de origem venha de uma chamada no local.
- Sua rede experimenta afunilamentos regulares que restringem o desempenho do dimensionamento.
- Seu ambiente está desconectado da Internet e não consegue acessar a nuvem pública.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte a [visão geral do Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como esse balanceador de carga de tráfego baseado em DNS funciona.
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de dimensionamento de nuvem cruzada](solution-deployment-guide-cross-cloud-scaling.md). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes. Você aprende a criar uma solução de nuvem cruzada para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado pelo Hub Azure Stack para um aplicativo Web hospedado do Azure. Você também aprende a usar o dimensionamento automático por meio do Traffic Manager, garantindo um utilitário de nuvem flexível e escalonável quando sob carga.
