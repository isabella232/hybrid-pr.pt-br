---
title: Padrão de detecção de footfall usando o Azure e o Hub de Azure Stack
description: Saiba como usar o Azure e o Hub de Azure Stack para implementar uma solução de detecção de footfall com base em ia para analisar o tráfego da loja de varejo.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909834"
---
# <a name="footfall-detection-pattern"></a>Padrão de detecção de footfall

Esse padrão fornece uma visão geral para implementar uma solução de detecção de footfall com base em ia para analisar o tráfego de visitantes em lojas de varejo. A solução gera informações de ações do mundo real, usando o Azure, o Hub Azure Stack e o kit de desenvolvimento de ia Visão Personalizada.

## <a name="context-and-problem"></a>Contexto e problema

As lojas da Contoso gostariam de obter informações sobre como os clientes estão recebendo seus produtos atuais em relação ao layout de armazenamento. Eles não podem posicionar a equipe em todas as seções e não é eficiente ter uma equipe de analistas para revisar toda a seqüência de câmeras da loja. Além disso, nenhum de seus armazenamentos tem largura de banda suficiente para transmitir vídeo de todas as suas câmeras para a nuvem para análise.

A contoso gostaria de encontrar uma maneira discreta e amigável de privacidade para determinar os dados demográficos, a lealdade e as reações dos clientes para armazenar os vídeos e os produtos.

## <a name="solution"></a>Solução

Esse padrão de análise de varejo usa uma abordagem em camadas para inferência na borda. Usando o Visão Personalizada ia dev Kit, somente imagens com faces humanas são enviadas para análise para um hub de Azure Stack privado que executa serviços cognitivas do Azure. Os dados de agregação anônimos são enviados para o Azure para agregação em todas as lojas e visualização no Power BI. Combinar a borda e a nuvem pública permite que a contoso Aproveite a tecnologia de ia moderna, enquanto também permanece em conformidade com suas políticas corporativas e respeitando a privacidade de seus clientes.

[![Solução padrão de detecção de footfall](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Aqui está um resumo de como a solução funciona:

1. O Visão Personalizada ia dev kit Obtém uma configuração do Hub IoT, que instala o tempo de execução IoT Edge e um modelo de ML.
2. Se o modelo vir uma pessoa, ele usará uma imagem e a carregará para Azure Stack armazenamento de blobs de Hub.
3. O serviço blob dispara uma função do Azure no Hub Azure Stack.
4. A função do Azure chama um contêiner com o API de Detecção Facial para obter dados demográficos e emoções da imagem.
5. Os dados são anônimos e enviados para um cluster de hubs de eventos do Azure.
6. O cluster de hubs de eventos envia os dados para Stream Analytics.
7. Stream Analytics agrega os dados e envia-os por push para Power BI.

## <a name="components"></a>Componentes

Essa solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Hardware na loja | [Kit de desenvolvimento de ia Visão Personalizada](https://azure.github.io/Vision-AI-DevKit-Pages/) | Fornece filtragem na loja usando um modelo ML local que captura apenas imagens de pessoas para análise. Provisionado e atualizado com segurança por meio do Hub IoT.<br><br>|
| Azure | [Hubs de eventos do Azure](/azure/event-hubs/) | Os hubs de eventos do Azure fornecem uma plataforma escalonável para a ingestão de dados anônimos que se integram de maneira organizada com Azure Stream Analytics. |
|  | [Stream Analytics do Azure](/azure/stream-analytics/) | Um trabalho de Azure Stream Analytics agrega os dados anônimos e os agrupa em janelas de 15 segundos para visualização. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI fornece uma interface de painel fácil de usar para exibir a saída de Azure Stream Analytics. |
| Hub de Azure Stack | [Serviço de Aplicativo](/azure-stack/operator/azure-stack-app-service-overview.md) | O RP (provedor de recursos do serviço de aplicativo) fornece uma base para componentes de borda, incluindo recursos de hospedagem e gerenciamento para aplicativos Web/APIs e funções. |
| | Cluster do mecanismo do serviço de kubernetes do Azure [(AKs)](https://github.com/Azure/aks-engine) | O RP do AKS com o cluster AKS-Engine implantado no Hub Azure Stack fornece um mecanismo escalonável e resiliente para executar o contêiner API de Detecção Facial. |
| | [Contêineres de API de detecção facial](/azure/cognitive-services/face/face-how-to-install-containers) de serviços cognitivas do Azure| Os serviços cognitivas do Azure RP com contêineres API de Detecção Facial fornecem detecção de visitante demográfica, emoções e exclusiva na rede privada da contoso. |
| | Armazenamento de Blobs | As imagens capturadas do ia dev kit são carregadas no armazenamento de BLOBs do Hub Azure Stack. |
| | Funções do Azure | Uma função do Azure em execução no Hub Azure Stack recebe a entrada do armazenamento de BLOBs e gerencia as interações com o API de Detecção Facial. Ele emite dados anônimos para um cluster de hubs de eventos localizado no Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

Para permitir que essa solução seja dimensionada em várias câmeras e locais, você precisará certificar-se de que todos os componentes possam lidar com a carga aumentada. Talvez seja necessário executar ações como:

- Aumente o número de unidades de streaming de Stream Analytics.
- Escalar horizontalmente a implantação de API de Detecção Facial.
- Aumente a taxa de transferência do cluster dos hubs de eventos.
- Para casos extremos, migrar de Azure Functions para uma máquina virtual pode ser necessário.

### <a name="availability"></a>Disponibilidade

Como essa solução é em camadas, é importante pensar em como lidar com falhas de rede ou de energia. Dependendo das necessidades dos negócios, talvez você queira implementar um mecanismo para armazenar imagens em cache localmente e, em seguida, encaminhar para Azure Stack Hub quando a conectividade retornar. Se o local for grande o suficiente, a implantação de um Data Box Edge com o contêiner de API de Detecção Facial para esse local poderá ser uma opção melhor.

### <a name="manageability"></a>Capacidade de gerenciamento

Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado. Os [serviços de IOT do Azure](/azure/iot-fundamentals/) podem ser usados para colocar automaticamente novos locais e dispositivos online e mantê-los atualizados.

### <a name="security"></a>Segurança

Essa solução captura imagens de clientes, tornando a segurança uma consideração fundamental. Verifique se todas as contas de armazenamento estão protegidas com as políticas de acesso adequadas e alterne as chaves regularmente. Garanta que as contas de armazenamento e os hubs de eventos tenham políticas de retenção que atendam às normas de privacidade corporativas e governamentais. Além disso, certifique-se de hierarquizar os níveis de acesso do usuário. A disposição em camadas garante que os usuários tenham acesso apenas aos dados de que precisam para sua função.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte o [padrão de dados em camadas](https://aka.ms/tiereddatadeploy), que é aproveitado pelo padrão de detecção de footfall.
- Consulte o [visão personalizada ia dev kit](https://azure.github.io/Vision-AI-DevKit-Pages/) para saber mais sobre como usar a visão personalizada. 

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação de detecção de footfall](solution-deployment-guide-retail-footfall-detection.md). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.
