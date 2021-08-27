---
title: Padrão de aplicativo distribuído geograficamente no Azure Stack Hub
description: Saiba mais sobre o padrão de aplicativo distribuído geograficamente com o Azure e o Azure Stack Hub para a borda inteligente.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281220"
---
# <a name="geo-distributed-app-pattern"></a>Padrão de aplicativo distribuído geograficamente

Saiba como fornecer pontos de extremidade de aplicativo em várias regiões e rotear o tráfego do usuário com base nas necessidades de local e conformidade.

## <a name="context-and-problem"></a>Contexto e problema

As empresas que abrangem diversas geografias se esforçam para distribuir e permitir o acesso aos dados com segurança e precisão, garantindo os níveis necessários de segurança, conformidade e desempenho por usuário, local e dispositivo entre as fronteiras.

## <a name="solution"></a>Solução

O padrão de roteamento de tráfego geográfico do Azure Stack Hub (ou aplicativos distribuídos geograficamente) permite que o tráfego seja direcionado para pontos de extremidade específicos com base em diversas métricas. Ao criar um Gerenciador de Tráfego com roteamento baseado em geografia e configuração de ponto de extremidade, você garante que o tráfego seja roteado para pontos de extremidade com base em requisitos regionais, regulamentações corporativas e internacionais e necessidades relativas aos dados.

![Padrão de distribuição geográfica](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Componentes

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gerenciador de Tráfego

No diagrama, ele está fora do grupo da nuvem pública, mas precisa coordenar o tráfego tanto nela quanto no datacenter local. O balanceador roteia esse tráfego para os locais geográficos.

#### <a name="domain-name-system-dns"></a>Sistema de nome de domínio (DNS)

O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.

### <a name="public-cloud"></a>Nuvem pública

#### <a name="cloud-endpoint"></a>Ponto de extremidade de nuvem

Use os endereços IP públicos para rotear o tráfego de entrada pelo gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo da nuvem pública.  

### <a name="local-clouds"></a>Nuvens locais

#### <a name="local-endpoint"></a>Ponto de extremidade local

Use os endereços IP públicos para rotear o tráfego de entrada pelo gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo da nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

O padrão faz o roteamento geográfico do tráfego em vez de lidar com os aumentos dele realizando o dimensionamento. No entanto, você pode combinar esse padrão com outras soluções locais e do Azure. Por exemplo, ele pode ser usado com o padrão de dimensionamento entre nuvens.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

O padrão garante gerenciamento contínuo e uma interface familiar entre os ambientes.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Sua empresa tem filiais internacionais que requerem políticas regionais personalizadas de segurança e distribuição.
- Cada escritório da sua empresa efetua pull de dados de funcionários, negócios e instalações, o que requer o relatório das atividades conforme os regulamentos locais e os fusos horários.
- É possível atender a requisitos de alta escala dimensionando aplicativos e realizando várias implantações deles em uma única região ou em várias para lidar com requisitos de carga extrema.
- Os aplicativos precisam ser altamente disponíveis e responsivos às solicitações do cliente, mesmo em caso de interrupções de região única.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Confira a [visão geral do Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre o funcionamento do balanceador de carga de tráfego baseado em DNS.
- Confira as [considerações sobre o design de aplicativos híbridos](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas a outras perguntas.
- Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar a solução de exemplo, siga o [guia de implantação de solução de aplicativo distribuído geograficamente](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes. Saiba como usar o padrão de aplicativos distribuídos geograficamente para direcionar o tráfego a pontos de extremidade específicos com base em várias métricas. Criar um perfil do Gerenciador de Tráfego com o roteamento baseado em geografia e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.