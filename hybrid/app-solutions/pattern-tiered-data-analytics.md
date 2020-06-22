---
title: Dados em camadas para o padrão de análise usando o Azure e o Hub de Azure Stack
description: Saiba como usar o Azure e o Hub de Azure Stack para implementar uma solução de dados em camadas em toda a nuvem híbrida.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909837"
---
# <a name="tiered-data-for-analytics-pattern"></a>Dados em camadas para o padrão de análise

Esse padrão ilustra como usar Azure Stack Hub e o Azure para preparar, analisar, processar, limpar e armazenar dados em vários locais de nuvem e local.

## <a name="context-and-problem"></a>Contexto e problema

Um dos problemas enfrentados pelas organizações empresariais no cenário de tecnologia moderna se refere ao armazenamento de dados seguro, processamento e análise. As considerações incluem:

- conteúdo de dados
- local
- requisitos de segurança e privacidade
- permissões de acesso
- manutenção
- depósito de armazenamento

O Azure, junto com o Hub de Azure Stack, soluciona problemas de dados e oferece soluções de baixo custo. Essa solução é melhor expressa por meio de uma empresa distribuída de manufatura ou logística.

A solução é baseada no cenário a seguir:

- Uma grande organização de manufatura de várias ramificações.
- O armazenamento de dados rápido e seguro, o processamento e a distribuição entre locais remotos globais e suas sedes centrais são necessários.
- Atividade de funcionários e máquinas, informações de instalações e dados de relatórios de negócios que devem permanecer seguros. Os dados devem ser distribuídos adequadamente e atender às normas da política de conformidade regional e do setor.

## <a name="solution"></a>Solução

O uso de ambientes de nuvem pública e local atende às demandas de várias instalações de empresas. O Hub de Azure Stack oferece uma solução rápida, segura e flexível para coletar, processar, armazenar e distribuir dados locais e remotos. Esse padrão é especialmente útil quando a segurança, a confidencialidade, a política corporativa e os requisitos regulatórios podem ser diferentes entre locais e usuários.

![Padrão de dados em camadas para arquitetura de solução de análise](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Componentes

Esse padrão usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Azure | Armazenamento | Uma conta de [armazenamento do Azure](/azure/storage/) fornece um ponto de extremidade de consumo de dados estéril. O armazenamento do Azure é uma solução de armazenamento em nuvem da Microsoft para cenários de armazenamento de dados modernos. O armazenamento do Azure oferece um armazenamento de objetos altamente escalonável para objetos de dados e um serviço de sistema de arquivos para a nuvem. Ele também fornece um repositório de mensagens para mensagens confiáveis e um repositório NoSQL. |
| Hub de Azure Stack | Armazenamento | Uma conta de [armazenamento de Hub Azure Stack](/azure-stack/user/azure-stack-storage-overview) é usada para vários serviços:<br><br>- **Armazenamento de BLOBs** para armazenamento de dados brutos. O armazenamento de BLOBs pode conter qualquer tipo de dados de texto ou binários, como um documento, arquivo de mídia ou instalador de aplicativo. Cada blob é organizado em um contêiner. Os contêineres fornecem uma maneira útil de atribuir políticas de segurança a grupos de objetos. Uma conta de armazenamento pode conter qualquer número de contêineres, e um contêiner pode conter qualquer número de BLOBs, até o limite de capacidade de 500 TB da conta de armazenamento.<br>- **Armazenamento de BLOBs** para arquivamento de dados. Há benefícios para o armazenamento de baixo custo para arquivamento de dados frios. Exemplos de dados interessantes incluem backups, conteúdo de mídia, dados científicos, conformidade e dados de arquivamento. Em geral, todos os dados que são acessados raramente são considerados armazenamento frio. Camadas de dados com base em atributos como frequência de acesso e período de retenção. Os dados do cliente são acessados com pouca frequência, mas exigem latência e desempenho semelhantes para dados ativos.<br>- **Armazenamento de filas** para armazenamento de dados processados. O armazenamento de filas fornece mensagens de nuvem entre os componentes do aplicativo. Na criação de aplicativos para escala, os componentes do aplicativo geralmente são dissociados para que possam ser dimensionados de forma independente. O armazenamento de filas fornece mensagens assíncronas para comunicação entre os componentes do aplicativo, se eles estão em execução na nuvem, na área de trabalho, em um servidor local ou em um dispositivo móvel. O armazenamento de Fila também dá suporte ao gerenciamento de tarefas assíncronas e à criação de fluxos de trabalho do processo. |
| | Funções do Azure | O serviço de [Azure Functions](/azure/azure-functions/) é fornecido pelo [serviço de Azure app no provedor de recursos do Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) . Azure Functions permite que você execute seu código em um ambiente simples e sem servidor em resposta a uma variedade de eventos. Azure Functions escalar para atender à demanda sem precisar criar uma VM ou publicar um aplicativo Web, usando a linguagem de programação de sua escolha. As funções são usadas pela solução para:<br><br>- **Entrada de dados**<br>- **Sterilization de dados.** As funções disparadas manualmente podem executar processamento de dados agendado, limpeza e arquivamento. Exemplos podem incluir limpezas de lista de clientes à noite e processamento de relatório mensal.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

As soluções de Azure Functions e armazenamento são dimensionadas para atender às demandas de processamento e volume de dados. Para obter informações de escalabilidade e destinos do Azure, consulte [documentação de escalabilidade do armazenamento do Azure](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Disponibilidade

O armazenamento é a principal consideração de disponibilidade para esse padrão. A conexão por meio de links rápidos é necessária para o processamento e a distribuição de grandes volumes de dados.

### <a name="manageability"></a>Capacidade de gerenciamento

A capacidade de gerenciamento dessa solução depende das ferramentas de criação em uso e do envolvimento do controle do código-fonte.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte o [armazenamento do Azure](/azure/storage/) e a documentação de [Azure Functions](/azure/azure-functions/) . Esse padrão faz uso intensivo de contas de armazenamento do Azure e Azure Functions no Azure e no Hub de Azure Stack.
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para perguntas adicionais.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com os [dados em camadas para o guia de implantação da solução de análise](https://aka.ms/tiereddatadeploy). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.
