---
title: Implantar um aplicativo que escale entre nuvem no Azure e no Hub de Azure Stack
description: Saiba como implantar um aplicativo que dimensiona entre nuvem no Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909877"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Implantar um aplicativo que escale entre nuvem usando o Azure e o Hub de Azure Stack

Saiba como criar uma solução de nuvem cruzada para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado pelo Hub Azure Stack para um aplicativo Web hospedado do Azure com dimensionamento automático por meio do Gerenciador de tráfego. Esse processo garante um utilitário de nuvem flexível e escalonável quando sob carga.

Com esse padrão, seu locatário pode não estar pronto para executar seu aplicativo na nuvem pública. No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda para o aplicativo. Seu locatário pode fazer uso da elasticidade da nuvem pública com sua solução local.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Crie um aplicativo Web de vários nós.
> - Configure e gerencie o processo de implantação contínua (CD).
> - Publique o aplicativo Web no Hub Azure Stack.
> - Crie uma versão.
> - Aprenda a monitorar e acompanhar suas implantações.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

- Assinatura do Azure. Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Um sistema integrado de Hub Azure Stack ou implantação de Kit de Desenvolvimento do Azure Stack (ASDK).
  - Para obter instruções sobre como instalar Azure Stack Hub, consulte [instalar o ASDK](/azure-stack/asdk/asdk-install.md).
  - Para um script de automação pós-implantação do ASDK, acesse:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Essa instalação pode exigir algumas horas para ser concluída.
- Implante serviços de PaaS do [serviço de aplicativo](/azure-stack/operator/azure-stack-app-service-deploy.md) para Azure Stack Hub.
- [Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) dentro do ambiente de Hub de Azure Stack.
- [Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro do ambiente de Hub de Azure Stack.
- Crie um aplicativo Web dentro da assinatura de locatário. Anote a nova URL do aplicativo Web para uso posterior.
- Implante Azure Pipelines máquina virtual (VM) na assinatura do locatário.
- A VM do Windows Server 2016 com o .NET 3,5 é necessária. Essa VM será criada na assinatura de locatário no Hub de Azure Stack como o agente de compilação particular.
- O [Windows Server 2016 com a imagem de VM do SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Marketplace do Hub de Azure Stack. Se essa imagem não estiver disponível, trabalhe com um operador de Hub de Azure Stack para garantir que ele seja adicionado ao ambiente.

## <a name="issues-and-considerations"></a>Problemas e considerações

### <a name="scalability"></a>Escalabilidade

O principal componente do dimensionamento entre nuvem é a capacidade de fornecer dimensionamento imediato e sob demanda entre a infraestrutura de nuvem pública e local, fornecendo um serviço consistente e confiável.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

A solução de nuvem cruzada garante o gerenciamento contínuo e a interface familiar entre ambientes. O PowerShell é recomendado para gerenciamento de plataforma cruzada.

## <a name="cross-cloud-scaling"></a>Dimensionamento entre nuvem

### <a name="get-a-custom-domain-and-configure-dns"></a>Obter um domínio personalizado e configurar o DNS

Atualize o arquivo de zona DNS para o domínio. O AD do Azure verificará a propriedade do nome de domínio personalizado. Use o [DNS do Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) para registros DNS do Azure/Office 365/externos no Azure ou adicione a entrada DNS em [um registrador de DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registre um domínio personalizado com um registrador público.
2. Entre no registrador de nome de domínio para o domínio. Um administrador aprovado pode ser necessário para fazer atualizações de DNS.
3. Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. (A entrada DNS não afetará o roteamento de email ou os comportamentos de hospedagem na Web.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Criar um aplicativo Web de vários nós padrão no Hub de Azure Stack

Configure a integração contínua híbrida e a CI/CD (implantação contínua) para implantar aplicativos Web no Azure e no Hub de Azure Stack e para enviar alterações por push a ambas as nuvens.

> [!Note]  
> O Hub de Azure Stack com imagens apropriadas agregadas para execução (Windows Server e SQL) e implantação do serviço de aplicativo são necessários. Para obter mais informações, examine os pré-requisitos de documentação do serviço [de aplicativo para implantar o serviço de aplicativo no Hub de Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Adicionar código a Azure Repos

Azure Repos

1. Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Repos.

    CI/CD híbrido pode ser aplicado tanto ao código do aplicativo quanto ao código de infraestrutura. Use [modelos de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privada e hospedado.

    ![Conectar-se a um projeto no Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

    ![Clonar repositório no aplicativo Web do Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implantação de aplicativo Web independente para serviços de aplicativos em ambas as nuvens

1. Edite o arquivo **WebApplication. csproj** . Selecione `Runtimeidentifier` e adicione `win10-x64` . (Consulte a documentação [de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)

    ![Editar arquivo de projeto do aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Faça check-in no código para Azure Repos usando Team Explorer.

3. Confirme se o código do aplicativo foi verificado Azure Repos.

## <a name="create-the-build-definition"></a>Criar a definição de compilação

1. Entre no Azure Pipelines para confirmar a capacidade de criar definições de compilação.

2. Adicione **-r win10-código x64** . Essa adição é necessária para disparar uma implantação independente com o .NET Core.

    ![Adicionar código ao aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Execute a compilação. O processo de [compilação de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que são executados no Azure e Azure Stack Hub.

## <a name="use-an-azure-hosted-agent"></a>Usar um agente hospedado do Azure

Usar um agente de compilação hospedado no Azure Pipelines é uma opção conveniente para criar e implantar aplicativos Web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="manage-and-configure-the-cd-process"></a>Gerenciar e configurar o processo de CD

Azure Pipelines e Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável para versões para vários ambientes, como desenvolvimento, preparo, QA e ambientes de produção; incluindo a necessidade de aprovações em estágios específicos.

## <a name="create-release-definition"></a>Criar definição de versão

1. Selecione o botão de **adição** para adicionar uma nova versão na guia **versões** na seção **Build e versão** do Azure DevOps Services.

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Aplique o modelo de implantação do serviço de Azure App.

   ![Aplicar Azure App modelo de implantação do serviço](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Em **Adicionar artefato**, adicione o artefato para o aplicativo de compilação na nuvem do Azure.

   ![Adicionar artefato à compilação na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Na guia pipeline, selecione a **fase,** o link de tarefa do ambiente e defina os valores de ambiente de nuvem do Azure.

   ![Definir valores de ambiente de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Defina o **nome do ambiente** e selecione a **assinatura do Azure** para o ponto de extremidade de nuvem do Azure.

      ![Selecione a assinatura do Azure para o ponto de extremidade de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Em **nome do serviço de aplicativo**, defina o nome do serviço de aplicativo do Azure necessário.

      ![Definir nome do serviço de aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Insira "Hosted VS2017" na **fila do agente** para o ambiente hospedado na nuvem do Azure.

      ![Definir fila do agente para o ambiente hospedado na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. No menu implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente. Selecione **OK** para a **pasta local**.
  
      ![Selecionar pacote ou pasta para o ambiente de serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecionar pacote ou pasta para o ambiente de serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Salve todas as alterações e volte para o **pipeline de liberação**.

    ![Salvar alterações no pipeline de lançamento](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Adicione um novo artefato selecionando a compilação para o aplicativo de Hub de Azure Stack.

    ![Adicionar novo artefato para o aplicativo de Hub de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Adicione mais um ambiente aplicando a implantação do serviço de Azure App.

    ![Adicionar ambiente à implantação do serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Nomeie o novo ambiente "Azure Stack".

    ![Ambiente de nome na implantação do serviço Azure App](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Localize o ambiente Azure Stack na guia **tarefa** .

    ![Ambiente de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selecione a assinatura para o ponto de extremidade de Azure Stack.

    ![Selecione a assinatura para o ponto de extremidade Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Defina o nome do aplicativo Web Azure Stack como o nome do serviço de aplicativo.
    ![Definir Azure Stack nome do aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selecione o agente de Azure Stack.

    ![Selecione o agente de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Na seção implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente. Selecione **OK** para a pasta local.

    ![Selecionar pasta para implantação de serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecionar pasta para implantação de serviço de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Na guia variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , defina seu valor como **true**e escopo como Azure Stack.

    ![Adicionar variável à implantação de Azure App](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selecione o ícone de gatilho de implantação **contínua** em ambos os artefatos e habilite o gatilho de implantação **continua** .

    ![Selecionar gatilho de implantação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selecione o ícone condições de **pré-implantação** no ambiente de Azure Stack e defina o gatilho para **após a liberação.**

    ![Selecionar condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Salve todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) ao criar uma definição de versão a partir de um modelo. Essas configurações não podem ser modificadas nas configurações da tarefa; em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publicar no Hub de Azure Stack por meio do Visual Studio

Criando pontos de extremidade, um Azure DevOps Services Build pode implantar aplicativos de serviço do Azure para Azure Stack Hub. Azure Pipelines se conecta ao agente de compilação, que se conecta ao Hub Azure Stack.

1. Entre no Azure DevOps Services e vá para a página de configurações do aplicativo.

2. Em **Configurações**, selecione **Segurança**.

3. Em **grupos do VSTS**, selecione **criadores de ponto de extremidade**.

4. Na guia **Membros**, selecione **Adicionar**.

5. Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.

6. Selecione **Salvar alterações**.

7. Na lista de **grupos do VSTS** , selecione administradores de ponto de **extremidade**.

8. Na guia **Membros**, selecione **Adicionar**.

9. Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.

10. Selecione **Salvar alterações**.

Agora que as informações do ponto de extremidade existem, o Azure Pipelines para Azure Stack conexão de Hub está pronto para uso. O agente de compilação no Hub Azure Stack Obtém instruções de Azure Pipelines e, em seguida, o agente transmite informações de ponto de extremidade para comunicação com o Hub de Azure Stack.

## <a name="develop-the-app-build"></a>Desenvolver a compilação do aplicativo

> [!Note]  
> O Hub de Azure Stack com imagens apropriadas agregadas para execução (Windows Server e SQL) e implantação do serviço de aplicativo são necessários. Para obter mais informações, consulte [pré-requisitos para implantar o serviço de aplicativo no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Use [modelos de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) como código do aplicativo web de Azure Repos para implantar em ambas as nuvens.

### <a name="add-code-to-an-azure-repos-project"></a>Adicionar código a um projeto Azure Repos

1. Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Hub Azure Stack.

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implantação de aplicativo Web independente para serviços de aplicativos em ambas as nuvens

1. Edite o arquivo **WebApplication. csproj** : selecione `Runtimeidentifier` e adicione `win10-x64` . Para obter mais informações, consulte a documentação de [implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

2. Use Team Explorer para verificar o código em Azure Repos.

3. Confirme se o código do aplicativo foi verificado em Azure Repos.

### <a name="create-the-build-definition"></a>Criar a definição de compilação

1. Entre no Azure Pipelines com uma conta que possa criar uma definição de compilação.

2. Vá para a página **criar aplicativo Web** para o projeto.

3. Em **argumentos**, adicione o código **win10-x64** . Essa adição é necessária para disparar uma implantação independente com o .NET Core.

4. Execute a compilação. O processo de [compilação de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Hub de Azure Stack.

#### <a name="use-an-azure-hosted-build-agent"></a>Usar um agente de compilação hospedado do Azure

Usar um agente de compilação hospedado no Azure Pipelines é uma opção conveniente para criar e implantar aplicativos Web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configurar o processo de implantação contínua (CD)

Azure Pipelines e Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável para versões para vários ambientes, como desenvolvimento, preparo, controle de qualidade (QA) e produção. Esse processo pode incluir a exigência de aprovações em estágios específicos do ciclo de vida do aplicativo.

#### <a name="create-release-definition"></a>Criar definição de versão

A criação de uma definição de versão é a etapa final no processo de compilação do aplicativo. Esta definição de versão é usada para criar uma versão e implantar uma compilação.

1. Entre no Azure Pipelines e vá para **Compilar e liberar** para o projeto.

2. Na guia **versões** , selecione **[+]** e escolha **criar definição de versão**.

3. Em **selecionar um modelo**, escolha **Azure app implantação de serviço**e, em seguida, selecione **aplicar**.

4. Em **Adicionar artefato**, na **origem (definição de compilação)**, selecione o aplicativo Azure cloud Build.

5. Na guia **pipeline** , selecione a **fase 1**, 1 link de **tarefa** para **Exibir tarefas de ambiente**.

6. Na guia **tarefas** , insira Azure como o **nome do ambiente** e selecione o AzureCloud Traders-Web EP na lista de **assinaturas do Azure** .

7. Insira o **nome do serviço de aplicativo do Azure**, que está `northwindtraders` na próxima captura de tela.

8. Para a fase do agente, selecione **Hosted VS2017** na lista de **filas do agente** .

9. Em **implantar Azure app serviço**, selecione o **pacote ou a pasta** válida para o ambiente.

10. Em **Selecionar arquivo ou pasta**, selecione **OK** para o **local**.

11. Salve todas as alterações e volte para o **pipeline**.

12. Na guia **pipeline** , selecione **Adicionar artefato**e escolha o **NorthwindCloud Traders-recipiente** da lista de **origem (definição de compilação)** .

13. Em **selecionar um modelo**, adicione outro ambiente. Escolha **implantação do serviço de Azure app** e, em seguida, selecione **aplicar**.

14. Insira `Azure Stack Hub` como o **nome do ambiente**.

15. Na guia **tarefas** , localize e selecione Azure Stack Hub.

16. Na lista de **assinaturas do Azure** , selecione **AzureStack Traders-recipiente EP** para o ponto de extremidade do hub de Azure Stack.

17. Insira o nome do aplicativo Web do hub de Azure Stack como o **nome do serviço de aplicativo**.

18. Em **seleção do agente**, escolha **AzureStack-b Douglas Fir** na lista **fila do agente** .

19. Para **implantar Azure app serviço**, selecione o **pacote ou a pasta** válida para o ambiente. Em **Selecionar arquivo ou pasta**, selecione **OK** para o **local**da pasta.

20. Na guia **variável** , localize a variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . Defina o valor da variável como **true**e defina seu escopo como **Azure Stack Hub**.

21. Na guia **pipeline** , selecione o ícone de **gatilho de implantação contínua** para o artefato da Web do NorthwindCloud Traders e defina o **gatilho de implantação contínua** como **habilitado**. Faça a mesma coisa para o artefato de **embarcação do NorthwindCloud Traders** .

22. Para o ambiente de Hub de Azure Stack, selecione o ícone **condições de pré-implantação** definir o gatilho para **após a liberação**.

23. Salve todas as alterações.

> [!Note]  
> Algumas configurações para tarefas de liberação são definidas automaticamente como [variáveis de ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) ao criar uma definição de versão a partir de um modelo. Essas configurações não podem ser modificadas nas configurações de tarefa, mas podem ser modificadas nos itens do ambiente pai.

## <a name="create-a-release"></a>Criar uma versão

1. Na guia **pipeline** , abra a lista **versão** e selecione **criar versão**.

2. Insira uma descrição para a versão, verifique se os artefatos corretos estão selecionados e, em seguida, selecione **criar**. Após alguns instantes, uma faixa é exibida indicando que a nova versão foi criada e o nome da versão é exibido como um link. Selecione o link para ver a página de resumo da versão.

3. A página Resumo da versão mostra detalhes sobre a versão. Na captura de tela a seguir para a "versão 2", a seção **ambientes** mostra o **status de implantação** do Azure como "em andamento", e o status para Azure Stack Hub é "êxito". Quando o status de implantação do ambiente do Azure muda para "êxito", é exibida uma faixa indicando que a versão está pronta para aprovação. Quando uma implantação está pendente ou falhou, um ícone de informações azul **(i)** é mostrado. Passe o mouse sobre o ícone para ver um pop-up que contém o motivo de atraso ou falha.

4. Outras exibições, como a lista de versões, também exibem um ícone que indica que a aprovação está pendente. O pop-up deste ícone mostra o nome do ambiente e mais detalhes relacionados à implantação. É fácil para um administrador ver o progresso geral das versões e ver quais versões estão aguardando aprovação.

## <a name="monitor-and-track-deployments"></a>Monitorar e acompanhar implantações

1. Na página Resumo da **versão 2** , selecione **logs**. Durante uma implantação, essa página mostra o log ao vivo do agente. O painel esquerdo mostra o status de cada operação na implantação para cada ambiente.

2. Selecione o ícone de pessoa na coluna **ação** para uma aprovação de pré-implantação ou pós-implantação para ver quem aprovou (ou recusou) a implantação e a mensagem fornecida.

3. Após a conclusão da implantação, todo o arquivo de log será exibido no painel direito. Selecione qualquer **etapa** no painel esquerdo para ver o arquivo de log para uma única etapa, como **inicializar trabalho**. A capacidade de ver logs individuais torna mais fácil rastrear e depurar partes da implantação geral. **Salve** o arquivo de log para uma etapa ou **Baixe todos os logs como zip**.

4. Abra a guia **Resumo** para ver informações gerais sobre a versão. Essa exibição mostra detalhes sobre a compilação, os ambientes em que foram implantados, o status da implantação e outras informações sobre a versão.

5. Selecione um link de ambiente (**Azure** ou **Hub de Azure Stack**) para ver informações sobre implantações existentes e pendentes em um ambiente específico. Use essas exibições como uma maneira rápida de verificar se a mesma compilação foi implantada em ambos os ambientes.

6. Abra o **aplicativo de produção implantado** em um navegador. Por exemplo, para o site do Azure App Services, abra a URL `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>A integração do Azure e do hub de Azure Stack fornece uma solução escalonável entre nuvem

Um serviço flexível e robusto de várias nuvens fornece segurança de dados, backup e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escalonáveis e roteamento de conformidade geográfica. Esse processo disparado manualmente garante a troca de carga confiável e eficiente entre aplicativos Web hospedados e a disponibilidade imediata de dados cruciais.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).
