---
title: Considerações de design de aplicativo híbrido no Azure e no Hub de Azure Stack
description: Saiba mais sobre considerações de design ao criar um aplicativo híbrido para a nuvem inteligente e a ponta inteligente, incluindo posicionamento, escalabilidade, disponibilidade e resiliência.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909818"
---
# <a name="hybrid-app-design-considerations"></a>Considerações de design de aplicativo híbrido

Microsoft Azure é a única nuvem híbrida consistente. Ele permite que você reutilize seus investimentos em desenvolvimento e habilite os aplicativos que podem abranger o Azure global, as nuvens do Azure soberanas e o Azure Stack, que é uma extensão do Azure em seu datacenter. Aplicativos que abrangem nuvens também são chamados de *aplicativos híbridos*.

O [*Guia de arquitetura do aplicativo Azure*](https://docs.microsoft.com/azure/architecture/guide) descreve uma abordagem estruturada para a criação de aplicativos escalonáveis, resilientes e altamente disponíveis. As considerações descritas no [*Guia de arquitetura de aplicativo Azure*](https://docs.microsoft.com/azure/architecture/guide) aplicam-se igualmente a aplicativos projetados para uma única nuvem e para aplicativos que abrangem nuvens.

Este artigo aumenta os [*pilares da qualidade de software*](https://docs.microsoft.com/azure/architecture/guide/pillars) abordada no [ *Guia de arquitetura*](https://docs.microsoft.com/azure/architecture/guide/) de [*aplicativo Azure*](https://docs.microsoft.com/azure/architecture/guide/) , concentrando-se especificamente na criação de aplicativos híbridos. Além disso, adicionamos um pilar de *posicionamento* , pois aplicativos híbridos não são exclusivos para uma nuvem ou um datacenter local.

Os cenários híbridos variam muito com os recursos que estão disponíveis para desenvolvimento e abrangem considerações como geografia, segurança, acesso à Internet e outras considerações. Embora este guia não possa enumerar suas considerações específicas, ele pode fornecer algumas diretrizes importantes e práticas recomendadas a serem seguidas. Projetar, configurar, implantar e manter com êxito uma arquitetura de aplicativo híbrido envolve muitas considerações de design que podem não ser inerentes a você.

Este documento tem como objetivo agregar as possíveis perguntas que podem surgir durante a implementação de aplicativos híbridos e fornece considerações (esses pilares) e práticas recomendadas para trabalhar com eles. Ao abordar essas perguntas durante a fase de design, você evitará os problemas que poderiam causar na produção.

Basicamente, essas são as perguntas que você precisa pensar antes de criar um aplicativo híbrido. Para começar, você precisa fazer o seguinte:

- Identifique e avalie os componentes do aplicativo.
- Avalie os componentes do aplicativo em relação aos pilares.

## <a name="evaluate-the-app-components"></a>Avaliar os componentes do aplicativo

Cada componente de um aplicativo tem sua própria função específica dentro do aplicativo maior e deve ser revisado com todas as considerações de design. Os requisitos e recursos de cada componente devem ser mapeados para essas considerações para ajudar a determinar a arquitetura do aplicativo.

Decompor seu aplicativo em seus componentes estudando a arquitetura do aplicativo e determinando o que ele consiste. Os componentes também podem incluir outros aplicativos com os quais seu aplicativo interage. Ao identificar os componentes, avalie as operações híbridas pretendidas de acordo com suas características, fazendo estas perguntas:

- Qual é a finalidade do componente?
- Quais são as interdependências entre os componentes?

Por exemplo, um aplicativo pode ter um front-end e um back-end definidos como dois componentes. Em um cenário híbrido, o front-end está em uma nuvem e o back-end está no outro. O aplicativo fornece canais de comunicação entre o front-end e o usuário e também entre o front-end e o back-end.

Um componente de aplicativo é definido por muitos formulários e cenários. A tarefa mais importante é identificá-los e sua localização na nuvem ou local.

Os componentes comuns do aplicativo a serem incluídos no seu inventário são listados na tabela 1.

### <a name="table-1-common-app-components"></a>Tabela 1. Componentes comuns do aplicativo

| **Componente** | **Diretrizes do aplicativo híbrido** |
| ---- | ---- |
| Conexões de cliente | Seu aplicativo (em qualquer dispositivo) pode acessar usuários de várias maneiras, de um ponto de entrada única, incluindo as seguintes maneiras:<br>-Um modelo de cliente-servidor que exige que o usuário tenha um cliente do instalado para trabalhar com o aplicativo. Um aplicativo baseado em servidor que é acessado em um navegador.<br>-As conexões de cliente podem incluir notificações quando a conexão é interrompida ou alerta quando os encargos de roaming podem ser aplicados. |
| Autenticação  | A autenticação pode ser necessária para um usuário que se conecta ao aplicativo ou de um componente que se conecta a outro. |
| APIs  | Você pode fornecer aos desenvolvedores acesso programático ao seu aplicativo com conjuntos de API e bibliotecas de classes e fornecer uma interface de conexão baseada em padrões da Internet. Você também pode usar APIs para decompor um aplicativo em unidades lógicas operacionais independentes. |
| Serviços  | Você pode empregar serviços sucintos para fornecer os recursos para um aplicativo. Um serviço pode ser o mecanismo no qual o aplicativo é executado. |
| Filas | Você pode usar filas para organizar o status dos ciclos de vida e dos Estados dos componentes do seu aplicativo. Essas filas podem fornecer mensagens, notificações e recursos de buffer para as partes assinantes. |
| Armazenamento de dados | Um aplicativo pode ser sem estado ou com estado. Aplicativos com estado precisam de armazenamento de dados que podem ser atendidos por vários formatos e volumes. |
| Armazenamento de dados em cache  | Um componente de cache de dados em seu design pode resolver problemas de latência de maneira estratégica e desempenhar uma função no disparo da intermitência de nuvem. |
| Ingestão de dados | Os dados podem ser enviados para um aplicativo de várias maneiras, variando de valores enviados pelo usuário em um formulário da Web para o fluxo de dados de alto volume continuamente. |
| Processamento de dados | Suas tarefas de processamento de dados (como relatórios, análises, exportações em lote e transformação de dados) podem ser processadas na origem ou descarregadas em um componente separado usando uma cópia dos dados. |

## <a name="assess-app-components-for-pillars"></a>Avaliar os componentes do aplicativo para os pilares

Para cada componente, avalie suas características para cada pilar. Conforme você avalia cada componente com todos os pilares, as perguntas que você pode não ter considerado podem ser conhecidas por você que afetam o design do aplicativo híbrido. Agir nessas considerações pode agregar valor na otimização de seu aplicativo. A tabela 2 fornece uma descrição de cada pilar à medida que ele se relaciona com aplicativos híbridos.

### <a name="table-2-pillars"></a>Tabela 2. Pilares

| **Pilar** | **Descrição** |
| ----------- | --------------------------------------------------------- |
| Posicionamento  | O posicionamento estratégico de componentes em aplicativos híbridos. |
| Escalabilidade  | A capacidade de um sistema para lidar com aumentos de carga. |
| Disponibilidade  | A proporção de tempo em que um aplicativo híbrido está funcional e funcionando. |
| Resiliência | A capacidade de recuperação de um aplicativo híbrido. |
| Capacidade de gerenciamento | Processos de operações que mantêm um sistema em execução em produção. |
| Segurança | Proteção de aplicativos e dados híbridos contra ameaças. |

## <a name="placement"></a>Posicionamento

Um aplicativo híbrido tem, inerentemente, uma consideração de posicionamento, como para o datacenter.

O posicionamento é a tarefa importante de posicionar componentes para que eles possam atender melhor um aplicativo híbrido. Por definição, os aplicativos híbridos abrangem locais, como do local para a nuvem e entre diferentes nuvens. Você pode posicionar os componentes do aplicativo nas nuvens de duas maneiras:

- **Aplicativos híbridos verticais**  
    Os componentes do aplicativo são distribuídos entre locais. Cada componente individual pode ter várias instâncias localizadas apenas em um único local.

- **Aplicativos híbridos horizontais**  
    Os componentes do aplicativo são distribuídos entre locais. Cada componente individual pode ter várias instâncias abrangendo vários locais.

    Alguns componentes podem estar cientes de sua localização enquanto outros não têm nenhum conhecimento de seu local e posicionamento. Esse virtuousness pode ser obtido com uma camada de abstração. Essa camada, com uma estrutura de aplicativo moderna, como os microservices, pode definir como o aplicativo é atendido pelo posicionamento dos componentes do aplicativo operando em nós entre nuvens.

### <a name="placement-checklist"></a>Lista de verificação de posicionamento

**Verifique os locais necessários.** Verifique se o aplicativo ou qualquer um de seus componentes é necessário para operar ou exigir certificação para uma nuvem específica. Isso pode incluir requisitos da soberania de sua empresa ou ditados por lei. Além disso, determine se qualquer operação local é necessária para um local ou localidade específica.

**Verificar dependências de conectividade.** Os locais necessários e outros fatores podem determinar as dependências de conectividade entre seus componentes. Ao posicionar os componentes, determine a conectividade e a segurança ideais para a comunicação entre eles. As opções incluem [ *VPN*,](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) e [ *conexões híbridas*.](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Avalie os recursos da plataforma.** Para cada componente de aplicativo, veja se o provedor de recursos necessário para o componente de aplicativo está disponível na nuvem e se a largura de banda pode acomodar os requisitos esperados de taxa de transferência e latência.

**Planeje a portabilidade.** Use estruturas de aplicativo modernas, como contêineres ou microservices, para planejar operações de movimentação e para evitar dependências de serviço.

**Determinar os requisitos da soberania de dados.** Os aplicativos híbridos são direcionados para acomodar o isolamento de dados, como em um datacenter local. Examine o posicionamento de seus recursos para otimizar o sucesso para acomodar esse requisito.

**Planejar a latência.** As operações entre nuvem podem introduzir a distância física entre os componentes do aplicativo. Verificar os requisitos para acomodar qualquer latência.

**Controlar fluxos de tráfego.** Lide com o uso de pico e as comunicações apropriadas e seguras para dados de informações de identificação pessoal quando acessados pelo front-end em uma nuvem pública.

## <a name="scalability"></a>Escalabilidade

A escalabilidade é a capacidade de um sistema de lidar com o aumento da carga em um aplicativo, que pode variar ao longo do tempo, pois outros fatores e forças afetam o tamanho do público, além do tamanho e do escopo do aplicativo.

Para a principal discussão sobre esse pilar, consulte [*escalabilidade*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) nos cinco pilares da excelência em arquitetura.

Uma abordagem de dimensionamento horizontal para aplicativos híbridos permite adicionar mais instâncias para atender à demanda e, em seguida, desabilitá-las durante períodos mais silenciosos.

Em cenários híbridos, a expansão de componentes individuais exige consideração adicional quando os componentes são distribuídos entre nuvens. O dimensionamento de uma parte do aplicativo pode exigir o dimensionamento de outro. Por exemplo, se o número de conexões de cliente aumentar, mas os serviços Web do aplicativo não forem dimensionados adequadamente, a carga no banco de dados poderá saturar o aplicativo.

Alguns componentes de aplicativo podem ser expandidos linearmente, enquanto outros têm dependências de dimensionamento e podem ser limitados a qual extensão eles são capazes de dimensionar. Por exemplo, um túnel VPN que fornece conectividade híbrida para os locais dos componentes do aplicativo tem um limite para a largura de banda e a latência em que ele pode ser dimensionado. Como os componentes do aplicativo são dimensionados para garantir que esses requisitos sejam atendidos?

### <a name="scalability-checklist"></a>Lista de verificação de escalabilidade

**Verificar limites de dimensionamento.** Para lidar com as várias dependências em seu aplicativo, determine a extensão até a qual os componentes de aplicativo em diferentes nuvens podem ser dimensionados independentemente um do outro, enquanto ainda atendem aos requisitos para executar o aplicativo. Aplicativos híbridos geralmente precisam dimensionar áreas específicas no aplicativo para lidar com um recurso enquanto ele interage e afeta o restante do aplicativo. Por exemplo, exceder um número de instâncias de front-end pode exigir o dimensionamento do back-end.

**Definir agendamentos de escala.** A maioria dos aplicativos tem períodos ocupados, portanto, você precisa agregar seus horários de pico em agendas para coordenar o dimensionamento ideal.

**Use um sistema de monitoramento centralizado.** Os recursos de monitoramento de plataforma podem fornecer dimensionamento automático, mas aplicativos híbridos precisam de um sistema de monitoramento centralizado que agrega a integridade do sistema e a carga. Um sistema de monitoramento centralizado pode iniciar o dimensionamento de um recurso em um local e o dimensionamento, dependendo do recurso em outro local. Além disso, um sistema de monitoramento central pode acompanhar quais nuvens dimensionamento automático de recursos e quais nuvens não têm.

**Aproveite os recursos de dimensionamento automático (como disponível).** Se os recursos de dimensionamento automático fizerem parte de sua arquitetura, você implementará o dimensionamento automático definindo limites que definem quando um componente de aplicativo precisa ser escalado verticalmente, horizontalmente ou horizontalmente. Um exemplo de dimensionamento automático é uma conexão de cliente que é dimensionada automaticamente em uma nuvem para lidar com a capacidade aumentada, mas faz com que outras dependências do aplicativo, espalhadas por diferentes nuvens, também sejam dimensionadas. Os recursos de dimensionamento automático desses componentes dependentes devem ser determinados.

Se o dimensionamento automático não estiver disponível, considere implementar scripts e outros recursos para acomodar o dimensionamento manual, disparado por limites no sistema de monitoramento centralizado.

**Determinar a carga esperada por local.** Aplicativos híbridos que lidam com solicitações de clientes podem depender principalmente de um único local. Quando a carga de solicitações do cliente excede um limite, recursos adicionais podem ser adicionados em um local diferente para distribuir a carga de solicitações de entrada. Certifique-se de que as conexões de cliente possam lidar com cargas aumentadas e também determinar quaisquer procedimentos automatizados para as conexões de cliente para lidar com a carga.

## <a name="availability"></a>Disponibilidade

A disponibilidade é o momento em que um sistema está funcionando e funcionando. A disponibilidade é medida como uma porcentagem de tempo de atividade. Erros de aplicativo, problemas de infraestrutura e carga do sistema podem reduzir a disponibilidade.

Para a principal discussão sobre esse pilar, confira [*disponibilidade*](/azure/architecture/framework/) nos cinco pilares da excelência em arquitetura.

### <a name="availability-checklist"></a>Lista de verificação de disponibilidade

**Forneça redundância para conectividade.** Aplicativos híbridos exigem conectividade entre as nuvens em que o aplicativo está espalhado. Você tem uma opção de tecnologias para conectividade híbrida, assim, além da sua principal escolha de tecnologia, use outra tecnologia para fornecer redundância com recursos de failover automatizados caso a tecnologia principal falhe.

**Classificar domínios de falha.** Aplicativos tolerantes a falhas exigem vários domínios de falha. Os domínios de falha ajudam a isolar o ponto de falha, como se um único disco rígido falhar localmente, se um comutador Top-of-rack ficar inativo ou se o datacenter completo estiver indisponível. Em um aplicativo híbrido, um local pode ser classificado como um domínio de falha. Com mais requisitos de disponibilidade, mais você precisa avaliar como um único domínio de falha deve ser classificado.

**Classificar domínios de atualização.** Os domínios de atualização são usados para garantir que as instâncias de componentes do aplicativo estejam disponíveis, enquanto outras instâncias do mesmo componente estão sendo atendidas com atualizações ou atualizações de recursos. Assim como ocorre com os domínios de falha, os domínios de atualização podem ser classificados por seu posicionamento entre locais. Você deve determinar se um componente de aplicativo pode acomodar a atualização em um local antes de ser atualizado em outro local ou se outras configurações de domínio forem necessárias. Um único local pode ter vários domínios de atualização.

**Rastreie instâncias e disponibilidade.** Os componentes de aplicativos altamente disponíveis podem estar disponíveis por meio de balanceamento de carga e replicação de dados síncrona. Você deve determinar quantas instâncias podem ficar offline antes de o serviço ser interrompido.

**Implemente auto-recuperação.** No caso de um problema causar uma interrupção para a disponibilidade do aplicativo, uma detecção por um sistema de monitoramento poderia iniciar as atividades de auto-recuperação para o aplicativo, como descarregar a instância com falha e reimplantá-la. É muito provável que isso exija uma solução de monitoramento central, integrada com uma integração contínua híbrida e um pipeline de CI/CD (entrega contínua). O aplicativo é integrado a um sistema de monitoramento para identificar problemas que podem exigir a reimplantação de um componente de aplicativo. O sistema de monitoramento também pode disparar CI/CD híbrido para reimplantar o componente do aplicativo e potencialmente quaisquer outros componentes dependentes no mesmo ou em outros locais.

**Mantenha SLAs (contratos de nível de serviço).** A disponibilidade é essencial para qualquer contrato para manter a conectividade com os serviços e aplicativos que você tem com seus clientes. Cada local com o qual seu aplicativo híbrido depende pode ter seu próprio SLA. Esses diferentes SLAs podem afetar o SLA geral de seu aplicativo híbrido.

## <a name="resiliency"></a>Resiliência

Resiliência é a capacidade de um aplicativo híbrido e um sistema de se recuperar de falhas e continuar a funcionar. O objetivo da resiliência é retornar o aplicativo para um estado totalmente funcional após a ocorrência de uma falha. As estratégias de resiliência incluem soluções como backup, replicação e recuperação de desastres.

Para a principal discussão sobre esse pilar, confira [*resiliência*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) nos cinco pilares da excelência em arquitetura.

### <a name="resiliency-checklist"></a>Lista de verificação de resiliência

**Descubra dependências de recuperação de desastre.** A recuperação de desastres em uma nuvem pode exigir alterações nos componentes do aplicativo em outra nuvem. Se um ou vários componentes de uma nuvem passarem por failover para outro local, na mesma nuvem ou em outra nuvem, os componentes dependentes precisarão estar cientes dessas alterações. Isso também inclui as dependências de conectividade. A resiliência requer um plano de recuperação de aplicativo totalmente testado para cada nuvem.

**Estabeleça o fluxo de recuperação.** Um design de fluxo de recuperação eficaz avaliou componentes de aplicativo para sua capacidade de acomodar buffers, novas tentativas, repetir a transferência de dados com falha e, se necessário, retornar a um serviço ou fluxo de trabalho diferente. Você deve determinar o mecanismo de backup a ser usado, o que o procedimento de restauração envolve e com que frequência ele é testado. Você também deve determinar a frequência para backups incrementais e completos.

**Teste recuperações parciais.** Uma recuperação parcial para parte do aplicativo pode fornecer a garantia para os usuários que não estão disponíveis. Essa parte do plano deve garantir que uma restauração parcial não tenha nenhum efeito colateral, como um serviço de backup e restauração que interage com o aplicativo para desligá-lo normalmente antes que o backup seja feito.

**Determine os instigantes de recuperação de desastre e atribua a responsabilidade.** Um plano de recuperação deve descrever quem e quais funções podem iniciar ações de backup e recuperação além do que pode ser feito backup e restaurado.

**Compare os limites de auto-recuperação com a recuperação de desastre.** Determine os recursos de auto-recuperação de um aplicativo para o início automático da recuperação e o tempo necessário para que a auto-recuperação de um aplicativo seja considerada uma falha ou êxito. Determine os limites para cada nuvem.

**Verifique a disponibilidade dos recursos de resiliência.** Determine a disponibilidade de recursos de resiliência e recursos para cada local. Se um local não fornecer os recursos necessários, considere a integração desse local em um serviço centralizado que fornece os recursos de resiliência.

**Determinar os tempos de inatividade.** Determine o tempo de inatividade esperado devido à manutenção do aplicativo como um todo e como componentes do aplicativo.

**Documentar procedimentos de solução de problemas.** Defina procedimentos de solução de problemas para reimplantação de recursos e componentes de aplicativo.

## <a name="manageability"></a>Capacidade de gerenciamento

As considerações sobre como gerenciar seus aplicativos híbridos são essenciais para projetar sua arquitetura. Um aplicativo híbrido bem gerenciado fornece uma infraestrutura como código que permite a integração do código do aplicativo consistente em um pipeline de desenvolvimento comum. Ao implementar testes consistentes do sistema e individuais de alterações na infraestrutura, você pode garantir uma implantação integrada se as alterações passarem pelos testes, permitindo que eles sejam mesclados no código-fonte.

Para a principal discussão sobre esse pilar, confira [*DevOps*](/azure/architecture/framework/#devops) nos cinco pilares da excelência em arquitetura.

### <a name="manageability-checklist"></a>Lista de verificação da capacidade de gerenciamento

**Implemente o monitoramento.** Use um sistema de monitoramento centralizado de componentes de aplicativos espalhados por nuvens para fornecer uma visão agregada de sua integridade e desempenho. Esse sistema inclui o monitoramento dos componentes do aplicativo e dos recursos de plataforma relacionados.

Determine as partes do aplicativo que exigem monitoramento.

**Coordenar políticas.** Cada local que um aplicativo híbrido abrange pode ter sua própria política que abrange tipos de recursos permitidos, convenções de nomenclatura, marcas e outros critérios.

**Definir e usar funções.** Como administrador de banco de dados, você precisa determinar as permissões necessárias para pessoas diferentes (como um proprietário do aplicativo, um administrador de banco de dados e um usuário final) que precisam acessar recursos do aplicativo. Essas permissões precisam ser configuradas nos recursos e dentro do aplicativo. Um sistema RBAC (controle de acesso baseado em função) permite que você defina essas permissões nos recursos do aplicativo. Esses direitos de acesso são desafiadores quando todos os recursos são implantados em uma única nuvem, mas exigem ainda mais atenção quando os recursos são distribuídos entre nuvens. As permissões nos recursos definidos em uma nuvem não se aplicam aos recursos definidos em outra nuvem.

**Use pipelines de CI/CD.** Um pipeline de CI/CD (integração contínua e desenvolvimento contínuo) pode fornecer um processo consistente para a criação e implantação de aplicativos que se estendem por nuvens e para fornecer garantia de qualidade para sua infraestrutura e aplicativo. Esse pipeline permite que a infraestrutura e o aplicativo sejam testados em uma nuvem e implantados em outra nuvem. O pipeline até mesmo permite implantar determinados componentes do seu aplicativo híbrido em uma nuvem e outros componentes em outra nuvem, basicamente formando a base para a implantação de aplicativos híbridos. Um sistema de CI/CD é essencial para lidar com as dependências que os componentes do aplicativo têm para o outro durante a instalação, como o aplicativo Web que precisa de uma cadeia de conexão para o banco de dados.

**Gerencie o ciclo de vida.** Como os recursos de um aplicativo híbrido podem abranger locais, cada recurso de gerenciamento do ciclo de vida de cada local precisa ser agregado em uma única unidade de gerenciamento do ciclo de vida. Considere como eles são criados, atualizados e excluídos.

**Examine as estratégias de solução de problemas.** A solução de problemas de um aplicativo híbrido envolve mais componentes de aplicativo do que o mesmo aplicativo em execução em uma única nuvem. Além da conectividade entre as nuvens, o aplicativo é executado em duas plataformas em vez de uma. Uma tarefa importante na solução de problemas de aplicativos híbridos é examinar a integridade agregada e o monitoramento de desempenho dos componentes do aplicativo.

## <a name="security"></a>Segurança

A segurança é uma das principais considerações para qualquer aplicativo de nuvem e torna-se ainda mais importante para aplicativos de nuvem híbrida.

Para a principal discussão sobre esse pilar, confira [*segurança*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) nos cinco pilares da excelência em arquitetura.

### <a name="security-checklist"></a>Lista de verificação de segurança

**Assuma a violação.** Se uma parte do aplicativo for comprometida, verifique se há soluções em vigor para minimizar a disseminação da violação, não apenas dentro do mesmo local, mas também entre locais.

**Monitore o acesso à rede permitido.** Determine as políticas de acesso à rede para o aplicativo, como acessar somente o aplicativo de uma sub-rede específica e permitir apenas as portas e protocolos mínimos entre os componentes necessários para que o aplicativo funcione corretamente.

**Empregue autenticação robusta.** Um esquema de autenticação robusto é essencial para a segurança do seu aplicativo. Considere o uso de um provedor de identidade federado que fornece recursos de logon único e emprega um ou mais dos seguintes esquemas: logon de nome de usuário e senha, chaves públicas e privadas, autenticação de dois fatores ou multifator e grupos de segurança confiáveis. Determine os recursos apropriados para armazenar dados confidenciais e outros segredos para autenticação de aplicativo, além de tipos de certificado e seus requisitos.

**Usar criptografia.** Identifique quais áreas do aplicativo usam criptografia, como para armazenamento de dados ou comunicação do cliente e acesso.

**Use canais seguros.** Um canal seguro entre as nuvens é essencial para fornecer verificações de segurança e autenticação, proteção em tempo real, quarentena e outros serviços entre nuvens.

**Definir e usar funções.** Implemente funções para configurações de recursos e acesso de identidade única entre nuvens. Determine os requisitos de RBAC (controle de acesso baseado em função) para o aplicativo e seus recursos de plataforma.

**Auditar seu sistema.** O monitoramento do sistema pode registrar e agregar dados dos componentes do aplicativo e das operações de plataforma de nuvem relacionadas.

## <a name="summary"></a>Resumo

Este artigo fornece uma lista de verificação dos itens que são importantes para serem considerados durante a criação e o design de seus aplicativos híbridos. Revisar esses pilares antes de implantar seu aplicativo impede que você execute essas perguntas em interrupções de produção e potencialmente exigindo que você revisite o design.

Pode parecer uma tarefa demorada com antecedência, mas você obtém facilmente seu retorno sobre o investimento se projetar seu aplicativo com base nesses pilares.

## <a name="next-steps"></a>Próximas etapas

Para saber mais, consulte os recursos a seguir:

- [Nuvem híbrida](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Aplicativos de nuvem híbrida](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Desenvolva modelos do Azure Resource Manager para consistência de nuvem](https://aka.ms/consistency)
