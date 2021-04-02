---
title: Padrões híbridos e exemplos de solução para o Azure e o Azure Stack Hub
description: Uma visão geral dos padrões híbridos e dos exemplos de solução para aprender e criar soluções híbridas no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4f86e5ae4b8b9bd7693617b07419b67dfcf05dc1
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895305"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Padrões híbridos e exemplos de solução para o Azure e o Azure Stack

A Microsoft fornece os produtos e as soluções do Azure e do Azure Stack como um ecossistema consistente do Azure. A família Microsoft Azure Stack é uma extensão do Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>A nuvem híbrida e os aplicativos híbridos

O Azure Stack traz a agilidade da computação em nuvem para o ambiente local e a borda habilitando uma *nuvem híbrida*. O Azure Stack Hub, o Azure Stack HCI e o Azure Stack Edge estendem o Azure da nuvem para os data centers soberanos, filiais, campo e além. Com esse conjunto variado de recursos, você pode:

- Reutilizar o código e executar aplicativos nativos de nuvem de maneira consistente no Azure e nos ambientes locais.
- Executar cargas de trabalho virtualizadas tradicionais com conexões opcionais aos serviços do Azure.
- Transferir dados para a nuvem ou mantê-los no datacenter soberano para manter a conformidade.
- Executar cargas de trabalho de aprendizado de máquina, em contêineres ou virtualizadas com aceleração de hardware, tudo na borda inteligente.

Os aplicativos que abrangem nuvens também são chamados de *aplicativos híbridos*. Você pode criar aplicativos de nuvem híbrida no Azure e implantá-los em um datacenter conectado ou desconectado, localizado em qualquer lugar.

Os cenários de aplicativos híbridos variam muito de acordo com os recursos disponíveis para desenvolvimento. Eles também abrangem considerações como geografia, segurança, acesso à Internet e outras. Embora os padrões e as soluções descritos aqui possam não atender a todos os requisitos, eles oferecem diretrizes e exemplos para exploração e reutilização ao implementar soluções híbridas.

## <a name="design-patterns"></a>Padrões de design

Os padrões de design extraem diretrizes de design generalizadas e repetíveis de experiências e cenários reais de clientes. Um padrão é abstrato e pode ser aplicado a diferentes tipos de cenários ou setores verticais. Cada padrão documenta o contexto e o problema e apresenta uma visão geral de um exemplo de solução. O exemplo de solução é uma implementação possível do padrão.

Existem dois tipos de artigo de padrão:

- Padrão único: apresenta diretrizes de design para um único cenário de finalidade geral.
- Multipadrão: apresenta diretrizes de design com a aplicação de vários padrões. Esse padrão é frequentemente necessário para resolver cenários mais complexos ou problemas específicos do setor.

## <a name="solution-deployment-guides"></a>Guias de implantação de solução

Os guias de implantação passo a passo auxiliam na implantação de um exemplo de solução. O guia também pode se referir a um exemplo de código complementar, armazenado no [repositório de exemplo de soluções](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) do GitHub.

## <a name="next-steps"></a>Próximas etapas

- Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.
- Explore as seções "Padrões" e "Guias de implantação da solução" do Sumário para saber mais sobre cada uma delas.
- Leia sobre as [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para examinar os pilares da qualidade de software para criação, implantação e operação de aplicativos híbridos.
- [Configure um ambiente de desenvolvimento no Azure Stack](/azure-stack/user/azure-stack-dev-start) e [implante seu primeiro aplicativo](/azure-stack/user/azure-stack-dev-start-deploy-app) no Azure Stack.
