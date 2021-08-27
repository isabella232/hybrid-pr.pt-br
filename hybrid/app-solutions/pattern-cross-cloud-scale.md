---
title: Padrão de dimensionamento entre nuvens no Azure Stack Hub
description: Saiba como criar um aplicativo escalonável entre nuvens no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281254"
---
# <a name="cross-cloud-scaling-pattern"></a>Padrão de dimensionamento entre nuvens

Adicione recursos automaticamente a um aplicativo existente para acomodar um aumento na carga.

## <a name="context-and-problem"></a>Contexto e problema

Não é possível aumentar a capacidade do aplicativo para atender a aumentos inesperados na demanda. Devido à falta de escalabilidade, os usuários não conseguem acessar o aplicativo em horários de pico de uso. O aplicativo pode atender a um número fixo de usuários.

As empresas globais precisam de aplicativos baseados em nuvem que ofereçam segurança, confiança e disponibilidade. É essencial atender ao aumento na demanda e usar a infraestrutura correta para dar o respectivo suporte. Balancear os custos e a manutenção com a segurança, o armazenamento e a disponibilidade em tempo real dos dados de negócios é uma tarefa bastante complexa para as empresas.

Você pode não conseguir executar o aplicativo na nuvem pública. No entanto, pode ser economicamente inviável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda do aplicativo. Com esse padrão, você pode usar a elasticidade da nuvem pública com sua solução local.

## <a name="solution"></a>Solução

O padrão de dimensionamento entre nuvens estende um aplicativo que está em uma nuvem local com os recursos da nuvem pública. Ele é disparado por um aumento ou uma diminuição na demanda e, como consequência, adiciona recursos à nuvem ou os remove dela. Esses recursos fornecem redundância, disponibilidade rápida e roteamento com conformidade geográfica.

![Padrão de dimensionamento entre nuvens](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Este padrão aplica-se somente aos componentes sem estado do aplicativo.

## <a name="components"></a>Componentes

O padrão de dimensionamento entre nuvens consiste nos seguintes componentes:

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gerenciador de Tráfego

No diagrama, ele está fora do grupo da nuvem pública, mas precisa coordenar o tráfego tanto nela quanto no datacenter local. O balanceador garante alta disponibilidade para o aplicativo ao monitorar pontos de extremidade e fornecer redistribuição de failover quando necessário.

#### <a name="domain-name-system-dns"></a>Sistema de nome de domínio (DNS)

O sistema de nomes de domínio, ou DNS, é responsável por converter (ou seja, resolver) um nome do site ou serviço para seu endereço IP.

### <a name="cloud"></a>Nuvem

#### <a name="hosted-build-server"></a>Servidor de build hospedado

Um ambiente para hospedar o pipeline de build.

#### <a name="app-resources"></a>Recursos do aplicativo

Deve ser possível reduzir e escalar horizontalmente os recursos do aplicativo, assim como ocorre nos conjuntos de dimensionamento de máquinas virtuais e nos contêineres.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Use um nome de domínio personalizado para o glob de solicitações de roteamento.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Use os endereços IP públicos para rotear o tráfego de entrada pelo gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo da nuvem pública.  

### <a name="local-cloud"></a>Nuvem local

#### <a name="hosted-build-server"></a>Servidor de build hospedado

Um ambiente para hospedar o pipeline de build.

#### <a name="app-resources"></a>Recursos do aplicativo

Deve ser possível reduzir e escalar horizontalmente os recursos do aplicativo, assim como ocorre nos conjuntos de dimensionamento de máquinas virtuais e nos contêineres.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Use um nome de domínio personalizado para o glob de solicitações de roteamento.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Use os endereços IP públicos para rotear o tráfego de entrada pelo gerenciador de tráfego para o ponto de extremidade de recursos do aplicativo da nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

O principal componente do dimensionamento entre nuvens é a capacidade de oferecer dimensionamento sob demanda. Isso ocorre entre a infraestrutura da nuvem pública e a da nuvem local e fornece um serviço consistente e confiável de acordo com a demanda.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

O padrão entre nuvens garante gerenciamento contínuo e uma interface familiar entre os ambientes.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use este padrão:

- Quando for necessário aumentar a capacidade do aplicativo de acordo com demandas inesperadas ou periódicas.
- Quando você não quiser investir em recursos que serão usados somente durante picos. Pague apenas pelo que você usar.

Este padrão não é recomendado:

- Quando sua solução precisa que os usuários se conectem pela Internet.
- Quando sua empresa tem regulamentos locais que exigem que a conexão de origem venha de uma chamada no local.
- Quando sua rede passa por gargalos regulares que podem restringir o desempenho do dimensionamento.
- Quando seu ambiente está desconectado da Internet e não pode acessar a nuvem pública.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Confira a [visão geral do Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre o funcionamento do balanceador de carga de tráfego baseado em DNS.
- Confira as [considerações sobre o design de aplicativos híbridos](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas a outras perguntas.
- Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar a solução de exemplo, siga o [guia de implantação da solução de dimensionamento entre nuvens](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes. Você aprende a criar uma solução entre nuvens que fornece um processo disparado manualmente para alternar de um aplicativo Web hospedado no Azure Stack Hub para um aplicativo Web hospedado no Azure. O guia também ensina você a usar o dimensionamento automático por meio do gerenciador de tráfego, o que garante um utilitário de nuvem flexível e escalonável para aumentos de carga.