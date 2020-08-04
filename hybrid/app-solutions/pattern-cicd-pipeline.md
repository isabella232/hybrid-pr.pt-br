---
title: O padrão DevOps no Azure Stack Hub
description: Saiba mais sobre o padrão DevOps para que você possa garantir a consistência entre implantações no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477228"
---
# <a name="devops-pattern"></a>Padrão DevOps

Codifique a partir de um único local e implante em vários destinos em ambientes de desenvolvimento, teste e produção que estejam em seu datacenter local, em nuvens privadas ou na nuvem pública.

## <a name="context-and-problem"></a>Contexto e problema

A continuidade, a segurança e a confiabilidade da implantação de aplicativos são essenciais para as organizações e para as equipes de desenvolvimento.

Geralmente, os aplicativos exigem código refatorado para serem executados em cada ambiente de destino. Isso significa que um aplicativo não é totalmente portátil. Ele deve ser atualizado, testado e validado à medida que passa por cada ambiente. Por exemplo, o código escrito em um ambiente de desenvolvimento deve ser reescrito para funcionar em um ambiente de teste e reescrito mais uma vez quando finalmente chegar em um ambiente de produção. Além disso, esse código estará especificamente vinculado ao host. Isso aumenta o custo e a complexidade de manutenção do aplicativo. Cada versão do aplicativo está vinculada a cada ambiente. O excesso de complexidade e duplicação aumenta o risco à segurança e à qualidade do código. O código também não poderá ser reimplantado prontamente quando você remover hosts com falha na restauração ou implantar hosts adicionais para lidar com aumentos na demanda.

## <a name="solution"></a>Solução

O padrão DevOps permite criar, testar e implantar um aplicativo para ser executado em várias nuvens. Esse padrão unifica as práticas de integração contínua e entrega contínua. Na integração contínua, o código é criado e testado sempre que um membro da equipe confirma uma alteração no controle de versão. A entrega contínua automatiza cada etapa, desde a compilação até o ambiente de produção. Juntos, esses procedimentos criam um processo de liberação que dá suporte à implantação em diversos ambientes. Com esse padrão, você pode rascunhar seu código e, em seguida, implantar o mesmo código em um ambiente local, em diferentes nuvens privadas e nas nuvens públicas. As diferenças no ambiente exigem uma alteração no arquivo de configuração em vez de alterações no código.

![Padrão DevOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Com um conjunto consistente de ferramentas de desenvolvimento em ambientes locais, de nuvem privada e de nuvem pública, você pode implementar uma prática de integração contínua e entrega contínua. Os aplicativos e serviços implantados usando o padrão DevOps são intercambiáveis e podem ser executados em qualquer ambiente, tirando proveito dos recursos e funcionalidades locais e de nuvem pública.

O uso de um pipeline de lançamento DevOps ajuda a:

- Iniciar uma nova compilação com base em confirmações de código em um único repositório.
- Implantar automaticamente o código recém-criado na nuvem pública para o teste de aceitação do usuário.
- Fazer a implantação automática em uma nuvem privada depois que o código passa no teste.

## <a name="issues-and-considerations"></a>Problemas e considerações

O padrão DevOps destina-se a garantir a consistência entre implantações, independentemente do ambiente de destino. No entanto, os recursos variam em ambientes de nuvem e locais. Considere os seguintes pontos:

- As funções, os pontos de extremidade, os serviços e outros recursos de sua implantação estão disponíveis nos locais de destino?
- Os artefatos de configuração são armazenados em locais acessíveis entre nuvens?
- Os parâmetros de implantação funcionarão em todos os ambientes de destino?
- As propriedades específicas dos recursos estão disponíveis em todas as nuvens de destino?

Para obter mais informações, confira [Desenvolva modelos do Azure Resource Manager para consistência de nuvem](/azure/azure-resource-manager/templates-cloud-consistency).

Além disso, os seguintes pontos devem ser considerados na decisão de como implementar esse padrão:

### <a name="scalability"></a>Escalabilidade

Os sistemas de automação de implantação são o ponto de controle principal nos padrões DevOps. As Implementações podem variar. A seleção do tamanho de servidor correto depende do tamanho da carga de trabalho esperada. Dimensionar VMs é mais oneroso do que dimensionar contêineres. Para usar os contêineres para o dimensionamento, no entanto, o processo de compilação deve executar com os contêineres.

### <a name="availability"></a>Disponibilidade

Disponibilidade no contexto do DevPattern significa ser capaz de recuperar as informações de estado associadas ao fluxo de trabalho, como resultados de teste, dependências do código ou outros artefatos. Para avaliar seus requisitos de disponibilidade, considere duas métricas comuns:

- O RTO (Objetivo de Tempo de Recuperação) especifica quanto tempo você pode ficar sem um sistema.

- O RPO (Objetivo de Ponto de Recuperação) indica quantos dados você pode perder se uma interrupção no serviço afetar o sistema.

Na prática, o RTO e o RPO representam a redundância e o backup. Na nuvem global do Azure, a disponibilidade não é uma questão de recuperação de hardware - que faz parte do Azure - mas sim de garantir que você mantenha o estado de seus sistemas DevOps. No Azure Stack Hub, a recuperação de hardware pode ser uma consideração.

Outra consideração importante na criação do sistema usado para a automação da implantação é o controle de acesso e o gerenciamento adequado dos direitos necessários para a implantação de serviços em ambientes de nuvem. Que direitos são necessários para a criação, exclusão ou modificação de implantações? Por exemplo, normalmente há um conjunto de direitos para a criação de um grupo de recursos no Azure e outro para a implantação de serviços no grupo de recursos.

### <a name="manageability"></a>Capacidade de gerenciamento

O design de qualquer sistema baseado no padrão DevOps deve considerar automação, registro em log e alertas para cada serviço de todo o portfólio. Use serviços compartilhados, uma equipe de aplicativos, ou ambos, e acompanhe também a governança e as políticas de segurança.

Implante ambientes de produção e ambientes de desenvolvimento/teste em grupos de recursos separados no Azure ou no Azure Stack Hub. Assim, você poderá monitorar os recursos de cada ambiente e acumular os custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é útil para implantações de teste.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão se:

- Puder desenvolver códigos em um ambiente que atenda às necessidades de seus desenvolvedores e para fazer a implantação em um ambiente específico para sua solução, onde seja difícil desenvolver um novo código.
- Puder utilizar o código e as ferramentas que seus desenvolvedores desejam, contanto que eles possam seguir o processo de integração contínua e entrega contínua no padrão DevOps.

Este padrão não é recomendável:

- Se você não puder automatizar a infraestrutura, o provisionamento de recursos, a configuração, a identidade e as tarefas de segurança.
- Se as equipes não tiverem acesso aos recursos de nuvem híbrida para implementar uma abordagem de CI/CD (integração contínua/desenvolvimento contínuo).

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Confira a [documentação do Azure DevOps](/azure/devops) para se informar melhor sobre o Azure DevOps e as ferramentas relacionadas, incluindo o Azure Repos e o Azure Pipelines.
- Confira a [família de produtos e soluções do Azure Stack](/azure-stack) para se informar melhor sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo de solução, dê prosseguimento com o [guia de implantação de solução híbrida de CI/CD do DevOps](https://aka.ms/hybriddevopsdeploy). O guia de implantação fornece instruções passo a passo para a implantação e o teste de seus componentes. Você pode aprender a implantar um aplicativo no Azure e no Azure Stack Hub usando um pipeline híbrido de CI/CD (integração contínua/entrega contínua).
