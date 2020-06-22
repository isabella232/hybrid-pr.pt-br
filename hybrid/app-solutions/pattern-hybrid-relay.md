---
title: Padrão de retransmissão híbrida no Azure e no Hub de Azure Stack
description: Use o padrão de retransmissão híbrida no Azure e Azure Stack Hub para se conectar aos recursos de borda protegidos por firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909823"
---
# <a name="hybrid-relay-pattern"></a>Padrão de retransmissão híbrida

Saiba como se conectar a recursos de borda ou dispositivos protegidos por firewalls usando o padrão de retransmissão híbrida e a retransmissão do Azure.

## <a name="context-and-problem"></a>Contexto e problema

Dispositivos de borda geralmente estão protegidos por um firewall corporativo ou dispositivo NAT. Embora eles sejam seguros, eles podem não conseguir se comunicar com os dispositivos de nuvem pública ou de borda em outras redes corporativas. Pode ser necessário expor determinadas portas e funcionalidades para os usuários na nuvem pública de maneira segura.

## <a name="solution"></a>Solução

O padrão de retransmissão híbrida usa a retransmissão do Azure para estabelecer um túnel WebSocket entre dois pontos de extremidade que não podem se comunicar diretamente. Dispositivos que não são locais, mas precisam se conectar a um ponto de extremidade local se conectarão a um ponto de extremidade na nuvem pública. Esse ponto de extremidade redirecionará o tráfego em rotas predefinidas por um canal seguro. Um ponto de extremidade dentro do ambiente local recebe o tráfego e o roteia para o destino correto.

![arquitetura da solução de padrão de retransmissão híbrida](media/pattern-hybrid-relay/solution-architecture.png)

Veja como funciona o padrão de retransmissão híbrida:

1. Um dispositivo se conecta à VM (máquina virtual) no Azure, em uma porta predefinida.
2. O tráfego é encaminhado para a retransmissão do Azure no Azure.
3. A VM no Hub Azure Stack, que já estabeleceu uma conexão de vida longa para a retransmissão do Azure, recebe o tráfego e o encaminha para o destino.
4. O serviço ou ponto de extremidade local processa a solicitação.

## <a name="components"></a>Componentes

Essa solução usa os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Azure | VM do Azure | Uma VM do Azure fornece um ponto de extremidade publicamente acessível para o recurso local. |
| | Retransmissão do Azure | Uma [retransmissão do Azure](/azure/azure-relay/) fornece a infraestrutura para manter o túnel e a conexão entre a VM do Azure e a VM do Hub de Azure Stack.|
| Hub de Azure Stack | Computação | Uma VM de Hub de Azure Stack fornece o lado do servidor do túnel de retransmissão híbrida. |
| | Armazenamento | O cluster do mecanismo do AKS implantado no Hub Azure Stack fornece um mecanismo escalonável e resiliente para executar o contêiner API de Detecção Facial.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar essa solução:

### <a name="scalability"></a>Escalabilidade

Esse padrão só permite mapeamentos de porta 1:1 no cliente e no servidor. Por exemplo, se a porta 80 for encapsulada para um serviço no ponto de extremidade do Azure, ela não poderá ser usada para outro serviço. Os mapeamentos de porta devem ser planejados de acordo. A retransmissão do Azure e as VMs devem ser adequadamente dimensionadas para lidar com o tráfego.

### <a name="availability"></a>Disponibilidade

Esses túneis e conexões não são redundantes. Para garantir a alta disponibilidade, talvez você queira implementar o código de verificação de erro. Outra opção é ter um pool de VMs conectadas à retransmissão do Azure atrás de um balanceador de carga.

### <a name="manageability"></a>Capacidade de gerenciamento

Essa solução pode abranger vários dispositivos e locais, o que pode ficar complicado. Os serviços de IoT do Azure podem automaticamente colocar novos locais e dispositivos online e mantê-los atualizados.

### <a name="security"></a>Segurança

Esse padrão, conforme mostrado, permite o acesso irrestrito a uma porta em um dispositivo interno da borda. Considere adicionar um mecanismo de autenticação ao serviço no dispositivo interno ou na frente do ponto de extremidade de retransmissão híbrida.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre os tópicos apresentados neste artigo:

- Esse padrão usa a retransmissão do Azure. Para obter mais informações, consulte a [documentação de retransmissão do Azure](/azure/azure-relay/).
- Consulte [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) para saber mais sobre as práticas recomendadas e obter respostas para outras perguntas.
- Consulte a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando você estiver pronto para testar o exemplo de solução, continue com o [Guia de implantação da solução de retransmissão híbrida](https://aka.ms/hybridrelaydeployment). O guia de implantação fornece instruções passo a passo para implantar e testar seus componentes.