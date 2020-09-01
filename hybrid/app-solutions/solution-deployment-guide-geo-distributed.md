---
title: Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub
description: Saiba como direcionar o tráfego a pontos de extremidade específicos com uma solução de aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886825"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Azure Stack Hub

Saiba como direcionar o tráfego a pontos de extremidade específicos com base em várias métricas usando o padrão de aplicativos distribuídos geograficamente. Criar um perfil do Gerenciador de Tráfego com o roteamento baseado em geografia e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Criar um aplicativo distribuído geograficamente.
> - Usar o Gerenciador de Tráfego para direcionar seu aplicativo.

## <a name="use-the-geo-distributed-apps-pattern"></a>Usar o padrão de aplicativos distribuídos geograficamente

Com o padrão distribuído geograficamente, seu aplicativo abrangerá regiões. Você pode usar o envio à nuvem pública como padrão, mas alguns de seus usuários podem requerer que seus dados permaneçam na região. Com base nos requisitos dos usuários, você pode direcioná-los para a nuvem mais adequada.

### <a name="issues-and-considerations"></a>Problemas e considerações

#### <a name="scalability-considerations"></a>Considerações sobre escalabilidade

A solução que você desenvolverá com esse artigo não é a de acomodar a escalabilidade. No entanto, se usada junto com outras soluções locais e do Azure, você poderá acomodar os requisitos de escalabilidade. Para obter informações sobre como criar uma solução híbrida com dimensionamento automático por meio do Gerenciador de Tráfego, confira [Criar soluções de dimensionamento entre nuvens com o Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Considerações sobre disponibilidade

Assim como nas considerações de escalabilidade, essa solução não aborda diretamente a disponibilidade. No entanto, as soluções locais e do Azure podem ser implementadas nessa solução para garantir a alta disponibilidade para todos os componentes envolvidos.

### <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Sua organização tem ramificações internacionais que requerem políticas regionais personalizadas de segurança e distribuição.

- Cada escritório da sua organização recebe dados de funcionários, de negócios e de instalações, o que requer relatar as atividades conforme os regulamentos locais e fusos horários.

- Os requisitos de alta escala são atendidos pela escala horizontal de aplicativos, com várias implantações de aplicativo dentro de uma única região e entre regiões para lidar com requisitos de carga extrema.

### <a name="planning-the-topology"></a>Planejamento da topologia

Antes de desenvolver o volume do aplicativo distribuído, é importante saber o seguinte:

- **Domínio personalizado para o aplicativo:** Qual é o nome de domínio personalizado que os clientes usarão para acessar o aplicativo? Para o aplicativo de exemplo, o nome de domínio personalizado é *www\.scalableasedemo.com.*

- **Domínio do Gerenciador de Tráfego:** Um nome de domínio é escolhido quando criamos um [perfil do Gerenciador de Tráfego do Azure](/azure/traffic-manager/traffic-manager-manage-profiles). Esse nome é combinado com o sufixo *trafficmanager.net* para registrar uma entrada de domínio controlada pelo Gerenciador de Tráfego. Para o aplicativo de exemplo, o nome escolhido é *scalable-ase-demo*. Assim, o nome de domínio completo que é gerenciado pelo Gerenciador de Tráfego é *scalable-ase-demo.trafficmanager.net*.

- **Estratégia para dimensionar o volume do aplicativo:** Decida se o volume do aplicativo será distribuído em vários ambientes do Serviço de Aplicativo em uma única região, em várias regiões ou em um mix das duas abordagens. A decisão deve se basear nas expectativas de origem de tráfego do cliente e em quão bem o restante da infraestrutura de back-end de suporte de um aplicativo pode ser escalonado. Por exemplo, com um aplicativo 100% sem monitoração de estado, ele pode ser altamente dimensionado usando uma combinação de vários ambientes do Serviço de Aplicativo por região do Azure e multiplicado pelos ambientes do Serviço de Aplicativo implantados em várias regiões do Azure. Com mais de 15 regiões do Azure global disponíveis para escolha, os clientes podem realmente desenvolver um volume de aplicativo de hiperescala mundial. Para o aplicativo de exemplo usado aqui, três ambientes do Serviço de Aplicativo foram criados em uma única região do Azure (Centro-Sul dos EUA).

- **Convenção de nomenclatura para os ambientes do Serviço de Aplicativo:** Cada ambiente do Serviço de Aplicativo requer um nome exclusivo. Além de um ou dois ambientes do Serviço de Aplicativo, é útil ter uma convenção de nomenclatura para ajudar a identificar cada ambiente. Para o aplicativo de exemplo deste artigo, foi usada uma convenção de nomenclatura simples. Os nomes dos três ambientes do Serviço de Aplicativo são *fe1ase*, *fe2ase* e *fe3ase*.

- **Convenção de nomenclatura para os aplicativos:** como várias instâncias do aplicativo serão implantadas, é necessário um nome para cada instância do aplicativo implantado. Com o ambiente do Serviço de Aplicativo do Power Apps, o mesmo nome de aplicativo pode ser usado em vários ambientes. Como cada ambiente do Serviço de Aplicativo tem um sufixo de domínio exclusivo, os desenvolvedores podem optar por usar novamente o mesmo nome de aplicativo em cada ambiente. Por exemplo, um desenvolvedor poderia ter aplicativos nomeados da seguinte forma: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* e assim por diante. Em relação ao aplicativo usado aqui, cada instância de aplicativo tem um nome exclusivo. Os nomes de instância de aplicativo usados são *webfrontend1*, *webfrontend2* e *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="part-1-create-a-geo-distributed-app"></a>Parte 1: Criar um aplicativo distribuído geograficamente

Nessa parte, você criará um aplicativo Web.

> [!div class="checklist"]
> - Criar aplicativos Web e publicá-los.
> - Adicionar códigos ao Azure Repos.
> - Direcionar a compilação do aplicativo para vários destinos de nuvem.
> - Gerenciar e configurar o processo de CD.

### <a name="prerequisites"></a>Pré-requisitos

São necessárias uma assinatura do Azure e a instalação do Azure Stack Hub.

### <a name="geo-distributed-app-steps"></a>Etapas do aplicativo distribuído geograficamente

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Obter um domínio personalizado e configurar o DNS

Atualize o arquivo de zona DNS do domínio. Assim, o Azure AD poderá verificar a propriedade do nome de domínio personalizado. Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Microsoft 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Registre um domínio personalizado com um registrador público.

2. Entre no registrador de nome de domínio para o domínio. Um administrador aprovado pode ser necessário para fazer as atualizações de DNS.

3. Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. A entrada DNS não altera comportamentos, como o roteamento de e-mails ou a hospedagem na Web.

### <a name="create-web-apps-and-publish"></a>Criar aplicativos Web e publicá-los

Configure a integração contínua/entrega contínua (CI/CD) híbrida para implantar aplicativos Web no Azure e no Azure Stack Hub e para enviar alterações por push a ambas as nuvens.

> [!Note]  
> São necessários o Azure Stack Hub, com imagens apropriadas agregadas para execução (Windows Server e SQL), e a implantação do Serviço de Aplicativo. Para obter mais informações, confira [Pré-requisitos para implantar o Serviço de Aplicativo no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Adicionar código ao Azure Repos

1. Entre no Visual Studio com uma **conta que tenha direitos de criação de projeto** no Azure Repos.

    A CI/CD pode ser aplicada tanto ao código do aplicativo quanto ao código de infraestrutura. Use modelos do [Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privado e hospedado.

    ![Conectar-se a um projeto no Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

    ![Clonar repositório no Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Criar implantações de aplicativo Web em ambas as nuvens

1. Edite o arquivo **WebApplication.csproj**: Selecione `Runtimeidentifier` e adicione `win10-x64`. (Confira a documentação de [Implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)

    ![Editar arquivo de projeto de aplicativo Web no Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Faça check-in do código no Azure Repos** usando o Team Explorer.

3. Confirme se o **código do aplicativo** foi inserido no Azure Repos.

### <a name="create-the-build-definition"></a>Criar a definição de build

1. **Entre no Azure Pipelines** para confirmar a capacidade de criar definições de build.

2. Adicione o código `-r win10-x64`. Essa adição é necessária para disparar uma implantação independente com o .NET Core.

    ![Adicionar código à definição de build no Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Execute o build**. O processo de [build de implantação autossuficiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Azure Stack Hub.

#### <a name="using-an-azure-hosted-agent"></a>Usar um agente hospedado do Azure

Usar um agente hospedado no Azure Pipelines é uma opção conveniente para compilar e implantar aplicativos Web. A manutenção e as atualizações são executadas automaticamente pelo Microsoft Azure, o que permite desenvolvimentos ininterruptos, testes e implantações.

### <a name="manage-and-configure-the-cd-process"></a>Gerenciar e configurar o processo de CD

O Azure DevOps Services fornece um pipeline altamente configurável e gerenciável de versões para vários ambientes, como desenvolvimento, preparo, garantia de qualidade e ambientes de produção, incluindo a necessidade de aprovações em estágios específicos.

## <a name="create-release-definition"></a>Criar definição de versão

1. Selecione o botão **mais** para adicionar uma nova versão na guia **Versões** na seção **Build e Versão** do Azure DevOps Services.

    ![Criar uma definição de versão no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Aplique o modelo de implantação do Serviço de Aplicativo do Azure.

   ![Aplicar o modelo de implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. Em **Adicionar artefato**, adicione o artefato para o aplicativo de build no Azure Cloud.

   ![Adicionar artefato ao build da nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Na guia Pipeline, selecione o link **Fase, Tarefa** do ambiente e defina os valores do ambiente de nuvem do Azure.

   ![Definir os valores de ambiente de nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Defina o **nome do ambiente** e selecione a **assinatura do Azure** do ponto de extremidade da Nuvem do Azure.

      ![Selecionar a assinatura do Azure para o ponto de extremidade da Nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. Em **Nome do serviço de aplicativo**, defina o nome do serviço de aplicativo necessário do Azure.

      ![Definir o nome do serviço de aplicativo Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Insira “Hosted VS2017” em **Fila de agentes** para o ambiente hospedado na nuvem do Azure.

      ![Definir a Fila de agentes para o ambiente hospedado na nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. No menu Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a **localização da pasta**.
  
      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selecionar pacote ou pasta para o ambiente do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Salve todas as alterações e volte para o **pipeline de lançamento**.

    ![Salvar alterações no pipeline de lançamento no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Adicione um novo artefato selecionando o build do aplicativo Azure Stack Hub.

    ![Adicionar novo artefato do aplicativo Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Adicione mais um ambiente aplicando a Implantação do Serviço de Aplicativo do Azure.

    ![Adicionar ambiente à Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Nomeie o novo ambiente como Azure Stack Hub.

    ![Nomear o ambiente na Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Localize o ambiente do Azure Stack Hub na guia **Tarefa**.

    ![Ambiente do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selecione a assinatura do ponto de extremidade do Azure Stack Hub.

    ![Selecionar a assinatura do ponto de extremidade do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Defina o nome do aplicativo Web Azure Stack Hub como o nome do serviço de aplicativo.

    ![Definir o nome do aplicativo Web do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selecione o agente do Azure Stack Hub.

    ![Selecionar o agente do Azure Stack Hub no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Na seção Implantar Serviço de Aplicativo do Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a localização da pasta.

    ![Selecionar a pasta da Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selecionar a pasta da Implantação do Serviço de Aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Na guia Variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, defina seu valor como **true** e defina o escopo no Azure Stack Hub.

    ![Adicionar variável à Implantação do Aplicativo Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selecione o ícone de gatilho de implantação **Contínuo** em ambos os artefatos e habilite o gatilho de implantação **Continuar**.

    ![Selecionar o gatilho de implantação contínua no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selecione o ícone de condições de **Pré-implantação** no ambiente do Azure Stack Hub e defina o gatilho com **Após o lançamento.**

    ![Selecionar as condições de pré-implantação no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Salvar todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) na criação de uma definição da versão a partir de um modelo. Essas configurações não podem ser modificadas nas configurações da tarefa. Em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.

## <a name="part-2-update-web-app-options"></a>Parte 2: Atualizar as opções do aplicativo Web

O [Serviço de Aplicativo do Azure](/azure/app-service/overview) fornece um serviço de hospedagem na Web altamente escalonável e com aplicação automática de patches.

![Serviço de aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Mapear um nome DNS personalizado existente para aplicativos Web do Azure.
> - Use um **registro CNAME** e um **registro A** para mapear um nome DNS personalizado para o Serviço de Aplicativo.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mapear um nome DNS personalizado existente para aplicativos Web do Azure

> [!Note]  
> Use um CNAME para todos os nomes DNS personalizados, exceto um domínio raiz (por exemplo, northwind.com).

Para migrar um site ativo e seu nome de domínio DNS para o Serviço de Aplicativo, consulte [Migrar um nome DNS ativo para o Serviço de Aplicativo do Azure](/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Pré-requisitos

Para concluir essa solução:

- [Crie um aplicativo do Serviço de Aplicativo](/azure/app-service/) ou use um aplicativo criado para outra solução.

- Compre um nome de domínio e garanta o acesso ao registro de DNS do provedor de domínio.

Atualize o arquivo de zona DNS do domínio. O Azure AD verificará a propriedade do nome de domínio personalizado. Use o [DNS do Azure](/azure/dns/dns-getstarted-portal) para os registros no Azure/Microsoft 365/DNS externo dentro do Azure ou adicione a entrada DNS em [um registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

- Registre um domínio personalizado com um registrador público.

- Entre no registrador de nome de domínio para o domínio. (Um administrador aprovado pode ser necessário para fazer atualizações de DNS.)

- Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.

Por exemplo, para adicionar entradas DNS para northwindcloud.com e www\.northwindcloud.com, defina as configurações do DNS para o domínio raiz northwindcloud.com.

> [!Note]  
> Um nome de domínio pode ser comprado usando o [portal do Azure](/azure/app-service/manage-custom-dns-buy-domain). Para mapear um nome DNS personalizado para um aplicativo Web, o [plano do Serviço de Aplicativo](https://azure.microsoft.com/pricing/details/app-service/) do aplicativo Web deve ser uma camada paga (**Compartilhado**, **Básico**, **Standard** ou **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Criar e mapear registros CNAME e A

#### <a name="access-dns-records-with-domain-provider"></a>Acessar registros DNS com o provedor de domínio

> [!Note]  
>  Use o DNS do Azure para configurar um nome DNS personalizado para Aplicativos Web do Azure. Para obter mais informações, consulte [Usar o DNS do Azure para fornecer as configurações de domínio personalizadas para um serviço do Azure](/azure/dns/dns-custom-domain).

1. Entre no site do provedor principal.

2. Localize a página para gerenciamento de registros DNS. Todo provedor de domínio tem sua própria interface de registros DNS. Procure áreas do site rotuladas como **Nome de Domínio**, **DNS** ou **Gerenciamento de Servidor de Nomes**.

A página com os registros DNS pode ser exibida em **Meus domínios**. Encontre o link denominado **Arquivo de zona**, **Registros DNS** ou **Configuração avançada**.

A captura de tela a seguir é um exemplo de uma página de registros DNS:

![Exemplo de página de registros DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. Em Registrador de Nome de Domínio, selecione **Adicionar ou Criar** para criar um registro. Alguns provedores têm links diferentes para adicionar tipos de registro diferentes. Confira a documentação do provedor.

2. Adicione um registro CNAME para mapear um subdomínio para o nome do host padrão do aplicativo.

   Para o exemplo do domínio www\.northwindcloud.com, adicione um registro CNAME que mapeie o nome para `<app_name>.azurewebsites.net`.

Depois da inclusão do CNAME, a página de registros DNS ficará parecida com o seguinte exemplo:

![Navegação no Portal para o aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Habilitar o mapeamento de registro CNAME no Azure

1. Em uma nova guia, entre no portal do Azure.

2. Acesse Serviços de Aplicativos.

3. Selecione aplicativo Web.

4. No painel de navegação à esquerda da página do aplicativo no portal do Azure, selecione **Domínios personalizados**.

5. Selecione o ícone **+** ao lado de **Adicionar nome do host**.

6. Digite o nome de domínio totalmente qualificado, como `www.northwindcloud.com`.

7. Selecione **Validar**.

8. Caso seja indicado, adicione registros de outros tipos (`A` ou `TXT`) aos registros DNS dos registradores de nome de domínio. O Azure fornecerá os valores e os tipos desses registros:

   a.  Um registro **A** a ser mapeado para o endereço IP do aplicativo.

   b.  Um registro **TXT** a ser mapeado para o nome do host padrão do aplicativo `<app_name>.azurewebsites.net`. O Serviço de Aplicativo usa esse registro somente em tempo de configuração para verificar a propriedade de domínio personalizado. Após a verificação, exclua o registro TXT.

9. Conclua essa tarefa na guia do registrador de domínio e revalide até que o botão **Adicionar nome de host** seja ativado.

10. Verifique se **Tipo de registro de nome do host** está definido como **CNAME** (www.example.com ou qualquer subdomínio).

11. Selecione **Adicionar nome do host**.

12. Digite o nome de domínio totalmente qualificado, como `northwindcloud.com`.

13. Selecione **Validar**. A opção **Adicionar** está ativada.

14. Verifique se **Tipo de registro de nome de host** está definido como **Registro A** (example.com).

15. **Adicionar nome do host**.

    Pode levar algum tempo para que os novos nomes do host sejam refletidos na página **Domínios personalizados** do aplicativo. Tente atualizar o navegador para atualizar os dados.
  
    ![Domínios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Caso haja um erro, uma notificação de erro de verificação será exibida na parte inferior da página. ![Erro de verificação de domínio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  As etapas acima podem ser repetidas para mapear um domínio curinga (\*.northwindcloud.com). Isso permite que quaisquer subdomínios sejam adicionados a esse serviço de aplicativo sem a necessidade de criar um registro CNAME separado para cada um. Siga as instruções do registrador para definir essa configuração.

#### <a name="test-in-a-browser"></a>Testar em um navegador

Navegue até os nomes DNS configurados anteriormente (por exemplo, `northwindcloud.com` ou `www.northwindcloud.com`).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Parte 3: Associar um certificado SSL personalizado

Nesta parte, vamos:

> [!div class="checklist"]
> - Associar o certificado SSL personalizado ao Serviço de Aplicativo.
> - Impor o HTTPS ao aplicativo.
> - Automatizar a associação de certificado SSL com scripts.

> [!Note]  
> Caso seja necessário, obtenha um certificado SSL de cliente no portal do Azure e associe-o ao aplicativo Web. Para obter mais informações, confira o [Tutorial de Certificados do Serviço de Aplicativo](/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Pré-requisitos

Para concluir essa solução:

- [Crie um aplicativo do Serviço de Aplicativo.](/azure/app-service/)
- [Mapeie um nome DNS personalizado para o aplicativo Web.](/azure/app-service/app-service-web-tutorial-custom-domain)
- Adquira um certificado SSL de uma autoridade de certificado confiável e use a chave para assinar a solicitação.

### <a name="requirements-for-your-ssl-certificate"></a>Requisitos para o certificado SSL

Para usar o certificado no Serviço de Aplicativo, o certificado deve atender a todos os seguintes requisitos:

- Ser assinado por uma autoridade de certificado confiável.

- Ser exportado como um arquivo PFX protegido por senha.

- Conter chave privada com pelo menos 2.048 bits de extensão.

- Conter todos os certificados intermediários na cadeia de certificados.

> [!Note]  
> Os **certificados ECC (Criptografia de Curva Elíptica)** funcionam com o Serviço de Aplicativo, mas não estão incluídos neste guia. Confira uma autoridade de certificação para obter assistência quanto à criação de certificados ECC.

#### <a name="prepare-the-web-app"></a>Preparar o aplicativo Web

Para a associação de um certificado SSL personalizado ao aplicativo Web, o [plano do Serviço de Aplicativo](https://azure.microsoft.com/pricing/details/app-service/) deve estar no nível **Básico**, **Standard** ou **Premium**.

#### <a name="sign-in-to-azure"></a>Entrar no Azure

1. Abra o [portal do Azure](https://portal.azure.com/) e vá até o aplicativo Web.

2. No menu à esquerda, selecione **Serviços de Aplicativos** e, em seguida, selecione o nome do aplicativo Web.

![Selecionar o aplicativo Web no portal do Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Verifique o tipo de preço

1. No painel de navegação à esquerda da página do aplicativo Web, role até a seção **Configurações** e selecione **Escalar verticalmente (plano do Serviço de Aplicativo)** .

    ![Menu da escala vertical no aplicativo Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Garanta que o aplicativo Web não está no nível **Gratuito** ou **Compartilhado**. O tipo de preço atual do aplicativo Web está realçado por uma caixa azul escuro.

    ![Verificar o tipo de preço do aplicativo Web](media/solution-deployment-guide-geo-distributed/image35.png)

Não há suporte para o SSL personalizado nos tipos de preço **Gratuito** ou **Compartilhado**. Para aumentar a escala, siga as etapas da próxima seção na página **Escolher o tipo de preço** e vá para [Carregar e associar o certificado SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Escalar verticalmente seu Plano do Serviço de Aplicativo

1. Selecione uma das camadas **Básico**, **Standard** ou **Premium**.

2. Selecione **Selecionar**.

![Escolher o tipo de preço do seu aplicativo Web](media/solution-deployment-guide-geo-distributed/image36.png)

A operação de escala estará concluída quando a notificação for exibida.

![Escalar verticalmente a notificação](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Associar o certificado SSL e mesclar certificados intermediários

Mescle múltiplos certificados na cadeia.

1. **Abra cada certificado** recebido em um editor de texto.

2. Crie um arquivo para o certificado mesclado com o nome de *mergedcertificate.crt*. Em um editor de texto, copie o conteúdo de cada certificado para esse arquivo. A ordem de seus certificados deve seguir a ordem na cadeia de certificados, começando com o seu certificado e terminando com o certificado raiz. Ela se parece com o seguinte exemplo:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Exportar o certificado para PFX

Exporte o certificado SSL mesclado com a chave privada gerada pelo certificado.

Um arquivo de chave privada será criado via OpenSSL. Para exportar o certificado para o PFX, execute o seguinte comando e substitua os espaços reservados `<private-key-file>` e `<merged-certificate-file>` pelo caminho da chave privada e o arquivo de certificado mesclado:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Quando solicitado, defina uma senha de exportação para carregar seu certificado SSL para o Serviço de Aplicativo posteriormente.

Quando o IIS ou o **Certreq.exe** forem usados para gerar a solicitação de certificado, instale o certificado em um computador local e, em seguida, [exporte o certificado para PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

#### <a name="upload-the-ssl-certificate"></a>Carregar o certificado SSL

1. No painel de navegação esquerdo do aplicativo Web, selecione **configurações de SSL**.

2. Selecione **Carregar Certificado**.

3. Em **Arquivo de Certificado PFX**, selecione arquivo PFX.

4. Em **Senha do certificado**, digite a senha criada ao exportar o arquivo PFX.

5. Escolha **Carregar**.

    ![Carregar o certificado SSL](media/solution-deployment-guide-geo-distributed/image38.png)

Quando o Serviço de Aplicativo terminar de carregar o certificado, ele aparecerá na página **Configurações de SSL**.

![Configurações de SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Associar o certificado SSL

1. Na seção **Associações SSL**, selecione **Adicionar associação**.

    > [!Note]  
    >  Caso o certificado tenha sido atualizado, mas não apareça nos nomes de domínio no menu suspenso **Nome do host**, tente atualizar a página do navegador.

2. Na página **Adicionar Associação SSL**, use os menus suspensos para selecionar o nome de domínio a ser protegido e o certificado a ser usado.

3. Em **Tipo SSL**, selecione se deseja usar [**SNI (Indicação de Nome de Servidor)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) ou SSL baseado em IP.

    - **SSL com base em SNI**: Várias associações SSL baseadas em SNI podem ser adicionadas. Esta opção permite que vários certificados SSL protejam vários domínios no mesmo endereço IP. Navegadores mais modernos (incluindo Internet Explorer, Chrome, Firefox e Opera) dão suporte ao SNI (encontre informações de suporte ao navegador mais abrangentes em [Indicação de Nome de Servidor](https://wikipedia.org/wiki/Server_Name_Indication)).

    - **SSL baseado em IP**: Apenas uma associação SSL baseada em IP pode ser adicionada. Esta opção permite apenas um certificado SSL para proteger um endereço IP público dedicado. Para proteger vários domínios, proteja todos usando o mesmo certificado SSL. O SSL baseado em IP é a opção tradicional para a associação de SSL.

4. Selecione **Adicionar Associação**.

    ![Adicionar associação SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Quando o Serviço de Aplicativo terminar de carregar o certificado, ele aparecerá nas seções de **associações SSL**.

![Atualização concluída das associações SSL](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Remapear um registro para IP SSL

Se o SSL baseado em IP não for usado no aplicativo Web, pule para [Testar o HTTPS para seu domínio personalizado](/azure/app-service/app-service-web-tutorial-custom-ssl).

Por padrão, o aplicativo Web usa um endereço IP público compartilhado. Quando o certificado estiver associado ao SSL baseado em IP, o Serviço de Aplicativo criará um novo endereço IP dedicado para o aplicativo Web.

Quando um registro A for mapeado para o aplicativo Web, o registro de domínio deve ser atualizado com o endereço IP dedicado.

A página **Domínio personalizado** é atualizada com o novo endereço IP dedicado. Copie esse [endereço IP](/azure/app-service/app-service-web-tutorial-custom-domain) e, em seguida, remapeie o [registro A](/azure/app-service/app-service-web-tutorial-custom-domain) para esse novo endereço IP.

#### <a name="test-https"></a>Testar HTTPS

Em navegadores diferentes, acesse `https://<your.custom.domain>` para garantir que o aplicativo Web seja atendido.

![Navegar até o aplicativo Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Erros de validação de certificado podem ocorrer devido aos seguintes motivos: um certificado autoassinado; ou certificados intermediários podem ter sido deixados para ser desativados na exportação para o arquivo PFX.

#### <a name="enforce-https"></a>Impor HTTPS

Por padrão, todos podem acessar o aplicativo Web usando o HTTP. Todas as solicitações HTTP à porta HTTPS podem ser redirecionadas.

Na página do aplicativo Web, selecione **Configurações do SL**. Depois, em **HTTPS somente**, selecione **Ligado**.

![Impor HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Quando a operação for concluída, vá até alguma das URLs HTTP que aponte para seu aplicativo. Por exemplo:

- https://<app_name>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Impor o TLS 1.1/1.2

O aplicativo permite o protocolo [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 por padrão, o que não é mais considerado seguro pelos padrões do setor (como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)). Para impor versões superiores do TLS, execute estas etapas:

1. Na página do aplicativo Web, na navegação à esquerda, selecione **Configurações SSL**.

2. Em **Versão do TLS**, selecione a versão mínima do TLS.

    ![Impor o TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Criar um perfil do Gerenciador de Tráfego

1. Escolha **Criar um recurso** > **Rede** > **Perfil do Gerenciador de Tráfego** > **Criar**.

2. Em **Criar perfil do Gerenciador de Tráfego**, preencha os seguintes campos:

    1. Em **Nome**, forneça um nome para o perfil. Esse nome deve ser exclusivo na zona de tráfego manager.net e resultará no nome DNS trafficmanager.net, que é usado para acessar o perfil do Gerenciador de Tráfego.

    2. Em **Método de roteamento**, selecione o **Método de roteamento geográfico**.

    3. Em **Assinatura**, selecione a assinatura sob a qual o perfil deve ser criado.

    4. Em **Grupo de Recursos**, crie um novo grupo de recursos no qual colocar esse perfil.

    5. Em **Local do grupo de recursos**, selecione o local do grupo de recursos. Essa configuração refere-se ao local do grupo de recursos e não tem impacto no perfil do Gerenciador de Tráfego implantado globalmente.

    6. Selecione **Criar**.

    7. Quando a implantação global do perfil do Gerenciador de Tráfego estiver concluída, ela será listada no respectivo grupo de recursos como um dos recursos.

        ![Grupos de recursos na criação de perfil do Gerenciador de Tráfego](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos de extremidade do Gerenciador de Tráfego

1. Na barra de pesquisa do portal, pesquise o nome do **Perfil do Gerenciador de Tráfego** criado na seção anterior e selecione o perfil do gerenciador de tráfego nos resultados exibidos.

2. Em **Perfil do Gerenciador de Tráfego**, na seção **Configurações**, selecione **Pontos de extremidade**.

3. Selecione **Adicionar**.

4. Adicione o ponto de extremidade do Azure Stack Hub.

5. Para **Tipo**, selecione **Ponto de extremidade externo**.

6. Forneça um **Nome** para esse ponto de extremidade, idealmente o nome do Azure Stack Hub.

7. Para o nome de domínio totalmente qualificado (**FQDN**), use a URL externa para o aplicativo Web do Azure Stack Hub.

8. Em Mapeamento geográfico, selecione uma região/continente em que o recurso esteja localizado. Por exemplo, **Europa**.

9. No menu suspenso País/Região exibido, selecione o país que se aplica a esse ponto de extremidade. Por exemplo, **Alemanha**.

10. Mantenha a opção **Adicionar como desabilitado** desmarcada.

11. Selecione **OK**.

12. Para adicionar o Ponto de Extremidade do Azure:

    1. Em **Tipo**, selecione **Ponto de extremidade do Azure**.

    2. Forneça um **Nome** para o ponto de extremidade.

    3. Para **Tipo de recurso de destino**, selecione **Serviço de Aplicativo**.

    4. Para **Recurso de destino**, selecione **Escolher um serviço de aplicativo** para mostrar a lista dos Aplicativos Web sob a mesma assinatura. Em **Recursos**, escolha o Serviço de aplicativo usado como o primeiro ponto de extremidade.

13. Em Mapeamento geográfico, selecione uma região/continente em que o recurso esteja localizado. Por exemplo, **América do Norte/América Central/Caribe.**

14. No menu suspenso País/Região que é exibido, deixe esse ponto em branco para selecionar todo o agrupamento regional acima.

15. Mantenha a opção **Adicionar como desabilitado** desmarcada.

16. Selecione **OK**.

    > [!Note]  
    >  Crie pelo menos um ponto de extremidade com um escopo geográfico de Todos (Mundo) para servir como o ponto de extremidade padrão do recurso.

17. Quando a adição de ambos os pontos de extremidade estiver concluída, eles serão exibidos no **Perfil do Gerenciador de Tráfego**, juntamente com seu status de monitoramento como **Online**.

    ![Status do ponto de extremidade do perfil do Gerenciador de Tráfego](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>As empresas globais contam com os recursos de distribuição geográfica do Azure

Direcionar o tráfego de dados por meio do Gerenciador de Tráfego do Azure e pontos de extremidade específicos de geografia permite que as empresas globais sigam as normas regionais e mantenham os dados em conformidade e seguros, o que é crucial para o sucesso de regiões de negócios locais e remotos.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).
