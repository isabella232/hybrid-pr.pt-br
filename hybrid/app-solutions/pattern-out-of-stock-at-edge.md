---
title: Detecção de indisponibilidade de estoque usando o Azure e o Azure Stack Edge
description: Saiba como usar os serviços do Azure e do Azure Stack Edge para implementar a detecção de indisponibilidade de estoque.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343868"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Padrão de detecção de indisponibilidade de estoque na borda

Este padrão ilustra como determinar se as prateleiras têm itens fora de estoque usando um dispositivo do Azure Stack Edge ou Azure IoT Edge e câmeras de rede.

## <a name="context-and-problem"></a>Contexto e problema

Lojas de varejo físicas perdem vendas porque, quando os clientes procuram um item, ele não está disponível nas prateleiras. No entanto, o item pode estar no depósito da loja e ainda não ter sido reposto. As lojas gostariam de usar suas equipes com mais eficiência e de ser notificadas automaticamente quando os itens precisarem ser repostos.

## <a name="solution"></a>Solução

A solução de exemplo usa em cada loja um dispositivo de borda, como um Azure Stack Edge, que processa com eficiência os dados das câmeras na loja. Esse design otimizado permite que as lojas enviem apenas imagens e eventos relevantes para a nuvem. O design economiza largura de banda e espaço de armazenamento e garante a privacidade dos clientes. À medida que os quadros são lidos de cada câmera, um modelo de ML processa a imagem e retorna as áreas sem estoque. A imagem e as áreas sem estoque são exibidas em um aplicativo Web local. Esses dados podem ser enviados a um ambiente do Time Series Insights para exibir insights no Power BI.

![Arquitetura da solução de borda de indisponibilidade de estoque](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Veja como a solução funciona:

1. As imagens são capturadas de uma câmera de rede por HTTP ou RTSP.
2. A imagem é redimensionada e enviada ao driver de inferência, que se comunica com o modelo de ML para determinar se há alguma imagem com indisponibilidade de estoque.
3. O modelo de ML retorna as áreas em que há indisponibilidade de estoque.
4. O driver de inferência carrega a imagem bruta em um blob (se especificado) e envia os resultados do modelo ao Hub IoT do Azure e um processador de caixa delimitadora no dispositivo.
5. O processador de caixa delimitadora adiciona caixas delimitadoras à imagem e armazena em cache o caminho da imagem em um banco de dados na memória.
6. O aplicativo Web consulta imagens e as mostra na ordem recebida.
7. As mensagens do Hub IoT são agregadas no Time Series Insights.
8. O Power BI exibe um relatório interativo de itens fora de estoque ao longo do tempo com os dados do Time Series Insights.


## <a name="components"></a>Componentes

Esta solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Hardware local | Câmera de rede | Uma câmera de rede é necessária, com um feed HTTP ou RTSP, para fornecer as imagens para inferência. |
| Azure | Hub IoT do Azure | O [Hub IoT do Azure](/azure/iot-hub/) manipula o provisionamento de dispositivos e as mensagens para os dispositivos de borda. |
|  | Azure Time Series Insights | O [Azure Time Series Insights](/azure/time-series-insights/) armazena as mensagens do Hub IoT para visualização. |
|  | Power BI | O [Microsoft Power BI](https://powerbi.microsoft.com/) fornece relatórios de negócios dos eventos de indisponibilidade de estoque. O Power BI oferece uma interface de dashboard fácil de usar para exibir a saída do Azure Stream Analytics. |
| Dispositivo Azure Stack Edge ou<br>Azure IoT Edge | Azure IoT Edge | O [Azure IoT Edge](/azure/iot-edge/) orquestra o runtime dos contêineres locais e manipula o gerenciamento de dispositivos e as atualizações.|
| | Project Brainwave do Azure | Em um dispositivo do Azure Stack Edge, o [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) usa FPGAs (matrizes de porta programável no campo) para acelerar a inferência de ML.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

A maioria dos modelos de machine learning pode ser executada somente em um determinado número de quadros por segundo, dependendo do hardware fornecido. Determine a taxa de amostragem ideal de suas câmeras para garantir que o pipeline de ML não sofra atrasos. Tipos diferentes de hardware processarão números diferentes de câmeras e taxas de quadros.

### <a name="availability"></a>Disponibilidade

É importante considerar o que poderá acontecer se o dispositivo de borda perder a conectividade. Considere quais dados poderão ser perdidos no Time Series Insights e no dashboard do Power BI. A solução de exemplo não foi projetada para ser altamente disponível.

### <a name="manageability"></a>Capacidade de gerenciamento

Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado. Os serviços de Internet das Coisas do Azure podem colocar automaticamente novos locais e dispositivos online e mantê-los atualizados. Procedimentos adequados de governança de dados também devem ser seguidos.

### <a name="security"></a>Segurança

Esse padrão manipula dados potencialmente confidenciais. Verifique se as chaves são revezadas regularmente e se as permissões na conta de Armazenamento do Azure e nos compartilhamentos locais estão definidas corretamente.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:
- Vários serviços relacionados à IoT são usados neste padrão, incluindo o [Azure IoT Edge](/azure/iot-edge/), o [Hub IoT do Azure](/azure/iot-hub/) e o [Azure Time Series Insights](/azure/time-series-insights/).
- Para saber mais sobre o Microsoft Project Brainwave, confira [o comunicado do blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) e assista ao [vídeo sore Machine Learning Acelerado do Azure com o Project Brainwave](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Confira as [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e obter respostas para outras perguntas.
- Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar a solução de exemplo, prossiga com o [Guia de implantação de solução de inferência de ML de borda](https://aka.ms/edgeinferencingdeploy). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes.
