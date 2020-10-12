---
title: Considerações de design de aplicativo híbrido no Azure e no Azure Stack Hub
description: Saiba mais sobre as considerações de design ao criar um aplicativo híbrido para a nuvem inteligente e a borda inteligente, incluindo posicionamento, escalabilidade, disponibilidade e resiliência.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8b975c7b99807490d446f557e84b6e0eabf34649
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852483"
---
# <a name="hybrid-app-design-considerations"></a>Considerações de design de aplicativo híbrido

O Microsoft Azure é a única nuvem híbrida consistente. Ele permite que você reutilize seus investimentos em desenvolvimento e habilite aplicativos que englobem o Azure global, as nuvens soberanas do Azure, e o Azure Stack, que é uma extensão do Azure em seu datacenter. Os aplicativos que abrangem nuvens também são chamados de *aplicativos híbridos*.

O [*Guia de arquitetura do Aplicativo Azure*](/azure/architecture/guide) descreve uma abordagem estruturada para o design de aplicativos que sejam escalonáveis, resilientes e altamente disponíveis. Da mesma forma, as considerações descritas no [*Guia de arquitetura do Aplicativo Azure*](/azure/architecture/guide) se aplicam a aplicativos projetados para uma única nuvem e para aplicativos que englobem nuvens.

Este artigo aumenta os [*Pilares de qualidade de software*](/azure/architecture/guide/pillars) discutidos no [*Guia de arquitetura do*](/azure/architecture/guide/) [*Aplicativo Azure*,](/azure/architecture/guide/) com foco específico no design de aplicativos híbridos. Além disso, adicionamos um pilar de *posicionamento* já que os aplicativos híbridos não são exclusivos de uma nuvem ou de um datacenter local.

Os cenários híbridos variam muito de acordo com os recursos que estão disponíveis para o desenvolvimento e abrangem considerações como geografia, segurança, acesso à Internet, entre outras. Esse guia não descreverá suas considerações específicas, mas pode fornecer algumas diretrizes importantes e melhores práticas para você seguir. Projetar, configurar, implantar e fazer a manutenção de uma arquitetura de aplicativo híbrido com êxito envolve muitas considerações de design que podem não ser inerentemente conhecidas por você.

Este documento tem como objetivo agregar as perguntas que podem surgir durante a implementação de aplicativos híbridos e fornece considerações (esses pilares) e melhores práticas para o trabalho com eles. Ao abordar essas dúvidas durante a fase de design, você evitará os problemas que elas poderiam causar na produção.

Basicamente, essas são perguntas que você deve levar em consideração antes de criar um aplicativo híbrido. Para começar, você precisa fazer o seguinte:

- Identificar e avaliar os componentes do aplicativo.
- Avaliar os componentes do aplicativo em relação aos pilares.

## <a name="evaluate-the-app-components"></a>Avaliar os componentes do aplicativo

Cada componente de um aplicativo tem sua função específica dentro do aplicativo maior e deve ser revisado com todas as considerações de design. Os requisitos e recursos de cada componente devem ser mapeados em relação a essas considerações para ajudar a determinar a arquitetura do aplicativo.

Decomponha seu aplicativo conforme seus componentes estudando a arquitetura do aplicativo e determinando em que ele consiste. Os componentes também podem ser outros aplicativos com os quais seu aplicativo interage. Conforme os componentes forem sendo identificados, avalie as operações híbridas pretendidas de acordo com suas características por meio destas perguntas:

- Qual é o objetivo do componente?
- Quais são as interdependências entre os componentes?

Por exemplo, um aplicativo pode ter um front-end e um back-end definidos como dois componentes. Em um cenário híbrido, o front-end fica em uma nuvem e o back-end em outra. O aplicativo fornece canais de comunicação entre o front-end e o usuário, além de entre o front-end e o back-end.

Um componente de aplicativo é definido por vários formulários e cenários. A tarefa mais importante é que eles e suas localizações sejam identificados na nuvem ou no local.

Os componentes comuns de aplicativo a serem incluídos no seu inventário estão listados na Tabela 1.

### <a name="table-1-common-app-components"></a>Tabela 1. Componentes comuns do aplicativo

| **Componente** | **Diretrizes do aplicativo híbrido** |
| ---- | ---- |
| Conexões de cliente | Seu aplicativo (em qualquer dispositivo) pode acessar usuários de diversas maneiras a partir de um ponto de entrada única, incluindo as seguintes:<br>–   Um modelo cliente-servidor que exija que o usuário tenha um cliente instalado para trabalhar com o aplicativo. Um aplicativo baseado em servidor que seja acessado usando um navegador.<br>–   As conexões de cliente podem incluir notificações quando a conexão for interrompida ou alertas quando houver a possibilidade de encargos de roaming serem aplicados. |
| Autenticação  | A autenticação pode ser exigida a um usuário que estiver se conectando com o aplicativo ou de um componente se conectando a outro. |
| APIs  | Você pode fornecer aos desenvolvedores o acesso programático ao seu aplicativo com conjuntos de API e bibliotecas de classes, além de fornecer uma interface de conexão baseada em padrões da Internet. Também é possível usar APIs para decompor um aplicativo em unidades lógicas operacionais independentes. |
| Serviços  | Você pode empregar serviços sucintos para fornecer os recursos a um aplicativo. Um serviço pode ser o mecanismo em que o aplicativo é executado. |
| Filas | Você pode usar filas para organizar o status dos ciclos de vida e dos estados dos componentes do seu aplicativo. Essas filas podem fornecer recursos de mensagens, notificações e buffer para as partes assinantes. |
| Armazenamento de dados | Um aplicativo pode ser sem estado ou com estado. Aplicativos com estado precisam de armazenamento de dados, o que pode ser obtido por vários formatos e volumes. |
| Armazenamento de dados em cache  | Ter um componente de armazenamento de dados em cache em seu design pode resolver problemas de latência de maneira estratégica e desempenhar um papel no acionamento da intermitência de nuvem. |
| Ingestão de dados | Os dados podem ser enviados para um aplicativo de várias maneiras, desde via valores enviados pelo usuário em um formulário da Web até via fluxo de dados contínuo de alto volume. |
| Processamento de dados | Suas tarefas de processamento de dados (como relatórios, análises, exportações em lote e transformação de dados) podem tanto ser processadas na origem quanto descarregadas em um componente separado usando uma cópia dos dados. |

## <a name="assess-app-components-for-pillars"></a>Avaliar os componentes do aplicativo conforme os pilares

Para cada componente, avalie suas características conforme cada pilar. À medida que você avaliar cada componente conforme todos os pilares, talvez perceba que as perguntas que não levou em consideração afetem o design do aplicativo híbrido. Tomar ações frente a essas considerações pode agregar valor na otimização de seu aplicativo. A Tabela 2 fornece uma descrição de cada pilar no que diz respeito a aplicativos híbridos.

### <a name="table-2-pillars"></a>Tabela 2. Pilares

| **Pilar** | **Descrição** |
| ----------- | --------------------------------------------------------- |
| Posicionamento  | O posicionamento estratégico dos componentes em aplicativos híbridos. |
| Escalabilidade  | A capacidade de um sistema para lidar com aumentos de carga. |
| Disponibilidade  | A proporção de tempo que um aplicativo híbrido está funcional e operando. |
| Resiliência | A capacidade de recuperação de um aplicativo híbrido. |
| Capacidade de gerenciamento | Processos de operações que mantêm um sistema em execução em produção. |
| Segurança | Proteger aplicativos híbridos e dados contra ameaças. |

## <a name="placement"></a>Posicionamento

Inerentemente, um aplicativo híbrido tem uma consideração de posicionamento; por exemplo, para o datacenter.

O posicionamento é a importante tarefa de posicionar os componentes de maneira que eles possam atender melhor a um aplicativo híbrido. Por definição, os aplicativos híbridos abrangem localizações, como do local para a nuvem e entre diferentes nuvens. É possível posicionar os componentes do aplicativo nas nuvens de duas maneiras:

- **Aplicativos híbridos verticais**  
    Os componentes do aplicativo são distribuídos entre locais. Cada componente individual pode ter múltiplas instâncias situadas em um único local.

- **Aplicativos híbridos horizontais**  
    Os componentes do aplicativo são distribuídos entre locais. Cada componente individual pode ter múltiplas instâncias abrangendo vários locais.

    Alguns componentes podem estar cientes de sua localização, já outros podem não ter nenhum conhecimento de seu local e posicionamento. Essa virtuosidade pode ser obtida com uma camada de abstração. Essa camada, com uma estrutura de aplicativo moderna como os microsserviços, pode definir como o aplicativo será atendido pelo posicionamento de seus componentes operando em nós entre nuvens.

### <a name="placement-checklist"></a>Lista de verificação do posicionamento

**Verificar os locais necessários.** Verifique se o aplicativo ou algum de seus componentes é necessário para a operação em, ou exigem certificação para, uma nuvem específica. Isso pode incluir requisitos de soberania da sua empresa ou exigidos por lei. Além disso, determine se alguma operação local é necessária para um determinada localização ou localidade.

**Verificar dependências de conectividade.** Os locais requeridos e outros fatores podem determinar as dependências de conectividade entre seus componentes. Quando estiver posicionando os componentes, determine a conectividade e a segurança ideais para a comunicação entre eles. Entre as opções temos [*VPN*,](/azure/vpn-gateway/) [*ExpressRoute*](/azure/expressroute/) e [*Conexões híbridas*.](/azure/app-service/app-service-hybrid-connections)

**Avaliar as funcionalidades da plataforma.** Para cada componente do aplicativo, veja se o provedor de recursos necessário relacionado está disponível na nuvem e se a largura de banda pode acomodar os requisitos esperados de taxa de transferência e latência.

**Planejar a portabilidade.** Use estruturas modernas de aplicativo, como contêineres ou microsserviços, para planejar operações de movimentação e para evitar dependências de serviço.

**Determinar os requisitos de soberania de dados.** Os aplicativos híbridos são concebidos para acomodar o isolamento de dados, como em um datacenter local. Revise o posicionamento de seus recursos para otimizar o sucesso e acomodar esse requisito.

**Planejar a latência.** As operações entre nuvens podem apresentar uma distância física entre os componentes do aplicativo. Determine os requisitos para acomodar qualquer latência.

**Controlar fluxos de tráfego.** Lide com o uso de pico e as comunicações apropriadas e seguras para dados de informações pessoais identificáveis quando acessados pelo front-end em uma nuvem pública.

## <a name="scalability"></a>Escalabilidade

A escalabilidade é a capacidade de um sistema de lidar com o aumento da carga em um aplicativo, o que pode variar ao longo do tempo, uma vez que outros fatores e forças afetarem o tamanho do público e o tamanho e o escopo do aplicativo.

Para a discussão central sobre esse pilar, confira [*Escalabilidade*](/azure/architecture/guide/pillars#scalability) nos cinco pilares da excelência em arquitetura.

Uma abordagem de dimensionamento horizontal para aplicativos híbridos permite adicionar mais instâncias para atender a demanda e, em períodos mais calmos, desabilitá-las.

Em cenários híbridos, a escala horizontal de componentes individuais requer atenção adicional quando os componentes são distribuídos entre nuvens. A colocação em escala de uma parte do aplicativo pode requerer a colocação em escala de outra. Por exemplo, se o número de conexões do cliente aumentar, mas os serviços Web do aplicativo não forem escalados horizontalmente de modo apropriado, a carga no banco de dados pode saturar o aplicativo.

Alguns componentes de aplicativo podem ser escalados horizontalmente de modo linear, enquanto outros têm dependências de dimensionamento e podem apresentar limitações em relação a até que ponto podem ser dimensionados. Por exemplo, um túnel VPN que fornece conectividade híbrida para os locais dos componentes do aplicativo tem um limite quanto à largura de banda e à latência para as quais ele pode ser dimensionado. Como os componentes do aplicativo são dimensionados para garantir que esses requisitos sejam atendidos?

### <a name="scalability-checklist"></a>Lista de verificação de escalabilidade

**Verificar limites de dimensionamento.** Para lidar com as várias dependências do seu aplicativo, determine até que ponto os componentes em diferentes nuvens podem ser colocados em escala de modo independente um do outro, enquanto ainda atendem aos requisitos de execução do aplicativo. Geralmente, aplicativos híbridos precisam dimensionar áreas específicas para lidar com um recurso enquanto ele interage com e afeta o restante do aplicativo. Por exemplo, para exceder um número de instâncias de front-end, talvez seja preciso colocar o back-end em escala.

**Definir agendamentos de escala.** A maioria dos aplicativos passa por períodos ocupados, portanto, você precisa incluir os horários de pico em agendas para coordenar a colocação em escala ideal.

**Usar um sistema de monitoramento centralizado.** Os recursos de monitoramento de plataforma podem fornecer o dimensionamento automático, mas aplicativos híbridos precisam de um sistema de monitoramento centralizado que agregue a integridade e a carga do sistema. Um sistema de monitoramento centralizado pode iniciar o dimensionamento de um recurso em um local e o dimensionamento dependendo do recurso em outro local. Além disso, um sistema de monitoramento central pode acompanhar quais nuvens fazem ou não fazem o dimensionamento automático de recursos.

**Aproveitar os recursos de dimensionamento automático (conforme disponível).** Se os recursos de dimensionamento automático fizerem parte de sua arquitetura, você implementará o dimensionamento automático configurando os limites que definem quando um componente de aplicativo precisa ser escalado verticalmente, horizontalmente ou reduzido. Um exemplo de dimensionamento automático seria uma conexão de cliente dimensionada automaticamente em uma nuvem para lidar com a capacidade aumentada, mas que fizesse com que outras dependências do aplicativo, espalhadas por diferentes nuvens, também fossem escaladas. Devem ser determinados os recursos de dimensionamento automático desses componentes dependentes.

Caso o dimensionamento automático não esteja disponível, cogite implementar scripts e outros recursos para acomodar o dimensionamento manual, acionado por limites no sistema de monitoramento centralizado.

**Determinar a carga esperada por local.** Aplicativos híbridos que lidam com solicitações de clientes podem depender primordialmente de um único local. Quando a carga de solicitações do cliente excede um limite, é possível adicionar outros recursos em um local diferente para distribuir a carga de solicitações de entrada. Verifique se as conexões de cliente podem lidar com as cargas aumentadas e também determinar quaisquer procedimentos automatizados para as conexões de cliente para lidar com a carga.

## <a name="availability"></a>Disponibilidade

A disponibilidade é o período em que um sistema está funcional e em operação. A disponibilidade é medida como um percentual do tempo de atividade. Erros de aplicativo, problemas de infraestrutura e carga do sistema podem reduzir a disponibilidade.

Para a discussão central sobre esse pilar, confira [*Disponibilidade*](/azure/architecture/framework/) nos cinco pilares da excelência em arquitetura.

### <a name="availability-checklist"></a>Lista de verificação de disponibilidade

**Fornecer redundância para a conectividade.** Os aplicativos híbridos exigem conectividade entre as nuvens pelas quais o aplicativo está espalhado. Você pode optar por tecnologias de conectividade híbrida, assim, além da sua principal escolha de tecnologia, poderá usar outra tecnologia para fornecer redundância com recursos de failover automatizados caso a tecnologia principal falhe.

**Classificar domínios de falha.** Aplicativos tolerantes a falhas requerem vários domínios de falha. Os domínios de falha ajudam a isolar o ponto de falha, como se um único disco rígido falhar localmente, se um comutador top-of-rack ficar inativo ou se todo o datacenter estiver indisponível. Em um aplicativo híbrido, um local pode ser classificado como domínio de falha. Quanto maiores forem os requisitos de disponibilidade, mais você precisará avaliar como um único domínio de falha deve ser classificado.

**Classificar domínios de atualização.** Os domínios de atualização são usados para garantir que as instâncias de componentes do aplicativo estejam disponíveis enquanto outras instâncias do mesmo componente estão sendo atendidas com atualizações ou upgrades de recursos. Assim como ocorre com os domínios de falha, os domínios de atualização podem ser classificados conforme seu posicionamento entre locais. Você deve determinar se um componente de aplicativo poderá ser atualizado em um local antes de ser atualizado em outro ou se outras configurações de domínio serão necessárias. Um único local pode ter, ele próprio, vários domínios de atualização.

**Acompanhar instâncias e disponibilidade.** Os componentes de aplicativos altamente disponíveis podem ficar disponíveis por meio de balanceamento de carga e replicação de dados síncrona. Você deve determinar quantas instâncias poderão ficar offline antes que o serviço seja interrompido.

**Implementar a autorrecuperação.** Caso um problema cause uma interrupção da disponibilidade do aplicativo, uma detecção por um sistema de monitoramento poderia iniciar as atividades de autorrecuperação do aplicativo, como descarregar a instância com falha e reimplantá-la. É muito provável que isso exija uma solução de monitoramento central, incorporada a uma Integração contínua híbrida e um pipeline de Entrega contínua (CI/CD). O aplicativo é integrado a um sistema de monitoramento para identificar problemas que poderiam exigir a reimplantação de um componente de aplicativo. O sistema de monitoramento também pode acionar um CI/CD híbrido para reimplantar o componente do aplicativo e, potencialmente, quaisquer outros componentes dependentes no mesmo ou em outros locais.

**Manter SLAs (Contratos de Nível de Serviço).** A disponibilidade é essencial para qualquer contrato, a fim de manter a conectividade com os serviços e aplicativos que você tem com seus clientes. Cada local de dependência do seu aplicativo híbrido pode ter seu próprio SLA. Esses diferentes SLAs podem impactar o SLA geral do seu aplicativo híbrido.

## <a name="resiliency"></a>Resiliência

A resiliência é a capacidade de um aplicativo híbrido e um sistema se recuperarem de falhas e continuarem funcionando. A meta da resiliência é retornar o aplicativo a um estado totalmente funcional após uma falha. Entre as estratégias de resiliência, há soluções de backup, replicação e recuperação de desastres.

Para a discussão central sobre esse pilar, confira [*Resiliência*](/azure/architecture/guide/pillars#resiliency) nos cinco pilares da excelência em arquitetura.

### <a name="resiliency-checklist"></a>Lista de verificação de resiliência

**Descobrir dependências de recuperação de desastre.** A recuperação de desastre em uma nuvem pode requerer alterações nos componentes do aplicativo em outra nuvem. Caso um ou vários componentes de uma nuvem passem por failover para outro local, seja na mesma ou em outra nuvem, os componentes dependentes precisarão estar cientes dessas alterações. Isso também inclui as dependências de conectividade. A resiliência requer um plano de recuperação de aplicativo para cada nuvem que tenha sido totalmente testado.

**Estabelecer o fluxo de recuperação.** Um design de fluxo de recuperação eficaz avaliou os componentes do aplicativo quanto à capacidade de acomodar buffers, novas tentativas, repetir a transferência de dados com falha e, se necessário, retornar a um serviço ou fluxo de trabalho diferente. Você deve determinar qual mecanismo de backup deve ser usado, o que o seu procedimento de restauração envolve e com que frequência ele será testado. Você também deve determinar a frequência dos backups incrementais e completos.

**Testar recuperações parciais.** Uma recuperação parcial de parte do aplicativo pode garantir aos usuários que nem tudo está indisponível. Essa parte do plano deve garantir que uma restauração parcial não tenha nenhum efeito colateral, como um serviço de backup e restauração que interaja com o aplicativo para desligá-lo antes que o backup seja feito.

**Determinar os instigadores de recuperação de desastre e atribuir a responsabilidade.** Um plano de recuperação deve descrever quem e quais funções podem iniciar ações de backup e recuperação, além de o que pode passar por backup e ser restaurado.

**Comparar os limites de autorrecuperação com a recuperação de desastre.** Determine as funcionalidades de autorrecuperação de um aplicativo para o início da recuperação automática e o tempo necessário para que a autorrecuperação de um aplicativo seja considerada como falha ou bem-sucedida. Determine os limites de cada nuvem.

**Verificar a disponibilidade dos recursos de resiliência.** Determine a disponibilidade dos recursos de resiliência e das funcionalidades para cada local. Caso um local não forneça as funcionalidades necessárias, cogite fazer a integração desse local em um serviço centralizado que forneça os recursos de resiliência.

**Determinar os tempos de inatividade.** Determine o tempo de inatividade esperado devido à manutenção do aplicativo como um todo e dos seus componentes.

**Documentar os procedimentos de solução de problemas.** Defina os procedimentos de solução de problemas para reimplantação de recursos e componentes de aplicativo.

## <a name="manageability"></a>Capacidade de gerenciamento

As considerações sobre como gerenciar os aplicativos híbridos são essenciais para projetar sua arquitetura. Um aplicativo híbrido bem gerenciado fornece uma infraestrutura como código, a qual permite a integração do código de aplicativo consistente em um pipeline de desenvolvimento comum. Ao implementar testes individuais das alterações na infraestrutura que sejam consistentes e abranjam todo o sistema, você conseguirá garantir uma implantação integrada caso as alterações passem nos testes, permitindo que sejam mescladas no código-fonte.

Para a discussão central sobre esse pilar, confira [*DevOps*](/azure/architecture/framework/#devops) nos cinco pilares da excelência em arquitetura.

### <a name="manageability-checklist"></a>Lista de verificação da capacidade de gerenciamento

**Implementar o monitoramento.** Use um sistema de monitoramento centralizado de componentes de aplicativos espalhados por nuvens para fornecer uma visão agregada da integridade e do desempenho desses componentes. Esse sistema inclui o monitoramento dos componentes do aplicativo e os recursos de plataforma relacionados.

Determine as partes do aplicativo que requerem monitoramento.

**Coordenar políticas.** Cada local abrangido por um aplicativo híbrido pode ter sua própria política envolvendo os tipos de recursos permitidos, convenções de nomenclatura, marcas e outros critérios.

**Definir e usar funções.** Por ser administrador de banco de dados, você precisa determinar as permissões necessárias para as diferentes pessoas (como um proprietário do aplicativo, um administrador de banco de dados e um usuário final) que precisarão acessar os recursos do aplicativo. Essas permissões precisam estar configuradas nos recursos e dentro do aplicativo. Um sistema RBAC (controle de acesso baseado em função) permite definir essas permissões nos recursos do aplicativo. Esses direitos de acesso são complicados quando todos os recursos estão implantados em uma única nuvem, mas requerem ainda mais atenção quando os recursos são distribuídos entre nuvens. As permissões dos recursos definidos em uma nuvem não se aplicam aos definidos em outra nuvem.

**Usar pipelines de CI/CD.** Um pipeline de CI/CD (Integração contínua e desenvolvimento contínuo) pode fornecer um processo consistente para a criação e implantação de aplicativos que se estendam entre nuvens e para fornecer uma garantia de qualidade para a infraestrutura e o aplicativo. Esse pipeline permite que a infraestrutura e o aplicativo sejam testados em uma nuvem e implantados em outra. O pipeline permite até mesmo implantar determinados componentes do seu aplicativo híbrido em uma nuvem e outros componentes em outra, formando, basicamente, a base para a implantação de aplicativos híbridos. Um sistema de CI/CD é essencial para lidar com as dependências que os componentes do aplicativo têm entre si durante a instalação, como o aplicativo Web que precisa de uma cadeia de conexão para o banco de dados.

**Gerenciar o ciclo de vida.** Devido ao fato de os recursos de um aplicativo híbrido poderem abranger locais, cada funcionalidade de gerenciamento do ciclo de vida dos locais precisa ser agregada em uma única unidade de gerenciamento do ciclo de vida. Leve em consideração como eles são criados, atualizados e excluídos.

**Examinar as estratégias da solução de problemas.** A solução de problemas de um aplicativo híbrido envolve mais componentes de aplicativo do que o mesmo aplicativo que esteja sendo executado em uma única nuvem. Além da conectividade entre as nuvens, o aplicativo é executado em duas plataformas em vez de em apenas uma. Uma importante tarefa na solução de problemas de aplicativos híbridos é examinar a integridade agregada e o monitoramento de desempenho dos componentes do aplicativo.

## <a name="security"></a>Segurança

A segurança é uma das principais questões em qualquer aplicativo de nuvem e se torna ainda mais importante para aplicativos de nuvem híbrida.

Para a discussão central sobre esse pilar, confira [*Segurança*](/azure/architecture/guide/pillars#security) nos cinco pilares da excelência em arquitetura.

### <a name="security-checklist"></a>Lista de verificação de segurança

**Assumir a violação.** Caso uma parte do aplicativo esteja comprometida, verifique se há soluções definidas para minimizar a disseminação da violação, não apenas dentro do mesmo local, mas também entre locais.

**Monitorar o acesso permitido à rede.** Determine as políticas de acesso à rede do aplicativo, por exemplo, acessar o aplicativo apenas a partir de uma sub-rede específica e permitir apenas portas e protocolos mínimos entre os componentes necessários para que o aplicativo funcione corretamente.

**Empregar a autenticação robusta.** Para a segurança do seu aplicativo, é essencial ter um esquema de autenticação robusto. Cogite usar um provedor de identidade federado que forneça recursos de logon único e empregue um ou mais dos seguintes esquemas: logon com nome de usuário e senha, chaves públicas e privadas, autenticação de dois fatores ou multifator e grupos de segurança confiáveis. Determine os recursos apropriados para armazenar dados confidenciais e outros segredos para a autenticação do aplicativo, além de tipos de certificados e seus requisitos.

**Usar criptografia.** Identifique quais áreas do aplicativo usam criptografia; por exemplo, para armazenamento de dados ou comunicação e acesso do cliente.

**Usar canais seguros.** É essencial ter um canal seguro entre as nuvens para fornecer verificações de segurança e autenticação, proteção em tempo real, quarentena e outros serviços entre nuvens.

**Definir e usar funções.** Implemente funções para configurações de recursos e de acesso de identidade única entre nuvens. Determine os requisitos de RBAC (controle de acesso baseado em função) do aplicativo e seus recursos de plataforma.

**Auditar seu sistema.** O monitoramento do sistema consegue registrar e agregar dados dos componentes do aplicativo e das operações de plataforma de nuvem relacionadas.

## <a name="summary"></a>Resumo

Este artigo fornece uma lista de verificação dos itens que devem ser levados em consideração durante a criação e o design de seus aplicativos híbridos. Ao revisar esses pilares antes de implantar seu aplicativo, você evita se deparar com essas perguntas durante interrupções de produção e pode evitar ter de rever o design.

Inicialmente, pode parecer uma tarefa demorada, mas você obtém seu retorno sobre o investimento facilmente se projetar seu aplicativo com base nesses pilares.

## <a name="next-steps"></a>Próximas etapas

Para saber mais, consulte os recursos a seguir:

- [Nuvem híbrida](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Aplicativos de nuvem híbrida](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Desenvolva modelos do Azure Resource Manager para consistência de nuvem](/azure/azure-resource-manager/templates/templates-cloud-consistency)