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
# <a name="geo-distributed-app-pattern"></a>Padrão de aplicativo distribuído geograficamente

Saiba como fornecer pontos de extremidade de aplicativo em várias regiões e rotear o tráfego do usuário com base nas necessidades de local e conformidade.

## <a name="context-and-problem"></a>Contexto e problema

As organizações com geografias de longo alcance se esforçam para distribuir e proteger com segurança e precisão o acesso aos dados e, ao mesmo tempo, garantem os níveis necessários de segurança, conformidade e desempenho por usuário, local e dispositivo entre bordas.

## <a name="solution"></a>Solução

O padrão de roteamento de tráfego geográfico do hub de Azure Stack ou aplicativos distribuídos geograficamente permite que o tráfego seja direcionado para pontos de extremidade específicos com base em várias métricas. Criar um Gerenciador de tráfego com roteamento baseado em geográfico e configuração de ponto de extremidade roteia o tráfego para os pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e necessidades de dados.

![Padrão distribuído geograficamente](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Componentes

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gerenciador de Tráfego

No diagrama, o Gerenciador de tráfego está localizado fora da nuvem pública, mas ele precisa ser capaz de coordenar o tráfego no datacenter local e na nuvem pública. O balanceador roteia o tráfego para localizações geográficas.

#### <a name="domain-name-system-dns"></a>Sistema de nome de domínio (DNS)

O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.

### <a name="public-cloud"></a>Nuvem pública

#### <a name="cloud-endpoint"></a>Ponto de extremidade de nuvem

Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.  

### <a name="local-clouds"></a>Nuvens locais

#### <a name="local-endpoint"></a>Ponto de extremidade local

Os endereços IP públicos são usados para rotear o tráfego de entrada por meio do Gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo de nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

O padrão manipula o roteamento de tráfego geográfico em vez de dimensionar para atender a aumentos no tráfego. No entanto, você pode combinar esse padrão com outras soluções do Azure e locais. Por exemplo, esse padrão pode ser usado com o padrão de dimensionamento entre nuvens.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

O padrão garante o gerenciamento contínuo e a interface familiar entre ambientes.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Minha organização tem ramificações internacionais que exigem políticas regionais personalizadas de segurança e distribuição.
- Cada um dos escritórios da minha organização recebe dados de funcionários, de negócios e de instalações, exigindo a atividade de relatórios por regulamentos locais e fuso horário.
- Os requisitos de alta escala podem ser atendidos por meio da expansão horizontal de aplicativos, com várias implantações de aplicativo sendo feitas em uma única região e entre regiões para lidar com requisitos de carga extremo.
- Os aplicativos devem ser altamente disponíveis e responsivos às solicitações do cliente, mesmo em interrupções de região única.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte a [visão geral do Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como esse balanceador de carga de tráfego baseado em DNS funciona.
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de aplicativo distribuído geograficamente](solution-deployment-guide-geo-distributed.md). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes. Você aprende a direcionar o tráfego para pontos de extremidade específicos, com base em várias métricas usando o padrão de aplicativo distribuído geograficamente. Criar um perfil do Gerenciador de tráfego com roteamento baseado em geográfico e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.
