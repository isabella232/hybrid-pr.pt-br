---
title: Detecção de indisponibilidade de estoque usando o Azure e o Azure Stack Edge
description: Saiba como usar os serviços do Azure e do Azure Stack Edge para implementar a detecção de indisponibilidade de estoque.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909829"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Detecção de indisponibilidade de estoque no padrão de borda

Esse padrão ilustra como determinar se as prateleiras têm itens fora de estoque usando uma borda Azure Stack ou Azure IoT Edge dispositivos e câmeras de rede.

## <a name="context-and-problem"></a>Contexto e problema

Lojas de varejo físicas perdem vendas porque, quando os clientes procuram um item, ele não está presente na prateleira. No entanto, o item pode estar na parte de trás da loja e não foi recolocado em estoque. Os armazenamentos gostariam de usar sua equipe com mais eficiência e ser notificado automaticamente quando os itens precisarem de reabastecimento.

## <a name="solution"></a>Solução

O exemplo de solução usa um dispositivo de borda, como um Azure Stack Edge em cada loja, que processa com eficiência os dados de câmeras na loja. Esse design otimizado permite que os armazenamentos enviem apenas as imagens e os eventos relevantes para a nuvem. O design economiza largura de banda, espaço de armazenamento e garante a privacidade do cliente. À medida que os quadros são lidos de cada câmera, um modelo de ML processa a imagem e retorna todas as áreas de estoque. As áreas de imagem e fora de estoque são exibidas em um aplicativo Web local. Esses dados podem ser enviados para um ambiente de análise de série temporal para mostrar informações no Power BI.

![Fora de estoque na arquitetura da solução de borda](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Veja como a solução funciona:

1. As imagens são capturadas de uma câmera de rede por HTTP ou RTSP.
2. A imagem é redimensionada e enviada para o driver de inferência, que se comunica com o modelo ML para determinar se há alguma imagem de indisponibilidade de estoque.
3. O modelo ML retorna qualquer área fora de estoque.
4. O driver inferência carrega a imagem bruta em um blob (se especificado) e envia os resultados do modelo para o Hub IoT do Azure e um processador de caixa delimitadora no dispositivo.
5. O processador da caixa delimitadora adiciona caixas delimitadoras à imagem e armazena em cache o caminho da imagem em um banco de dados na memória.
6. O aplicativo Web consulta imagens e as mostra na ordem recebida.
7. As mensagens do Hub IoT são agregadas em Time Series Insights.
8. Power BI exibe um relatório interativo de itens fora de estoque ao longo do tempo com os dados de Time Series Insights.


## <a name="components"></a>Componentes

Essa solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Hardware local | Câmera de rede | Uma câmera de rede é necessária, com um feed HTTP ou RTSP para fornecer as imagens para inferência. |
| Azure | Hub IoT do Azure | O [Hub IOT do Azure](/azure/iot-hub/) manipula o provisionamento de dispositivos e mensagens para os dispositivos de borda. |
|  | Azure Time Series Insights | [Azure Time Series insights](/azure/time-series-insights/) armazena as mensagens do Hub IOT para visualização. |
|  | Power BI | [O Microsoft Power bi](https://powerbi.microsoft.com/) fornece relatórios voltados para a empresa de eventos de ausência de estoque. Power BI fornece uma interface de painel fácil de usar para exibir a saída de Azure Stream Analytics. |
| Borda de Azure Stack ou<br>Dispositivo Azure IoT Edge | Azure IoT Edge | [Azure IOT Edge](/azure/iot-edge/) orquestra o tempo de execução para os contêineres locais e manipula o gerenciamento de dispositivos e as atualizações.|
| | Brainwave do projeto do Azure | Em um dispositivo Azure Stack Edge, o [Project BrainWave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) usa FPGAs (matrizes de portão programável por campo) para acelerar o ml inferência.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

A maioria dos modelos de aprendizado de máquina só pode ser executada em um determinado número de quadros por segundo, dependendo do hardware fornecido. Determine a taxa de amostragem ideal de suas câmeras para garantir que o pipeline ML não faça backup. Tipos diferentes de hardware tratarão diferentes números de câmeras e taxas de quadros.

### <a name="availability"></a>Disponibilidade

É importante considerar o que pode acontecer se o dispositivo de borda perder a conectividade. Considere quais dados podem ser perdidos na Time Series Insights e Power BI painel. A solução de exemplo como fornecida não foi projetada para ser altamente disponível.

### <a name="manageability"></a>Capacidade de gerenciamento

Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado. Os serviços de IoT do Azure podem automaticamente colocar novos locais e dispositivos online e mantê-los atualizados. Os procedimentos de governança de dados adequados também devem ser seguidos.

### <a name="security"></a>Segurança

Esse padrão manipula dados potencialmente confidenciais. Verifique se as chaves são giradas regularmente e se as permissões na conta de armazenamento do Azure e nos compartilhamentos locais estão definidas corretamente.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:
- Vários serviços relacionados à IoT são usados neste padrão, incluindo [Azure IOT Edge](/azure/iot-edge/), [Hub IOT do Azure](/azure/iot-hub/)e [Azure Time Series insights](/azure/time-series-insights/).
- Para saber mais sobre o Microsoft Project BrainWave, confira [o comunicado do blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) e faça checkout do [Machine Learning acelerado do Azure com o Project BrainWave Video](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com os [dados em camadas para o guia de implantação da solução de análise](https://aka.ms/edgeinferencingdeploy). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.
