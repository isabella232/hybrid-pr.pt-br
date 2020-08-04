---
title: Implantar um aplicativo que escale entre nuvens no Azure e no Azure Stack Hub
description: Saiba como implantar um aplicativo que escala entre nuvens no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 10cb042e2c6d0c6cb567e14072cd80bc663d686c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477330"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Implantar um aplicativo que escale entre nuvens usando o Azure e o Azure Stack Hub

Saiba como criar uma solução entre nuvens para fornecer um processo disparado manualmente para alternar de um aplicativo Web hospedado no Azure Stack Hub para um aplicativo Web hospedado no Azure com dimensionamento automático por meio do gerenciador de tráfego. Esse processo garante um utilitário de nuvem flexível e escalonável quando sob carga.

Com esse padrão, seu locatário pode não estar pronto para executar seu aplicativo na nuvem pública. No entanto, pode ser economicamente inviável para a empresa manter a capacidade necessária em seu ambiente local para lidar com picos de demanda do aplicativo. Seu locatário pode fazer uso da elasticidade da nuvem pública com sua solução local.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Criar um aplicativo Web com vários nós.
> - Configurar e gerenciar o processo de CD (implantação contínua).
> - Publique o aplicativo Web no Azure Stack Hub.
> - Crie uma versão.
> - Aprenda a monitorar e acompanhar suas implantações.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

- Assinatura do Azure. Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Um sistema integrado do Azure Stack Hub ou implantação do ASDK (Kit de Desenvolvimento do Azure Stack).
  - Para obter instruções sobre como instalar o Azure Stack Hub, confira [Instalar o ASDK](/azure-stack/asdk/asdk-install.md).
  - Para um script de automação pós-implantação do ASDK, acesse: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - É possível que essa instalação precise de algumas horas para ser concluída.
- Implante serviços PaaS do [Serviço de Aplicativo](/azure-stack/operator/azure-stack-app-service-deploy.md) no Azure Stack Hub.
- [Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) no ambiente do Azure Stack Hub.
- [Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) no ambiente do Azure Stack Hub.
- Crie um aplicativo Web na assinatura de locatário. Anote a nova URL do aplicativo Web para uso posterior.
- Implante a máquina virtual (VM) do Azure Pipelines na assinatura do locatário.
- É necessário ter a VM do Windows Server 2016 com o .NET 3.5. Essa VM será criada na assinatura de locatário no Azure Stack Hub como o agente de compilação particular.
- [O Windows Server 2016 com a imagem da VM do SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Marketplace do Azure Stack Hub. Caso essa imagem esteja indisponível, trabalhe com um operador do Azure Stack Hub para garantir que ele seja adicionado ao ambiente.

## <a name="issues-and-considerations"></a>Problemas e considerações

### <a name="scalability"></a>Escalabilidade

O principal componente da escala entre nuvens é a capacidade de fornecer colocação em escala imediata e sob demanda entre a infraestrutura de nuvem pública e local, fornecendo um serviço consistente e confiável.

### <a name="availability"></a>Disponibilidade

Verifique se os aplicativos implantados localmente estão configurados para alta disponibilidade por meio da configuração de hardware local e da implantação de software.

### <a name="manageability"></a>Capacidade de gerenciamento

A solução entre nuvens garante o gerenciamento contínuo e a interface familiar entre ambientes. O PowerShell é recomendado para o gerenciamento entre plataformas.

## <a name="cross-cloud-scaling"></a>Escala entre nuvens

### <a name="get-a-custom-domain-and-configure-dns"></a>Obter um domínio personalizado e configurar o DNS

Atualize o arquivo de zona DNS do domínio. O Azure AD verificará a propriedade do nome de domínio personalizado. Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Office 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registre um domínio personalizado com um registrador público.
2. Entre no registrador de nome de domínio para o domínio. Talvez seja necessário um administrador aprovado para fazer atualizações de DNS.
3. Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. A entrada DNS não afetará o roteamento de email nem os comportamentos de hospedagem na Web.

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Criar um aplicativo Web de vários nós padrão no Azure Stack Hub

Configure o CI/CD (integração contínua/implantação contínua) para implantar aplicativos Web no Azure e no Azure Stack Hub e enviar alterações por push a ambas as nuvens.

> [!Note]  
> São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo. Para obter mais informações, confira a documentação do Serviço de Aplicativo sobre os [Pré-requisitos de implantação do Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Adicionar código ao Azure Repos

Azure Repos

1. Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Repos.

    O CI/CD híbrido pode ser aplicado tanto ao código do aplicativo quanto ao código de infraestrutura. Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privado e hospedado.

    ![Conectar-se a um projeto no Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

    ![Clonar repositório no aplicativo Web do Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implantação de aplicativo Web independente para Serviços de Aplicativos em ambas as nuvens

1. Edite o arquivo **WebApplication.csproj**. Selecione `Runtimeidentifier` e adicione `win10-x64`. Confira a documentação [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

    ![Editar arquivo de projeto de aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Faça check-in do código no Azure Repos usando o Team Explorer.

3. Confirme se o código do aplicativo foi verificado no Azure Repos.

## <a name="create-the-build-definition"></a>Criar a definição de build

1. Entre no Azure Pipelines para confirmar a capacidade de criar definições de build.

2. Adicione o código **-r win10-x64**. Essa adição é necessária para disparar uma implantação independente com o .NET Core.

    ![Adicionar código ao aplicativo Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Execute o build. O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que são executados no Azure e no Azure Stack Hub.

## <a name="use-an-azure-hosted-agent"></a>Usar um agente hospedado no Azure

Usar um agente de build hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="manage-and-configure-the-cd-process"></a>Gerenciar e configurar o processo de CD

O Azure Pipelines e o Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade e ambientes de produção, incluindo a exigência de aprovações em estágios específicos.

## <a name="create-release-definition"></a>Criar definição de versão

1. Selecione o botão **mais** para adicionar uma nova versão na guia **Versões** na seção **Build e Versão** do Azure DevOps Services.

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Aplique o modelo de implantação do Serviço de Aplicativo do Azure.

   ![Aplicar o modelo de implantação do Serviço de Aplicativo do Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Em **Adicionar artefato**, adicione o artefato para o aplicativo de build no Azure Cloud.

   ![Adicionar artefato ao build do Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Na guia Pipeline, selecione o link **Fase, Tarefa** do ambiente e defina os valores do ambiente de nuvem do Azure.

   ![Definir os valores do ambiente de nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Defina o **nome do ambiente** e selecione a **assinatura do Azure** do ponto de extremidade da Nuvem do Azure.

      ![Selecione a assinatura do Azure para o ponto de extremidade do Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Em **Nome do serviço de aplicativo**, defina o nome do serviço de aplicativo necessário do Azure.

      ![Definir o nome do serviço de aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Insira “Hosted VS2017” em **Fila de agentes** para o ambiente hospedado na nuvem do Azure.

      ![Definir fila de agentes para o ambiente hospedado na nuvem do Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. No menu Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a **localização da pasta**.
  
      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Salve todas as alterações e volte para o **pipeline de lançamento**.

    ![Salvar alterações no pipeline de lançamento](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Adicione um novo artefato selecionando o build do aplicativo Azure Stack Hub.

    ![Adicionar novo artefato para o aplicativo do Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Adicione mais um ambiente aplicando a Implantação do Serviço de Aplicativo do Azure.

    ![Adicionar ambiente à implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Nomeie o novo ambiente “Azure Stack”.

    ![Nomear ambiente na implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Localize o ambiente do Azure Stack na guia **Tarefa**.

    ![Ambiente do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selecione a assinatura para o ponto de extremidade do Azure Stack.

    ![Selecionar a assinatura para o ponto de extremidade do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Defina o nome do aplicativo Web Azure Stack como o nome do serviço de aplicativo.
    ![Definir o nome do aplicativo Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selecione o agente do Azure Stack.

    ![Selecionar o agente do Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Na seção Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a localização da pasta.

    ![Selecionar a pasta para a Implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecionar a pasta para a Implantação do Serviço de Aplicativo do Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Na guia Variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, defina seu valor como **true** e configura o escopo no Azure Stack.

    ![Adicionar variável à implantação do Aplicativo Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selecione o ícone de gatilho de implantação **Contínuo** em ambos os artefatos e habilite o gatilho de implantação **Continuar**.

    ![Selecionar gatilho de implantação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selecione o ícone de condições de **Pré-implantação** no ambiente do Azure Stack e defina o gatilho como **Após o lançamento.**

    ![Selecionar condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Salvar todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) na criação de uma definição da versão a partir de um modelo. Essas configurações não podem ser modificadas nas configurações da tarefa. Em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publicar no Azure Stack Hub por meio do Visual Studio

Criando pontos de extremidade, um build do Azure DevOps Services pode implantar aplicativos de serviço do Azure no Azure Stack Hub. O Azure Pipelines se conecta ao agente de build, que se conecta ao Azure Stack Hub.

1. Entre no Azure DevOps Services e acesse a página de configurações do aplicativo.

2. Em **Configurações**, selecione **Segurança**.

3. Em **grupos do VSTS**, selecione **Criadores de ponto de extremidade**.

4. Na guia **Membros**, selecione **Adicionar**.

5. Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.

6. Selecione **Salvar alterações**.

7. Na lista **Grupos do VSTS**, selecione **Administradores do ponto de extremidade**.

8. Na guia **Membros**, selecione **Adicionar**.

9. Em **Adicionar usuários e grupos**, insira um nome de usuário e selecione esse usuário na lista de usuários.

10. Selecione **Salvar alterações**.

Agora que as informações do ponto de extremidade existem, a conexão do Azure Pipelines com o Azure Stack Hub está pronta para uso. O agente de build no Azure Stack Hub obtém instruções do Azure Pipelines e transmite informações do ponto de extremidade para comunicação com o Azure Stack Hub.

## <a name="develop-the-app-build"></a>Desenvolver o build do aplicativo

> [!Note]  
> São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo. Para obter mais informações, confira [Pré-requisitos para implantar o Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) como código do aplicativo Web do Azure Repos para implantar em ambas as nuvens.

### <a name="add-code-to-an-azure-repos-project"></a>Adicionar código a um projeto do Azure Repos

1. Entre no Azure Repos com uma conta que tenha direitos de criação de projeto no Azure Stack Hub.

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implantação de aplicativo Web independente para Serviços de Aplicativos em ambas as nuvens

1. Edite o arquivo **WebApplication.csproj**: Selecione `Runtimeidentifier` e, em seguida, adicione `win10-x64`. Para obter mais informações, confira a documentação [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

2. Use o Team Explorer para verificar o código no Azure Repos.

3. Confirme se o código do aplicativo foi verificado no Azure Repos.

### <a name="create-the-build-definition"></a>Criar a definição de build

1. Entre no Azure Pipelines com uma conta que possa criar uma definição de build.

2. Acesse a página do projeto **Aplicativo Web do build**.

3. Em **Argumentos**, adicione o código **-r win10-x64**. Essa adição é necessária para disparar uma implantação autossuficiente com o .NET Core.

4. Execute o build. O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Azure Stack Hub.

#### <a name="use-an-azure-hosted-build-agent"></a>Usar um agente de build hospedado no Azure

Usar um agente de build hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configure o processo de CD (implantação contínua)

O Azure Pipelines e o Azure DevOps Services fornecem um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade (QA) e produção. Esse processo pode incluir a exigência de aprovações em estágios específicos do ciclo de vida do aplicativo.

#### <a name="create-release-definition"></a>Criar definição de versão

Criar uma definição da versão é a etapa final no processo de build do aplicativo. Esta definição da versão é usada para criar uma versão e implantar um build.

1. Entre no Azure Pipelines e acesse **Build e Versão** para navegar até o projeto.

2. Na guia **Versões**, selecione **[+]** e escolha **Criar definição de versão**.

3. Em **Selecionar um modelo**, escolha **Implantação do Serviço de Aplicativo do Azure**e selecione **Aplicar**.

4. Em **Adicionar artefato** na **Origem (Definição de build)** , selecione o aplicativo de build do Azure Cloud.

5. Na guia **Pipeline**, selecione o link **1 Fase**, **1 Tarefa** para **Exibir tarefas de ambiente**.

6. Na guia **Tarefas**, insira Azure como **Nome do ambiente** e selecione AzureCloud Traders-Web EP na lista de **Assinaturas do Azure**.

7. Insira o **nome do Serviço de Aplicativo do Azure**, que é `northwindtraders` na próxima captura de tela.

8. Na fase de agente, selecione **VS2017 Hospedado** na lista **Fila de agentes**.

9. Em **Implantar Serviço de Aplicativo do Azure**, selecione o **Pacote ou pasta** válido para o ambiente.

10. Em **Selecionar Arquivo ou Pasta**, selecione **OK** para a **Localização**.

11. Salve todas as alterações e volte para o **Pipeline**.

12. Na guia **Pipeline**, selecione **Adicionar artefato** e escolha **NorthwindCloud Traders-Vessel** na lista **Origem (Definição de build)** .

13. Em **Selecionar um Modelo**, adicione outro ambiente. Selecione **Implantação do Serviço de Aplicativo do Azure** e, em seguida, **Aplicar**.

14. Insira `Azure Stack Hub` como o **Nome do ambiente**.

15. Na guia **Tarefas**, localize e selecione Azure Stack Hub.

16. Na lista de **Assinaturas do Azure**, selecione **AzureStack Traders-Vessel EP** para o ponto de extremidade do Azure Stack Hub.

17. Insira o aplicativo Web do Azure Stack Hub como o **Nome do serviço de aplicativo**.

18. Em **Seleção do agente**, selecione **AzureStack -b Douglas Fir** na lista **Fila de agentes**.

19. Em **Implantar Serviço de Aplicativo do Azure**, selecione o **Pacote ou pasta** válido para o ambiente. Em **Selecionar Arquivo ou Pasta**, selecione **OK** para a pasta **Localização**.

20. Na guia **Variável**, localize a variável denominada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`. Defina o valor da variável como **true** e defina seu escopo como **Azure Stack Hub**.

21. Na guia **Pipeline**, selecione o ícone do **Gatilho de implantação contínua** para o artefato NorthwindCloud Traders-Web e defina o **Gatilho de implantação contínua** como **Habilitado**. Faça o mesmo para o artefato **NorthwindCloud Traders-Vessel**.

22. No ambiente do Azure Stack Hub, selecione o ícone das **Condições de pré-implantação** para definir o gatilho como **Após o lançamento**.

23. Salvar todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas de lançamento são automaticamente definidas como [variável de ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição da versão de um modelo. Essas configurações não podem ser modificadas nas configurações de tarefa, mas podem ser modificadas nos itens do ambiente pai.

## <a name="create-a-release"></a>Criar uma versão

1. Na guia **Pipeline**, abra a lista **Versão** e selecione **Criar versão**.

2. Insira uma descrição para a versão, verifique se os artefatos corretos estão selecionados e, em seguida, selecione **Criar**. Após alguns instantes, um banner aparece indicando que a nova versão foi criada e o nome da versão é exibido como um link. Selecione o link para ver a página de resumo da versão.

3. A página de resumo da versão mostra detalhes sobre ela. Na captura de tela a seguir da “Versão-2”, a seção **Ambientes** mostra o **Status de implantação** do Azure como “EM ANDAMENTO”, e o status do Azure Stack Hub aparece como “BEM-SUCEDIDO”. Quando o status de implantação do ambiente do Azure muda para “BEM-SUCEDIDO”, um banner aparece indicando que a versão está pronta para aprovação. Quando uma implantação está pendente ou apresentou falhas, um ícone de informações azul **(i)** aparece. Passe o mouse sobre o ícone para ver um pop-up que contém o motivo de atraso ou falha.

4. Outras exibições, como a lista de versões, também exibem um ícone que indica que a aprovação está pendente. O pop-up deste ícone mostra o nome do ambiente e mais detalhes relacionados à implantação. É fácil para um administrador ver o progresso geral das versões e ver quais versões estão aguardando aprovação.

## <a name="monitor-and-track-deployments"></a>Monitorar e acompanhar implantações

1. Na página de resumo **Versão-2**, selecione **Logs**. Durante uma implantação, essa página mostra o log ativo do agente. O painel esquerdo mostra o status de cada operação na implantação para cada ambiente.

2. Selecione o ícone de pessoa na coluna **Ação** para obter uma aprovação de pré-implantação ou pós-implantação e ver quem aprovou (ou recusou) a implantação e a mensagem que a pessoa forneceu.

3. Após a conclusão da implantação, todo o arquivo de log será exibido no painel direito. Selecione qualquer **Etapa** no painel esquerdo para ver o arquivo de log de uma etapa única, como **Inicializar Trabalho**. A habilidade de ver logs individuais facilita o rastreamento e a depuração de partes da implantação geral. **Salve** o arquivo de log de uma etapa ou **Baixe todos os logs como zip**.

4. Abra a guia **Resumo** para ver informações gerais sobre a versão. Essa exibição mostra detalhes sobre o build, os ambientes em que foram implantados, o status da implantação e outras informações sobre a versão.

5. Selecione um link de ambiente (**Azure** ou **Azure Stack Hub**) para ver informações sobre implantações existentes ou pendentes em um ambiente específico. Use essas exibições como uma maneira rápida de verificar se o mesmo build foi implantado em ambos os ambientes.

6. Abra o **aplicativo de produção implantado** em um navegador. Por exemplo, para o site dos Serviços de Aplicativos do Azure, abra a URL `https://[your-app-name\].azurewebsites.net`.

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>A integração do Azure e do Azure Stack Hub fornece uma solução escalonável entre nuvens

Um serviço flexível e robusto de várias nuvens fornece segurança de dados, backup e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escalonáveis e roteamento de conformidade geográfica. Esse processo disparado manualmente garante uma troca de cargas confiável e eficiente entre aplicativos Web hospedados e a disponibilidade imediata de dados cruciais.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).
