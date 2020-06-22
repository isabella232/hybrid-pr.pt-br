---
title: O padrão DevOps no Hub Azure Stack
description: Saiba mais sobre o padrão DevOps para que você possa garantir a consistência entre implantações no Azure e no Hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909819"
---
# <a name="devops-pattern"></a>Padrão DevOps

Codifique a partir de um único local e implante em vários destinos em ambientes de desenvolvimento, teste e produção que podem estar em seu datacenter local, em nuvens privadas ou na nuvem pública.

## <a name="context-and-problem"></a>Contexto e problema

A continuidade, a segurança e a confiabilidade da implantação de aplicativos são essenciais para as organizações e para as equipes de desenvolvimento.

Os aplicativos geralmente exigem código refatorado para serem executados em cada ambiente de destino. Isso significa que um aplicativo não é completamente portátil. Ele deve ser atualizado, testado e validado à medida que se move em cada ambiente. Por exemplo, o código escrito em um ambiente de desenvolvimento deve ser reescrito para funcionar em um ambiente de teste e reescrito quando ele finalmente chega em um ambiente de produção. Além disso, esse código está especificamente vinculado ao host. Isso aumenta o custo e a complexidade de manter seu aplicativo. Cada versão do aplicativo é vinculada a cada ambiente. A maior complexidade e a duplicação aumentam o risco de segurança e qualidade de código. Além disso, o código não pode ser reimplantado prontamente quando você remove os hosts com falha de restauração ou implanta hosts adicionais para lidar com aumentos na demanda.

## <a name="solution"></a>Solução

O padrão DevOps permite criar, testar e implantar um aplicativo que é executado em várias nuvens. Esse padrão une a prática da integração contínua e da entrega contínua. Com a integração contínua, o código é criado e testado toda vez que um membro da equipe confirma uma alteração no controle de versão. A entrega contínua automatiza cada etapa de um Build para um ambiente de produção. Juntos, esses processos criam um processo de liberação que dá suporte à implantação em diversos ambientes. Com esse padrão, você pode rascunhar seu código e, em seguida, implantar o mesmo código em um ambiente premisso, diferentes nuvens privadas e as nuvens públicas. As diferenças no ambiente exigem uma alteração em um arquivo de configuração em vez de alterações no código.

![Padrão DevOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Com um conjunto consistente de ferramentas de desenvolvimento em ambientes locais, de nuvem privada e de nuvem pública, você pode implementar uma prática de integração contínua e entrega contínua. Os aplicativos e serviços implantados usando o padrão DevOps são intercambiáveis e podem ser executados em qualquer um desses locais, tirando proveito dos recursos e funcionalidades de nuvem pública e local.

O uso de um pipeline de liberação do DevOps ajuda você a:

- Inicie uma nova compilação com base em confirmações de código em um único repositório.
- Implante automaticamente seu código recém-criado na nuvem pública para o teste de aceitação do usuário.
- Implante automaticamente em uma nuvem privada depois que seu código passar por testes.

## <a name="issues-and-considerations"></a>Problemas e considerações

O padrão DevOps destina-se a garantir a consistência entre implantações, independentemente do ambiente de destino. No entanto, os recursos variam em ambientes de nuvem e locais. Considere os seguintes pontos:

- As funções, os pontos de extremidade, os serviços e outros recursos em sua implantação estão disponíveis nos locais de implantação de destino?
- Os artefatos de configuração são armazenados em locais acessíveis entre nuvens?
- Os parâmetros de implantação funcionarão em todos os ambientes de destino?
- As propriedades específicas do recurso estão disponíveis em todas as nuvens de destino?

Para obter mais informações, consulte [desenvolver modelos de Azure Resource Manager para consistência de nuvem](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency).

Além disso, considere os seguintes pontos ao decidir como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

Os sistemas de automação de implantação são o ponto de controle principal nos padrões de DevOps. As implementações podem variar. A seleção do tamanho do servidor correto depende do tamanho da carga de trabalho esperada. As VMs custam mais para dimensionar os contêineres. Para usar os contêineres para o dimensionamento, no entanto, o processo de compilação deve executar com os contêineres.

### <a name="availability"></a>Disponibilidade

A disponibilidade no contexto do DevPattern significa ser capaz de recuperar qualquer informação de estado associada ao fluxo de trabalho, como resultados de teste, dependências de código ou outros artefatos. Para avaliar seus requisitos de disponibilidade, considere duas métricas comuns:

- O RTO (objetivo de tempo de recuperação) especifica por quanto tempo você pode ir sem um sistema.

- O RPO (objetivo de ponto de recuperação) indica a quantidade de dados que você pode perder se uma interrupção no serviço afetar o sistema.

Na prática, o RTO e o RPO implicam redundância e backup. Na nuvem global do Azure, a disponibilidade não é uma questão de recuperação de hardware — que faz parte do Azure — mas, em vez disso, garante que você mantenha o estado dos seus sistemas DevOps. No Hub Azure Stack, a recuperação de hardware pode ser uma consideração.

Outra consideração importante ao criar o sistema usado para a automação da implantação é o controle de acesso e o gerenciamento adequado dos direitos necessários para implantar serviços em ambientes de nuvem. Quais direitos são necessários para criar, excluir ou modificar implantações? Por exemplo, um conjunto de direitos normalmente é necessário para criar um grupo de recursos no Azure e outro para implantar serviços no grupo de recursos.

### <a name="manageability"></a>Capacidade de gerenciamento

O design de qualquer sistema baseado no padrão DevOps deve considerar a automação, o registro em log e o alerta para cada serviço em todo o portfólio. Use serviços compartilhados, uma equipe de aplicativos, ou ambos, e acompanhe políticas de segurança e governança também.

Implante ambientes de produção e ambientes de desenvolvimento/teste em grupos de recursos separados no Azure ou no Hub de Azure Stack. Em seguida, você pode monitorar os recursos de cada ambiente e acumular os custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é útil para implantações de teste.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão se:

- Você pode desenvolver código em um ambiente que atenda às necessidades de seus desenvolvedores e implantar em um ambiente específico para sua solução, onde pode ser difícil desenvolver um novo código.
- Você pode usar o código e as ferramentas que seus desenvolvedores desejam, contanto que eles possam seguir o processo de integração contínua e de entrega contínua no padrão DevOps.

Este padrão não é recomendável:

- Se você não puder automatizar a infraestrutura, o provisionamento de recursos, a configuração, a identidade e as tarefas de segurança.
- Se as equipes não tiverem acesso aos recursos de nuvem híbrida para implementar uma abordagem de CI/CD (integração contínua/desenvolvimento contínuo).

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Consulte a [documentação do Azure DevOps](/azure/devops) para saber mais sobre as DevOps do Azure e as ferramentas relacionadas, incluindo Azure Repos e Azure pipelines.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de CI/CD híbrido do DevOps](https://aka.ms/hybriddevopsdeploy). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes. Você aprende a implantar um aplicativo no Azure e no Hub de Azure Stack usando um pipeline de CI/CD (integração contínua/entrega contínua).
