---
title: Padrão de detecção de clientela usando o Azure e o Azure Stack Hub
description: Saiba como usar o Azure e o Azure Stack Hub para implementar uma solução de detecção de clientela com base em IA para analisar o tráfego de uma loja de varejo.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281271"
---
# <a name="footfall-detection-pattern"></a>Padrão de detecção de clientela

O padrão apresenta uma visão geral para implementar uma solução de detecção de clientela com base em IA para analisar o tráfego de visitantes em lojas de varejo. A solução gera insights de ações do mundo real usando o Azure, o Azure Stack Hub e o Kit de Desenvolvimento de IA da Visão Personalizada.

## <a name="context-and-problem"></a>Contexto e problema

A Contoso Stores gostaria de entender como os clientes estão recebendo os produtos atuais em relação ao layout da loja. A empresa não pode posicionar funcionários em todas as seções, e não é eficiente ter uma equipe de analistas para assistir a todos os vídeos das câmeras. Além disso, nenhuma das lojas tem largura de banda suficiente para transmitir vídeo de todas as câmeras para a nuvem para análise.

A Contoso quer encontrar uma forma discreta e amigável de determinar os dados demográficos, a fidelidade e a reação dos clientes aos mostruários e produtos nas lojas, respeitando a privacidade deles.

## <a name="solution"></a>Solução

Este padrão de análise de varejo usa uma abordagem em camadas para inferência na borda. Usando o Kit de Desenvolvimento de IA da Visão Personalizada, somente as imagens com faces humanas são enviadas para análise em um Azure Stack Hub privado, que executa Serviços Cognitivos do Azure. Os dados agregados e anônimos são enviados ao Azure para agregação em todas as lojas e visualização no Power BI. Ao combinar a borda e a nuvem pública, a Contoso pode usar a tecnologia moderna de IA sem deixar de cumprir as políticas corporativas e com respeito à privacidade dos clientes.

[![Solução de padrão de detecção de clientela](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Aqui está um resumo de como a solução funciona:

1. O Kit de Desenvolvimento de IA da Visão Personalizada recebe uma configuração do Hub IoT, que instala o Runtime do IoT Edge e um modelo de ML.
2. Quando detecta uma pessoa, o modelo captura uma imagem e a carrega no armazenamento de blobs do Azure Stack Hub.
3. O serviço Blob dispara uma função do Azure no Azure Stack Hub.
4. A função do Azure chama um contêiner com a API de Detecção Facial para obter dados demográficos e emocionais da imagem.
5. Os dados são anonimizados e enviados a um cluster dos Hubs de Eventos do Azure.
6. O cluster dos Hubs de Eventos envia os dados ao Stream Analytics.
7. O Stream Analytics agrega os dados e efetua push no Power BI.

## <a name="components"></a>Componentes

Esta solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Hardware na loja | [Kit de Desenvolvimento de IA da Visão Personalizada](https://azure.github.io/Vision-AI-DevKit-Pages/) | Realiza filtragem na loja usando um modelo de ML local que captura apenas imagens de pessoas para análise. Provisionado e atualizado com segurança por meio do Hub IoT.<br><br>|
| Azure | [Hubs de eventos do Azure](/azure/event-hubs/) | Os Hubs de Eventos do Azure são uma plataforma escalonável para a ingestão de dados anônimos, com integração organizada ao Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Um trabalho do Azure Stream Analytics agrega os dados anônimos e os agrupa em janelas de 15 segundos para visualização. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | O Power BI oferece uma interface de dashboard fácil de usar para exibir a saída do Azure Stream Analytics. |
| Azure Stack Hub | [Serviço de Aplicativo](/azure-stack/operator/azure-stack-app-service-overview) | O RP (provedor de recursos) do Serviço de Aplicativo oferece uma base para componentes de borda, incluindo recursos de hospedagem e gerenciamento para aplicativos Web/APIs e funções. |
| | Cluster de mecanismo do AKS [(Serviço de Kubernetes do Azure)](https://github.com/Azure/aks-engine) | O RP do AKS com o cluster do mecanismo do AKS implantado no Azure Stack Hub oferece um mecanismo escalonável e resiliente para executar o contêiner da API de Detecção Facial. |
| | [Contêineres da API de Detecção Facial](/azure/cognitive-services/face/face-how-to-install-containers) dos Serviços Cognitivos do Azure| O RP dos Serviços Cognitivos do Azure com contêineres da API de Detecção Facial realiza a detecção de dados demográficos, emocionais e de visitantes individuais na rede privada da Contoso. |
| | Armazenamento de Blobs | As imagens capturadas pelo Kit de Desenvolvimento de IA são carregadas no armazenamento de blobs do Azure Stack Hub. |
| | Funções do Azure | Uma função do Azure em execução no Azure Stack Hub recebe a entrada do armazenamento de blobs e gerencia as interações com a API de Detecção Facial. Ela emite dados anônimos para um cluster dos Hubs de Eventos localizado no Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

Para permitir que essa solução seja dimensionada em várias câmeras e locais, verifique se todos os componentes podem lidar com o aumento da carga. Talvez seja necessário executar ações como:

- Aumentar o número de unidades de streaming do Stream Analytics.
- Escalar horizontalmente a implantação da API de Detecção Facial.
- Aumentar a taxa de transferência do cluster dos Hubs de Eventos.
- Em casos extremos, pode ser necessário migrar do Azure Functions para uma máquina virtual.

### <a name="availability"></a>Disponibilidade

Como essa solução é em camadas, é importante pensar em como lidar com falhas de rede ou de energia. Dependendo das necessidades dos negócios, pode ser útil implementar um mecanismo para armazenar as imagens em cache localmente e, em seguida, encaminhá-las ao Azure Stack Hub quando a conectividade retornar. Se o local for grande o suficiente, a implantação de um Data Box Edge com o contêiner da API de Detecção Facial para esse local poderá ser uma opção melhor.

### <a name="manageability"></a>Capacidade de gerenciamento

Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado. Os [serviços de IoT do Azure](/azure/iot-fundamentals/) podem ser usados para colocar automaticamente novos locais e dispositivos online e mantê-los atualizados.

### <a name="security"></a>Segurança

Como essa solução captura imagens de clientes, a segurança é uma consideração fundamental. Proteja todas as contas de armazenamento com as políticas de acesso adequadas e alterne as chaves regularmente. Para as contas de armazenamento e os Hubs de Eventos, adote políticas de retenção que atendam às normas de privacidade corporativas e governamentais. Além disso, adote camadas nos níveis de acesso do usuário. A disposição em camadas garante que os usuários tenham acesso apenas aos dados de que precisam para sua função.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte o [padrão de dados em camadas](https://aka.ms/tiereddatadeploy), que é usado pelo padrão de detecção de clientela.
- Consulte o [Kit de Desenvolvimento de IA da Visão Personalizada](https://azure.github.io/Vision-AI-DevKit-Pages/) para saber mais sobre como usar a visão personalizada. 

Quando tudo estiver pronto para testar o exemplo de solução, prossiga com o [guia de implantação da detecção de clientela](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes.