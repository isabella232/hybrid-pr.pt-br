---
title: Treinar o modelo de aprendizado de máquina no padrão de borda
description: Saiba como fazer treinamento de modelo do Machine Learning na borda com o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909816"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Treinar o modelo de aprendizado de máquina no padrão de borda

Gere modelos de ML (aprendizado de máquina portátil) a partir de dados que só existem localmente.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações gostariam de revelar informações de seus dados locais ou herdados usando as ferramentas que seus cientistas de dados entendem. O [Azure Machine Learning](/azure/machine-learning/) fornece ferramentas nativas de nuvem para treinar, ajustar e implantar modelos de aprendizado profundo e ml.  

No entanto, alguns dados são muito grandes enviar para a nuvem ou não podem ser enviados para a nuvem por motivos regulatórios. Usando esse padrão, os cientistas de dados podem usar Azure Machine Learning para treinar modelos usando dados e computação locais.

## <a name="solution"></a>Solução

O treinamento no padrão de borda usa uma VM (máquina virtual) em execução no Hub Azure Stack. A VM é registrada como um destino de computação no Azure ML, permitindo que ele acesse dados somente disponíveis localmente. Nesse caso, os dados são armazenados no armazenamento de BLOBs do Hub Azure Stack.

Depois que o modelo é treinado, ele é registrado com o Azure ML, em contêiner e adicionado a um registro de contêiner do Azure para implantação. Para essa iteração do padrão, a VM de treinamento do Hub Azure Stack deve estar acessível pela Internet pública.

[![Treinar modelo ML na arquitetura de borda](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Veja como o padrão funciona:

1. A VM do hub de Azure Stack é implantada e registrada como um destino de computação com o Azure ML.
2. Um experimento é criado no Azure ML que usa a VM do Hub Azure Stack como um destino de computação.
3. Depois que o modelo é treinado, ele é registrado e em contêiner.
4. O modelo agora pode ser implantado em locais que estejam no local ou na nuvem.

## <a name="components"></a>Componentes

Essa solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) orquestra o treinamento do modelo ml. |
| | Registro de Contêiner do Azure | O Azure ML empacota o modelo em um contêiner e o armazena em um [registro de contêiner do Azure](/azure/container-registry/) para implantação.|
| Hub de Azure Stack | Serviço de Aplicativo | [Azure Stack Hub com o serviço de aplicativo](/azure-stack/operator/azure-stack-app-service-overview) fornece a base para os componentes na borda. |
| | Computação | Uma VM de Hub de Azure Stack que executa o Ubuntu com o Docker é usada para treinar o modelo de ML. |
| | Armazenamento | Os dados privados podem ser hospedados no armazenamento de blobs de Hub Azure Stack. |

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

Para habilitar essa solução para escala, você precisará criar uma VM de tamanho adequado no Hub Azure Stack para treinamento.

### <a name="availability"></a>Disponibilidade

Verifique se os scripts de treinamento e a VM do hub de Azure Stack têm acesso aos dados locais usados para treinamento.

### <a name="manageability"></a>Capacidade de gerenciamento

Verifique se os modelos e experimentos estão adequadamente registrados, com versão e marcados para evitar confusão durante a implantação do modelo.

### <a name="security"></a>Segurança

Esse padrão permite que o Azure ML acesse dados confidenciais possíveis no local. Verifique se a conta usada para SSH em Azure Stack VM do Hub tem uma senha forte e os scripts de treinamento não preservam nem carregam dados na nuvem.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte a [documentação do Azure Machine Learning](/azure/machine-learning) para obter uma visão geral do ml e tópicos relacionados.
- Consulte [registro de contêiner do Azure](/azure/container-registry/) para saber como criar, armazenar e gerenciar imagens para implantações de contêiner.
- Consulte [serviço de aplicativo no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) para saber mais sobre o provedor de recursos e como implantá-lo.
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e para obter respostas adicionais.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [modelo treinar ml no guia de implantação do Edge](https://aka.ms/edgetrainingdeploy). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.
