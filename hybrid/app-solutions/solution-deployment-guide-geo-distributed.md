---
title: Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Hub de Azure Stack
description: Saiba como direcionar o tráfego para pontos de extremidade específicos com uma solução de aplicativo distribuído geograficamente usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909848"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Direcionar o tráfego com um aplicativo distribuído geograficamente usando o Azure e o Hub de Azure Stack

Saiba como direcionar o tráfego para pontos de extremidade específicos com base em várias métricas usando o padrão de aplicativos distribuídos geograficamente. Criar um perfil do Gerenciador de tráfego com roteamento baseado em geográfico e configuração de ponto de extremidade garante que as informações sejam roteadas para pontos de extremidade com base em requisitos regionais, regulamentação corporativa e internacional e suas necessidades de dados.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Crie um aplicativo distribuído geograficamente.
> - Use o Gerenciador de tráfego para direcionar seu aplicativo.

## <a name="use-the-geo-distributed-apps-pattern"></a>Usar o padrão de aplicativos distribuídos geograficamente

Com o padrão distribuído geograficamente, seu aplicativo abrange regiões. Você pode padronizar para a nuvem pública, mas alguns de seus usuários podem exigir que seus dados permaneçam em sua região. Você pode direcionar os usuários para a nuvem mais adequada com base em seus requisitos.

### <a name="issues-and-considerations"></a>Problemas e considerações

#### <a name="scalability-considerations"></a>Considerações sobre escalabilidade

A solução que você criará com este artigo não é acomodar a escalabilidade. No entanto, se usado em combinação com outras soluções do Azure e locais, você pode acomodar os requisitos de escalabilidade. Para obter informações sobre como criar uma solução híbrida com dimensionamento automático por meio do Traffic Manager, consulte [criar soluções de dimensionamento entre nuvens com o Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Considerações sobre disponibilidade

Como é o caso de considerações de escalabilidade, essa solução não resolve diretamente a disponibilidade. No entanto, as soluções do Azure e locais podem ser implementadas nessa solução para garantir a alta disponibilidade para todos os componentes envolvidos.

### <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

- Sua organização tem ramificações internacionais que exigem políticas regionais personalizadas de segurança e distribuição.

- Cada um dos escritórios da sua organização recebe dados de funcionários, de negócios e de instalações, o que exige a atividade de relatórios por regulamentos locais e fusos horários.

- Os requisitos de alta escala são atendidos por meio da expansão horizontal de aplicativos com várias implantações de aplicativo em uma única região e entre regiões para lidar com requisitos de carga extremo.

### <a name="planning-the-topology"></a>Planejando a topologia

Antes de criar uma superfície de aplicativo distribuído, ele ajuda a conhecer as seguintes coisas:

- **Domínio personalizado para o aplicativo:** Qual é o nome de domínio personalizado que os clientes usarão para acessar o aplicativo? Para o aplicativo de exemplo, o nome de domínio personalizado é *www \. scalableasedemo.com.*

- **Domínio do Gerenciador de tráfego:** Um nome de domínio é escolhido ao criar um [perfil do Gerenciador de tráfego do Azure](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles). Esse nome é combinado com o sufixo *trafficmanager.net* para registrar uma entrada de domínio que é gerenciada pelo Gerenciador de tráfego. Para o aplicativo de exemplo, o nome escolhido é *scalable-ase-demo*. Como resultado, o nome de domínio completo que é gerenciado pelo Gerenciador de tráfego é *Scalable-ase-demo.trafficmanager.net*.

- **Estratégia para dimensionar a superfície do aplicativo:** Decida se a superfície do aplicativo será distribuída entre vários ambientes de serviço de aplicativo em uma única região, várias regiões ou uma combinação de ambas as abordagens. A decisão deve ser baseada nas expectativas de onde o tráfego do cliente será originado e no quão bem o restante da infraestrutura de back-end de suporte de um aplicativo pode ser dimensionada. Por exemplo, com um aplicativo sem estado de 100%, um aplicativo pode ser amplamente dimensionado usando uma combinação de vários ambientes de serviço de aplicativo por região do Azure, multiplicado por ambientes de serviço de aplicativo implantados em várias regiões do Azure. Com mais de 15 regiões globais do Azure disponíveis para escolher, os clientes podem realmente criar uma superfície de aplicativo em hiperescala mundial. Para o aplicativo de exemplo usado aqui, três ambientes de serviço de aplicativo foram criados em uma única região do Azure (EUA Central do Sul).

- **Convenção de nomenclatura para os ambientes do serviço de aplicativo:** Cada ambiente do serviço de aplicativo requer um nome exclusivo. Além de um ou dois ambientes de serviço de aplicativo, é útil ter uma Convenção de nomenclatura para ajudar a identificar cada ambiente de serviço de aplicativo. Para o aplicativo de exemplo usado aqui, foi usada uma Convenção de nomenclatura simples. Os nomes dos três ambientes de serviço de aplicativo são *fe1ase*, *fe2ase*e *fe3ase*.

- **Convenção de nomenclatura para os aplicativos:** Como várias instâncias do aplicativo serão implantadas, um nome será necessário para cada instância do aplicativo implantado. Com Ambiente do Serviço de Aplicativo para o Power apps, o mesmo nome de aplicativo pode ser usado em vários ambientes. Como cada ambiente do serviço de aplicativo tem um sufixo de domínio exclusivo, os desenvolvedores podem optar por reutilizar exatamente o mesmo nome de aplicativo em cada ambiente. Por exemplo, um desenvolvedor poderia ter aplicativos chamados da seguinte maneira: *MyApp.foo1.p.azurewebsites.net*, *MyApp.Foo2.p.azurewebsites.net*, *MyApp.foo3.p.azurewebsites.net*e assim por diante. Para o aplicativo usado aqui, cada instância de aplicativo tem um nome exclusivo. Os nomes de instância de aplicativo usados são *webfrontend1*, *webfrontend2* e *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="part-1-create-a-geo-distributed-app"></a>Parte 1: criar um aplicativo distribuído geograficamente

Nesta parte, você criará um aplicativo Web.

> [!div class="checklist"]
> - Crie aplicativos Web e publique.
> - Adicione código a Azure Repos.
> - Aponte a compilação do aplicativo para vários destinos de nuvem.
> - Gerencie e configure o processo de CD.

### <a name="prerequisites"></a>Pré-requisitos

Uma assinatura do Azure e a instalação do hub de Azure Stack são necessárias.

### <a name="geo-distributed-app-steps"></a>Etapas do aplicativo distribuído geograficamente

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Obter um domínio personalizado e configurar o DNS

Atualize o arquivo de zona DNS para o domínio. O Azure AD pode verificar a propriedade do nome de domínio personalizado. Use o [DNS do Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) para registros DNS do Azure/Office 365/externos no Azure ou adicione a entrada DNS em [um registrador de DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registre um domínio personalizado com um registrador público.

2. Entre no registrador de nome de domínio para o domínio. Um administrador aprovado pode ser solicitado a fazer as atualizações de DNS.

3. Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. A entrada DNS não altera comportamentos como roteamento de email ou hospedagem na Web.

### <a name="create-web-apps-and-publish"></a>Criar aplicativos Web e publicar

Configure a integração contínua híbrida/entrega contínua (CI/CD) para implantar o aplicativo Web no Azure e no Hub de Azure Stack e envie automaticamente as alterações para ambas as nuvens.

> [!Note]  
> O Hub de Azure Stack com imagens apropriadas agregadas para execução (Windows Server e SQL) e implantação do serviço de aplicativo são necessários. Para obter mais informações, consulte [pré-requisitos para implantar o serviço de aplicativo no Hub Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Adicionar código a Azure Repos

1. Entre no Visual Studio com uma **conta que tenha direitos de criação de projeto** em Azure repos.

    O CI/CD pode ser aplicado ao código do aplicativo e ao código de infraestrutura. Use [modelos de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) para desenvolvimento em nuvem privada e hospedado.

    ![Conectar-se a um projeto no Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Clone o repositório** criando e abrindo o aplicativo Web padrão.

    ![Clonar repositório no Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Criar implantação de aplicativo Web em ambas as nuvens

1. Edite o arquivo **WebApplication. csproj** : Select `Runtimeidentifier` e Add `win10-x64` . (Consulte a documentação [de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)

    ![Editar arquivo de projeto de aplicativo Web no Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Faça check-in no código para Azure Repos** usando Team Explorer.

3. Confirme se o **código do aplicativo** foi verificado Azure repos.

### <a name="create-the-build-definition"></a>Criar a definição de compilação

1. **Entre no Azure pipelines** para confirmar a capacidade de criar definições de compilação.

2. Adicionar `-r win10-x64` código. Essa adição é necessária para disparar uma implantação independente com o .NET Core.

    ![Adicionar código à definição de compilação no Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Execute a compilação**. O processo de [compilação de implantação independente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefatos que podem ser executados no Azure e no Hub de Azure Stack.

#### <a name="using-an-azure-hosted-agent"></a>Usando um agente hospedado do Azure

Usar um agente hospedado no Azure Pipelines é uma opção conveniente para criar e implantar aplicativos Web. A manutenção e as atualizações são executadas automaticamente pelo Microsoft Azure, o que permite desenvolvimento ininterrupto, teste e implantação.

### <a name="manage-and-configure-the-cd-process"></a>Gerenciar e configurar o processo de CD

Azure DevOps Services fornecer um pipeline altamente configurável e gerenciável para versões para vários ambientes, como desenvolvimento, preparo, QA e ambientes de produção; incluindo a necessidade de aprovações em estágios específicos.

## <a name="create-release-definition"></a>Criar definição de versão

1. Selecione o botão de **adição** para adicionar uma nova versão na guia **versões** na seção **Build e versão** do Azure DevOps Services.

    ![Criar uma definição de versão no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Aplique o modelo de implantação do serviço de Azure App.

   ![Aplicar Azure App modelo de implantação de serviço no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. Em **Adicionar artefato**, adicione o artefato para o aplicativo de compilação na nuvem do Azure.

   ![Adicionar artefato à compilação na nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Na guia pipeline, selecione a **fase,** o link de tarefa do ambiente e defina os valores de ambiente de nuvem do Azure.

   ![Definir valores de ambiente de nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Defina o **nome do ambiente** e selecione a **assinatura do Azure** para o ponto de extremidade de nuvem do Azure.

      ![Selecione a assinatura do Azure para o ponto de extremidade de nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. Em **nome do serviço de aplicativo**, defina o nome do serviço de aplicativo do Azure necessário.

      ![Definir o nome do serviço de aplicativo do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Insira "Hosted VS2017" na **fila do agente** para o ambiente hospedado na nuvem do Azure.

      ![Definir fila do agente para o ambiente hospedado na nuvem do Azure no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. No menu implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente. Selecione **OK** para a **pasta local**.
  
      ![Selecione o pacote ou a pasta para o ambiente de serviço Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selecione o pacote ou a pasta para o ambiente de serviço Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Salve todas as alterações e volte para o **pipeline de liberação**.

    ![Salvar alterações no pipeline de liberação no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Adicione um novo artefato selecionando a compilação para o aplicativo de Hub de Azure Stack.

    ![Adicionar novo artefato para o aplicativo de Hub de Azure Stack no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Adicione mais um ambiente aplicando a implantação do serviço de Azure App.

    ![Adicionar ambiente à implantação de serviço Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Nomeie o novo ambiente Azure Stack Hub.

    ![Ambiente de nome na implantação do serviço Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Localize o ambiente de Hub de Azure Stack na guia **tarefa** .

    ![Ambiente de Hub de Azure Stack no Azure DevOps Services no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selecione a assinatura para o ponto de extremidade do hub de Azure Stack.

    ![Selecione a assinatura para o ponto de extremidade do hub de Azure Stack em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Defina o nome do aplicativo Web do hub de Azure Stack como o nome do serviço de aplicativo.

    ![Defina o nome do aplicativo Web Hub de Azure Stack no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selecione o Azure Stack agente de Hub.

    ![Selecione o agente de Hub de Azure Stack no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Na seção implantar serviço de Azure App, selecione o **pacote ou a pasta** válida para o ambiente. Selecione **OK** para a pasta local.

    ![Selecione a pasta para a implantação do serviço de Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selecione a pasta para a implantação do serviço de Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Na guia variável, adicione uma variável chamada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , defina seu valor como **true**e escopo para Azure Stack Hub.

    ![Adicionar variável à implantação de Azure App no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selecione o ícone de gatilho de implantação **contínua** em ambos os artefatos e habilite o gatilho de implantação **continua** .

    ![Selecione o gatilho de implantação contínua no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selecione o ícone condições de **pré-implantação** no ambiente de Hub de Azure Stack e defina o gatilho como **após a liberação.**

    ![Selecionar condições de pré-implantação no Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Salve todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis de ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) ao criar uma definição de versão a partir de um modelo. Essas configurações não podem ser modificadas nas configurações da tarefa; em vez disso, o item do ambiente pai deve ser selecionado para editar essas configurações.

## <a name="part-2-update-web-app-options"></a>Parte 2: atualizar opções do aplicativo Web

O [Serviço de Aplicativo do Azure](https://docs.microsoft.com/azure/app-service/overview) fornece um serviço de hospedagem na Web altamente escalonável e com aplicação automática de patches.

![Serviço de aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Mapeie um nome DNS personalizado existente para aplicativos Web do Azure.
> - Use um **registro CNAME** e um **registro** a para mapear um nome DNS personalizado para o serviço de aplicativo.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mapear um nome DNS personalizado existente para aplicativos Web do Azure

> [!Note]  
> Use um CNAME para todos os nomes DNS personalizados, exceto um domínio raiz (por exemplo, northwind.com).

Para migrar um site ativo e seu nome de domínio DNS para o Serviço de Aplicativo, consulte [Migrar um nome DNS ativo para o Serviço de Aplicativo do Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Pré-requisitos

Para concluir esta solução:

- [Crie um aplicativo do serviço de aplicativo](https://docs.microsoft.com/azure/app-service/)ou use um aplicativo criado para outra solução.

- Adquira um nome de domínio e verifique o acesso ao registro DNS para o provedor de domínio.

Atualize o arquivo de zona DNS para o domínio. O AD do Azure verificará a propriedade do nome de domínio personalizado. Use o [DNS do Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) para registros DNS do Azure/Office 365/externos no Azure ou adicione a entrada DNS em [um registrador de DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Registre um domínio personalizado com um registrador público.

- Entre no registrador de nome de domínio para o domínio. (Pode ser necessário um administrador aprovado para fazer atualizações de DNS).

- Atualize o arquivo de zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.

Por exemplo, para adicionar entradas DNS para northwindcloud.com e www \. northwindcloud.com, defina as configurações de DNS para o domínio raiz northwindcloud.com.

> [!Note]  
> Um nome de domínio pode ser adquirido usando o [portal do Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). Para mapear um nome DNS personalizado para um aplicativo Web, o [plano do Serviço de Aplicativo](https://azure.microsoft.com/pricing/details/app-service/) do aplicativo Web deve ser uma camada paga (**Compartilhado**, **Básico**, **Standard** ou **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Criar e mapear registros CNAME e A

#### <a name="access-dns-records-with-domain-provider"></a>Acessar registros DNS com o provedor de domínio

> [!Note]  
>  Use o DNS do Azure para configurar um nome DNS personalizado para aplicativos Web do Azure. Para obter mais informações, consulte [Usar o DNS do Azure para fornecer as configurações de domínio personalizadas para um serviço do Azure](https://docs.microsoft.com/azure/dns/dns-custom-domain).

1. Entre no site do provedor principal.

2. Localize a página para gerenciamento de registros DNS. Cada provedor de domínio tem sua própria interface de registros DNS. Procure áreas do site rotuladas como **Nome de Domínio**, **DNS** ou **Gerenciamento de Servidor de Nomes**.

A página de registros DNS pode ser exibida em **meus domínios**. Localize o link **arquivo de zona**nomeado, **registros DNS**ou **Configuração avançada**.

A captura de tela a seguir é um exemplo de uma página de registros DNS:

![Exemplo de página de registros DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. Em registrador de nomes de domínio, selecione **Adicionar ou criar** para criar um registro. Alguns provedores têm links diferentes para adicionar tipos de registro diferentes. Consulte a documentação do provedor.

2. Adicione um registro CNAME para mapear um subdomínio para o nome de host padrão do aplicativo.

   Para o \. exemplo de domínio northwindcloud.com da www, adicione um registro CNAME que mapeia o nome para `<app_name>.azurewebsites.net` .

Depois de adicionar o CNAME, a página de registros DNS é semelhante ao exemplo a seguir:

![Navegação no Portal para o aplicativo do Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Habilitar o mapeamento de registro CNAME no Azure

1. Em uma nova guia, entre no portal do Azure.

2. Acesse Serviços de Aplicativos.

3. Selecione aplicativo Web.

4. No painel de navegação à esquerda da página do aplicativo no portal do Azure, selecione **Domínios personalizados**.

5. Selecione o **+** ícone ao lado de **adicionar nome do host**.

6. Digite o nome de domínio totalmente qualificado, como `www.northwindcloud.com` .

7. Selecione **Validar**.

8. Se indicado, adicione outros registros de outros tipos ( `A` ou `TXT` ) aos registros DNS dos registradores de nome de domínio. O Azure fornecerá os valores e os tipos desses registros:

   a.  Um registro **A** a ser mapeado para o endereço IP do aplicativo.

   b.  Um registro **TXT** a ser mapeado para o nome do host padrão do aplicativo `<app_name>.azurewebsites.net`. O serviço de aplicativo usa esse registro somente no momento da configuração para verificar a propriedade de domínio personalizada. Após a verificação, exclua o registro TXT.

9. Conclua essa tarefa na guia registrador de domínio e revalide até que o botão **adicionar nome de host** seja ativado.

10. Verifique se o **tipo de registro hostname** está definido como **CNAME** (www.example.com ou qualquer subdomínio).

11. Selecione **Adicionar nome do host**.

12. Digite o nome de domínio totalmente qualificado, como `northwindcloud.com` .

13. Selecione **Validar**. A **adição** está ativada.

14. Verifique se o **tipo de registro hostname** está definido como **um registro** (example.com).

15. **Adicionar nome do host**.

    Pode levar algum tempo para que os novos nomes de host sejam refletidos na página **domínios personalizados** do aplicativo. Tente atualizar o navegador para atualizar os dados.
  
    ![Domínios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Se houver um erro, uma notificação de erro de verificação será exibida na parte inferior da página. ![Erro de verificação de domínio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  As etapas acima podem ser repetidas para mapear um domínio curinga ( \* . northwindcloud.com). Isso permite a adição de quaisquer subdomínios adicionais a esse serviço de aplicativo sem a necessidade de criar um registro CNAME separado para cada um. Siga as instruções do registrador para definir essa configuração.

#### <a name="test-in-a-browser"></a>Testar em um navegador

Navegue até os nomes DNS configurados anteriormente (por exemplo, `northwindcloud.com` ou `www.northwindcloud.com` ).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Parte 3: associar um certificado SSL personalizado

Nesta parte, iremos:

> [!div class="checklist"]
> - Associe o certificado SSL personalizado ao serviço de aplicativo.
> - Impor HTTPS para o aplicativo.
> - Automatizar a associação de certificado SSL com scripts.

> [!Note]  
> Se necessário, obtenha um certificado SSL do cliente no portal do Azure e vincule-o ao aplicativo Web. Para obter mais informações, consulte o [tutorial de certificados do serviço de aplicativo](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Pré-requisitos

Para concluir esta solução:

- [Crie um aplicativo do serviço de aplicativo.](https://docs.microsoft.com/azure/app-service/)
- [Mapeie um nome DNS personalizado para seu aplicativo Web.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Adquira um certificado SSL de uma autoridade de certificação confiável e use a chave para assinar a solicitação.

### <a name="requirements-for-your-ssl-certificate"></a>Requisitos para o certificado SSL

Para usar o certificado no Serviço de Aplicativo, o certificado deve atender a todos os seguintes requisitos:

- Assinado por uma autoridade de certificação confiável.

- Exportado como um arquivo PFX protegido por senha.

- Contém a chave privada com pelo menos 2048 bits de comprimento.

- Contém todos os certificados intermediários na cadeia de certificados.

> [!Note]  
> Os **certificados ECC (criptografia de curva elíptica)** funcionam com o serviço de aplicativo, mas não estão incluídos neste guia. Consulte uma autoridade de certificação para obter assistência na criação de certificados ECC.

#### <a name="prepare-the-web-app"></a>Preparar o aplicativo Web

Para associar um certificado SSL personalizado ao aplicativo Web, o [plano do serviço de aplicativo](https://azure.microsoft.com/pricing/details/app-service/) deve estar na camada **básica**, **Standard**ou **Premium** .

#### <a name="sign-in-to-azure"></a>Entrar no Azure

1. Abra o [portal do Azure](https://portal.azure.com/) e vá para o aplicativo Web.

2. No menu à esquerda, selecione **serviços de aplicativos**e, em seguida, selecione o nome do aplicativo Web.

![Selecione o aplicativo Web no portal do Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Verifique o tipo de preço

1. No painel de navegação à esquerda da página do aplicativo Web, role até a seção **configurações** e selecione **escalar verticalmente (plano do serviço de aplicativo)**.

    ![Menu de escala vertical no aplicativo Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Verifique se o aplicativo Web não está na camada **gratuita** ou **compartilhada** . A camada atual do aplicativo Web é realçada em uma caixa azul escura.

    ![Verificar tipo de preço no aplicativo Web](media/solution-deployment-guide-geo-distributed/image35.png)

Não há suporte para SSL personalizado na camada **gratuita** ou **compartilhada** . Para fazer up-scale, siga as etapas na próxima seção ou na página **escolha seu tipo de preço** e pule para [carregar e associar seu certificado SSL](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Escalar verticalmente seu Plano do Serviço de Aplicativo

1. Selecione uma das camadas **Básico**, **Standard** ou **Premium**.

2. Selecione **Selecionar**.

![Escolha o tipo de preço para seu aplicativo Web](media/solution-deployment-guide-geo-distributed/image36.png)

A operação de dimensionamento é concluída quando a notificação é exibida.

![Escalar verticalmente a notificação](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Associar seu certificado SSL e mesclar certificados intermediários

Mescle vários certificados na cadeia.

1. **Abra cada certificado** recebido em um editor de texto.

2. Crie um arquivo para o certificado mesclado chamado *mergedcertificate. CRT*. Em um editor de texto, copie o conteúdo de cada certificado para esse arquivo. A ordem de seus certificados deve seguir a ordem na cadeia de certificados, começando com o seu certificado e terminando com o certificado raiz. Ela se parece com o seguinte exemplo:

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

Um arquivo de chave privada é criado via OpenSSL. Para exportar o certificado para o PFX, execute o seguinte comando e substitua os espaços reservados `<private-key-file>` e `<merged-certificate-file>` pelo caminho da chave privada e pelo arquivo de certificado mesclado:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Quando solicitado, defina uma senha de exportação para carregar seu certificado SSL para o serviço de aplicativo mais tarde.

Quando o IIS ou **Certreq.exe** são usados para gerar a solicitação de certificado, instale o certificado em um computador local e, em seguida, [exporte o certificado para o pfx](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Carregar o certificado SSL

1. Selecione **configurações de SSL** no painel de navegação esquerdo do aplicativo Web.

2. Selecione **carregar certificado**.

3. No **arquivo de certificado pfx**, selecione arquivo PFX.

4. Em **senha do certificado**, digite a senha criada ao exportar o arquivo PFX.

5. Escolha **Carregar**.

    ![Carregar certificado SSL](media/solution-deployment-guide-geo-distributed/image38.png)

Quando o serviço de aplicativo termina de carregar o certificado, ele aparece na página **configurações de SSL** .

![Configurações SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Associar o certificado SSL

1. Na seção **associações SSL** , selecione **Adicionar Associação**.

    > [!Note]  
    >  Se o certificado tiver sido carregado, mas não aparecer em nomes de domínio na lista suspensa nome do **host** , tente atualizar a página do navegador.

2. Na página **Adicionar Associação SSL** , use os menus suspensos para selecionar o nome de domínio a ser protegido e o certificado a ser usado.

3. Em **Tipo SSL**, selecione se deseja usar [**SNI (Indicação de Nome de Servidor)**](https://en.wikipedia.org/wiki/Server_Name_Indication) ou SSL baseado em IP.

    - **SSL baseado em SNI**: várias associações SSL baseadas em SNI podem ser adicionadas. Esta opção permite que vários certificados SSL protejam vários domínios no mesmo endereço IP. Navegadores mais modernos (incluindo Internet Explorer, Chrome, Firefox e Opera) dão suporte ao SNI (encontre informações de suporte ao navegador mais abrangentes em [Indicação de Nome de Servidor](https://wikipedia.org/wiki/Server_Name_Indication)).

    - **SSL baseado em IP**: somente uma associação SSL com base em IP pode ser adicionada. Esta opção permite apenas um certificado SSL para proteger um endereço IP público dedicado. Para proteger vários domínios, proteja todos eles usando o mesmo certificado SSL. O SSL baseado em IP é a opção tradicional para associação SSL.

4. Selecione **Adicionar Associação**.

    ![Adicionar associação SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Quando o serviço de aplicativo termina de carregar o certificado, ele aparece nas seções **associações SSL** .

![Associações SSL finalizadas no carregamento](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Remapear o registro a para IP SSL

Se o SSL baseado em IP não for usado no aplicativo Web, pule para [testar HTTPS para seu domínio personalizado](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

Por padrão, o aplicativo Web usa um endereço IP público compartilhado. Quando o certificado é associado com SSL baseado em IP, o serviço de aplicativo cria um endereço IP novo e dedicado para o aplicativo Web.

Quando um registro A é mapeado para o aplicativo Web, o registro de domínio deve ser atualizado com o endereço IP dedicado.

A página de **domínio personalizada** é atualizada com o novo endereço IP dedicado. Copie esse [endereço IP](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)e remapeie o [registro](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) a desse novo endereço IP.

#### <a name="test-https"></a>Testar HTTPS

Em navegadores diferentes, vá para `https://<your.custom.domain>` para garantir que o aplicativo Web seja servido.

![navegar até o aplicativo Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Se ocorrerem erros de validação de certificado, um certificado autoassinado pode ser a causa, ou os certificados intermediários podem ter sido deixados para serem desativados ao exportar para o arquivo PFX.

#### <a name="enforce-https"></a>Impor o HTTPS

Por padrão, qualquer pessoa pode acessar o aplicativo Web usando HTTP. Todas as solicitações HTTP para a porta HTTPS podem ser redirecionadas.

Na página do aplicativo Web, selecione **configurações de SL**. Depois, em **HTTPS somente**, selecione **Ligado**.

![Impor HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Quando a operação for concluída, vá para qualquer uma das URLs HTTP que apontam para o aplicativo. Por exemplo:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Impor o TLS 1.1/1.2

O aplicativo permite o [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 por padrão, o que não é mais considerado seguro por padrões do setor (como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)). Para impor versões superiores do TLS, execute estas etapas:

1. Na página do aplicativo Web, no painel de navegação esquerdo, selecione **configurações de SSL**.

2. Em **versão TLS**, selecione a versão mínima do TLS.

    ![Impor o TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Criar um perfil do Gerenciador de Tráfego

1. Selecione **criar um recurso**  >  **rede**  >  **perfil do Gerenciador de tráfego**  >  **criar**.

2. Em **Criar perfil do Gerenciador de Tráfego**, preencha os seguintes campos:

    1. Em **nome**, forneça um nome para o perfil. Esse nome precisa ser exclusivo na zona de tráfego manager.net e resulta no nome DNS, trafficmanager.net, que é usado para acessar o perfil do Gerenciador de tráfego.

    2. Em **método de roteamento**, selecione o **método de roteamento geográfico**.

    3. Em **assinatura**, selecione a assinatura sob a qual este perfil será criado.

    4. Em **Grupo de Recursos**, crie um novo grupo de recursos no qual colocar esse perfil.

    5. Em **Local do grupo de recursos**, selecione o local do grupo de recursos. Essa configuração refere-se ao local do grupo de recursos e não tem impacto sobre o perfil do Gerenciador de tráfego implantado globalmente.

    6. Selecione **Criar**.

    7. Quando a implantação global do perfil do Gerenciador de tráfego for concluída, ela será listada no respectivo grupo de recursos como um dos recursos.

        ![Grupos de recursos em criar perfil do Gerenciador de tráfego](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos de extremidade do Gerenciador de Tráfego

1. Na barra de pesquisa do portal, procure o nome do **perfil do Gerenciador de tráfego** criado na seção anterior e selecione o perfil do Gerenciador de tráfego nos resultados exibidos.

2. No **perfil do Gerenciador de tráfego**, na seção **configurações** , selecione **pontos de extremidade**.

3. Selecione **Adicionar**.

4. Adicionando o ponto de extremidade do Hub Azure Stack.

5. Para **tipo**, selecione **ponto de extremidade externo**.

6. Forneça um **nome** para esse ponto de extremidade, idealmente o nome do Hub de Azure Stack.

7. Para o**FQDN**(nome de domínio totalmente qualificado), use a URL externa para o aplicativo Web do Hub de Azure Stack.

8. Em mapeamento geográfico, selecione uma região/continente onde o recurso está localizado. Por exemplo, **Europa.**

9. Na lista suspensa país/região que aparece, selecione o país que se aplica a esse ponto de extremidade. Por exemplo, **Alemanha**.

10. Mantenha a opção **Adicionar como desabilitado** desmarcada.

11. Selecione **OK**.

12. Adicionando o Ponto de Extremidade do Azure:

    1. Para **tipo**, selecione **ponto de extremidade do Azure**.

    2. Forneça um **nome** para o ponto de extremidade.

    3. Para **tipo de recurso de destino**, selecione **serviço de aplicativo**.

    4. Para **recurso de destino**, selecione **escolher um serviço de aplicativo** para mostrar a listagem dos aplicativos Web na mesma assinatura. Em **recurso**, escolha o serviço de aplicativo usado como o primeiro ponto de extremidade.

13. Em mapeamento geográfico, selecione uma região/continente onde o recurso está localizado. Por exemplo, **América do Norte/América Central/Caribe.**

14. Na lista suspensa país/região que aparece, deixe esse ponto em branco para selecionar todo o agrupamento regional acima.

15. Mantenha a opção **Adicionar como desabilitado** desmarcada.

16. Selecione **OK**.

    > [!Note]  
    >  Crie pelo menos um ponto de extremidade com um escopo geográfico de todos (mundo) para servir como o ponto de extremidade padrão para o recurso.

17. Quando a adição de ambos os pontos de extremidade for concluída, eles serão exibidos no **perfil do Gerenciador de tráfego** junto com seu status de monitoramento como **online**.

    ![Status do ponto de extremidade do perfil do Gerenciador de tráfego](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>O global Enterprise conta com os recursos de distribuição geográfica do Azure

Direcionar o tráfego de dados por meio do Gerenciador de tráfego do Azure e pontos de extremidade específicos de Geografia permite que as empresas globais sigam as normas regionais e mantenham os dados em conformidade e seguros, o que é crucial para o sucesso de locais de negócios locais e remotos.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).
