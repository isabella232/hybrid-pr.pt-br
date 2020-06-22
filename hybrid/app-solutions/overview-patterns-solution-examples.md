---
title: Padrões híbridos e exemplos de solução para o Azure e o Hub de Azure Stack
description: Uma visão geral dos padrões híbridos e exemplos de solução para aprender e criar soluções híbridas no Azure e no Hub de Azure Stack.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909790"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Padrões híbridos e exemplos de solução para o Azure e Azure Stack

A Microsoft fornece produtos e soluções do Azure e Azure Stack como um ecossistema do Azure consistente. A família de Microsoft Azure Stack é uma extensão do Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>A nuvem híbrida e os aplicativos híbridos

Azure Stack traz a agilidade da computação em nuvem para seu ambiente local e a borda habilitando uma *nuvem híbrida*. Azure Stack Hub, Azure Stack HCI e Azure Stack Edge estendem o Azure da nuvem para seus data centers do soberanas, filiais, campo e além. Com esse conjunto variado de recursos, você pode:

- Reutilize o código e execute aplicativos nativos de nuvem de forma consistente no Azure e em seus ambientes locais.
- Execute cargas de trabalho virtualizadas tradicionais com conexões opcionais para os serviços do Azure.
- Transfira dados para a nuvem ou mantenha-os em seu datacenter soberanas para manter a conformidade.
- Execute cargas de trabalho de aprendizado de máquina, em contêineres ou virtualizadas com aceleração de hardware, tudo na borda inteligente.

Aplicativos que abrangem nuvens também são chamados de *aplicativos híbridos*. Você pode criar aplicativos de nuvem híbrida no Azure e implantá-los em seu datacenter conectado ou desconectado, localizado em qualquer lugar.

Os cenários de aplicativos híbridos variam muito com os recursos que estão disponíveis para desenvolvimento. Eles também abrangem considerações como geografia, segurança, acesso à Internet e outros. Embora os padrões e as soluções descritos aqui possam não atender a todos os requisitos, eles fornecem diretrizes e exemplos para explorar e reutilizar enquanto implementam soluções híbridas.

## <a name="design-patterns"></a>Padrões de design

Os padrões de design escolhem diretrizes de design com repetição generalizada, de experiências e cenários reais de clientes. Um padrão é abstrato, permitindo que ele seja aplicável a diferentes tipos de cenários ou setores verticais. Cada padrão documenta o contexto e o problema e fornece uma visão geral de um exemplo de solução. O exemplo de solução é destinado como uma implementação possível do padrão.

Há dois tipos de artigos de padrão:

- Padrão único: fornece diretrizes de design para um único cenário de finalidade geral.
- Multipadrão: fornece diretrizes de design nas quais o aplicativo de vários padrões é usado. Esse padrão é frequentemente necessário para resolver cenários mais complexos ou problemas específicos do setor.

## <a name="solution-deployment-guides"></a>Guias de implantação da solução

Os guias passo a passo de implantação auxiliam na implantação de um exemplo de solução. O guia também pode se referir a um exemplo de código complementar, armazenado no [repositório de exemplo de soluções](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)do github.

## <a name="next-steps"></a>Próximas etapas

- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.
- Explore as seções "padrões" e "guias de implantação da solução" do Sumário para saber mais sobre cada uma delas.
- Leia sobre as [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para examinar os pilares da qualidade de software para design, implantação e operação de aplicativos híbridos.
- [Configure um ambiente de desenvolvimento no Azure Stack](/azure-stack/user/azure-stack-dev-start.md) e [implante seu primeiro aplicativo](/azure-stack/user/azure-stack-dev-start-deploy-app.md) no Azure Stack.
