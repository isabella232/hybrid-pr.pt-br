---
title: Implantar o aplicativo híbrido com dados locais que dimensionam entre nuvem
description: Saiba como implantar um aplicativo que usa dados locais e dimensiona entre nuvem usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909786"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Implantar o aplicativo híbrido com dados locais que dimensionam entre nuvem

Este guia de solução mostra como implantar um aplicativo híbrido que abrange o Hub do Azure e do Azure Stack e usa uma única fonte de dados local.

Usando uma solução de nuvem híbrida, você pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública. Seus desenvolvedores também podem aproveitar o ecossistema do desenvolvedor da Microsoft e aplicar suas habilidades aos ambientes locais e na nuvem.

## <a name="overview-and-assumptions"></a>Visão geral e suposições

Siga este tutorial para configurar um fluxo de trabalho que permite aos desenvolvedores implantar um aplicativo Web idêntico em uma nuvem pública e em uma nuvem privada. Este aplicativo pode acessar uma rede roteável que não seja da Internet hospedada na nuvem privada. Esses aplicativos Web são monitorados e quando há um pico no tráfego, um programa modifica os registros DNS para redirecionar o tráfego para a nuvem pública. Quando o tráfego cai para o nível antes do pico, o tráfego é roteado de volta para a nuvem privada.

Este tutorial cobre as seguintes tarefas:

> [!div class="checklist"]
> - Implante um servidor de banco de dados SQL Server conectado por híbrido.
> - Conecte um aplicativo Web no Azure global a uma rede híbrida.
> - Configure o DNS para dimensionamento entre nuvem.
> - Configurar certificados SSL para dimensionamento entre nuvem.
> - Configure e implante o aplicativo Web.
> - Crie um perfil do Gerenciador de tráfego e configure-o para o dimensionamento entre nuvem.
> - Configure Application Insights monitoramento e alertas para aumentar o tráfego.
> - Configure a alternância automática de tráfego entre o Azure global e o Azure Stack Hub.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

### <a name="assumptions"></a>Suposições

Este tutorial pressupõe que você tenha um conhecimento básico do Azure global e do Azure Stack Hub. Se você quiser saber mais antes de iniciar o tutorial, examine estes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave de Hub de Azure Stack](/azure-stack/operator/azure-stack-overview.md)

Este tutorial também pressupõe que você tenha uma assinatura do Azure. Se você não tiver uma assinatura, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar essa solução, verifique se você atende aos seguintes requisitos:

- Um Kit de Desenvolvimento do Azure Stack (ASDK) ou uma assinatura em um sistema integrado de Hub Azure Stack. Para implantar o ASDK, siga as instruções em [implantar o ASDK usando o instalador](/azure-stack/asdk/asdk-install.md).
- A instalação do hub de Azure Stack deve ter o seguinte instalado:
  - O serviço Azure App. Trabalhe com seu operador de Hub de Azure Stack para implantar e configurar o serviço de Azure App em seu ambiente. Este tutorial requer que o serviço de aplicativo tenha pelo menos uma (1) função de trabalho dedicada disponível.
  - Uma imagem do Windows Server 2016.
  - Um Windows Server 2016 com uma imagem Microsoft SQL Server.
  - Os planos e ofertas apropriados.
  - Um nome de domínio para seu aplicativo Web. Se você não tiver um nome de domínio, poderá comprar um de um provedor de domínio como GoDaddy, BlueHost e InMotion.
- Um certificado SSL para seu domínio de uma autoridade de certificação confiável como LetsEncrypt.
- Um aplicativo Web que se comunica com um banco de dados SQL Server e oferece suporte a Application Insights. Você pode baixar o aplicativo de exemplo [dotnetcore-SQLDB-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do github.
- Uma rede híbrida entre uma rede virtual do Azure e uma rede virtual do hub de Azure Stack. Para obter instruções detalhadas, consulte [Configurar a conectividade de nuvem híbrida com o Azure e o Hub de Azure Stack](solution-deployment-guide-connectivity.md).

- Um pipeline de CI/CD (integração contínua/implantação contínua) com um agente de compilação particular no Hub de Azure Stack. Para obter instruções detalhadas, consulte [Configurar a identidade de nuvem híbrida com o Azure e os aplicativos de Hub de Azure Stack](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Implantar um servidor de banco de dados SQL Server conectado por híbrido

1. Entre no portal do usuário do hub de Azure Stack.

2. No **painel**, selecione **Marketplace**.

    ![Azure Stack Marketplace do Hub](media/solution-deployment-guide-hybrid/image1.png)

3. No **Marketplace**, selecione **computação**e, em seguida, escolha **mais**. Em **mais**, selecione a **licença de SQL Server gratuita: SQL Server 2017 desenvolvedor na imagem do Windows Server** .

    ![Selecionar uma imagem de máquina virtual no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. Em **licença gratuita de SQL Server: SQL Server desenvolvedor 2017 no Windows Server**, selecione **criar**.

5. Em **noções básicas > definir configurações básicas**, forneça um **nome** para a VM (máquina virtual), um **nome de usuário** para o SA do SQL Server e uma **senha** para o SA.  Na lista suspensa **assinatura** , selecione a assinatura na qual você está implantando. Para **grupo de recursos**, use **escolher existente** e coloque a VM no mesmo grupo de recursos que o seu aplicativo Web de Azure Stack Hub.

    ![Definir configurações básicas para VM no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image3.png)

6. Em **tamanho**, escolha um tamanho para a VM. Para este tutorial, recomendamos A2_Standard ou uma DS2_V2_Standard.

7. Em **configurações > configurar recursos opcionais**, defina as seguintes configurações:

   - **Conta de armazenamento**: Crie uma nova conta se precisar de uma.
   - **Rede virtual**:

     > [!Important]  
     > Verifique se sua VM SQL Server está implantada na mesma rede virtual que os gateways de VPN.

   - **Endereço IP público**: Use as configurações padrão.
   - **Grupo de segurança de rede**: (NSG). Crie um novo NSG.
   - **Extensões e monitoramento**: Mantenha as configurações padrão.
   - **Conta de armazenamento de diagnóstico**: Crie uma nova conta se precisar de uma.
   - Selecione **OK** para salvar sua configuração.

     ![Configurar recursos de VM opcionais no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. Em **configurações de SQL Server**, defina as seguintes configurações:

   - Para **conectividade do SQL**, selecione **público (Internet)**.
   - Para **porta**, mantenha o padrão, **1433**.
   - Para **autenticação do SQL**, selecione **habilitar**.

     > [!Note]  
     > Quando você habilita a autenticação do SQL, ela deve ser preenchida automaticamente com as informações de "sqladmin" que você configurou em **noções básicas**.

   - Para o restante das configurações, mantenha os padrões. Selecione **OK**.

     ![Definir configurações de SQL Server no portal de usuário de Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. Em **Resumo**, examine a configuração da VM e, em seguida, selecione **OK** para iniciar a implantação.

    ![Resumo da configuração no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. Leva algum tempo para criar a nova VM. Você pode exibir o STATUS de suas VMs em **máquinas virtuais**.

    ![Status das máquinas virtuais no portal do usuário do hub de Azure Stack](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Criar aplicativos Web no Azure e no Hub de Azure Stack

O serviço de Azure App simplifica a execução e o gerenciamento de um aplicativo Web. Como o Hub de Azure Stack é consistente com o Azure, o serviço de aplicativo pode ser executado em ambos os ambientes. Você usará o serviço de aplicativo para hospedar seu aplicativo.

### <a name="create-web-apps"></a>Criar aplicativos Web

1. Crie um aplicativo Web no Azure seguindo as instruções em [gerenciar um plano do serviço de aplicativo no Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Certifique-se de colocar o aplicativo Web na mesma assinatura e grupo de recursos que a sua rede híbrida.

2. Repita a etapa anterior (1) no Hub Azure Stack.

### <a name="add-route-for-azure-stack-hub"></a>Adicionar rota para o Hub de Azure Stack

O serviço de aplicativo no Hub de Azure Stack deve ser roteável da Internet pública para permitir que os usuários acessem seu aplicativo. Se o Hub de Azure Stack estiver acessível pela Internet, anote o endereço IP ou a URL voltada para o público para o aplicativo Web do hub de Azure Stack.

Se você estiver usando um ASDK, poderá [configurar um mapeamento NAT estático](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o serviço de aplicativo fora do ambiente virtual.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Conectar um aplicativo Web no Azure a uma rede híbrida

Para fornecer conectividade entre o front-end da Web no Azure e o banco de dados SQL Server no Hub Azure Stack, o aplicativo Web deve estar conectado à rede híbrida entre o Azure e o Hub do Azure Stack. Para habilitar a conectividade, você terá que:

- Configure a conectividade ponto a site.
- Configure o aplicativo Web.
- Modifique o gateway de rede local no Hub Azure Stack.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configurar a rede virtual do Azure para conectividade ponto a site

O gateway de rede virtual no lado do Azure da rede híbrida deve permitir conexões ponto a site a serem integradas ao serviço Azure App.

1. No Azure, vá para a página gateway de rede virtual. Em **configurações**, selecione **configuração ponto a site**.

    ![Opção ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Selecione **Configurar agora** para configurar o ponto a site.

    ![Iniciar configuração de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Na página configuração **ponto a site** , insira o intervalo de endereços IP privado que você deseja usar no **pool de endereços**.

   > [!Note]  
   > Certifique-se de que o intervalo especificado não se sobrepõe a nenhum dos intervalos de endereços já usados por sub-redes nos componentes globais do Azure ou do hub de Azure Stack da rede híbrida.

   Em **tipo de túnel**, desmarque a **VPN IKEv2**. Selecione **salvar** para concluir a configuração de ponto a site.

   ![Configurações de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrar o aplicativo de serviço de Azure App com a rede híbrida

1. Para conectar o aplicativo à VNet do Azure, siga as instruções em [integração vnet necessária do gateway](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Vá para **configurações** para o plano do serviço de aplicativo que hospeda o aplicativo Web. Em **Configurações**, selecione **Rede**.

    ![Configurar a rede para o plano do serviço de aplicativo](media/solution-deployment-guide-hybrid/image11.png)

3. Em **integração VNET**, selecione **clique aqui para gerenciar**.

    ![Gerenciar a integração VNET para o plano do serviço de aplicativo](media/solution-deployment-guide-hybrid/image12.png)

4. Selecione a VNET que você deseja configurar. Em **endereços IP roteados para VNET**, insira o intervalo de endereços IP para a vnet do Azure, a vnet do Hub de Azure Stack e os espaços de endereço ponto a site. Selecione **salvar** para validar e salvar essas configurações.

    ![Intervalos de endereços IP a serem roteados na integração de rede virtual](media/solution-deployment-guide-hybrid/image13.png)

Para saber mais sobre como o serviço de aplicativo se integra ao Azure VNets, consulte [integrar seu aplicativo a uma rede virtual do Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Configurar a rede virtual do hub de Azure Stack

O gateway de rede local na rede virtual do hub de Azure Stack precisa ser configurado para rotear o tráfego do intervalo de endereços de ponto a site do serviço de aplicativo.

1. No Hub Azure Stack, vá para **Gateway de rede local**. Em **Configurações**, escolha **Configuração**.

    ![Opção de configuração de gateway no gateway de rede local do Hub Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. Em **espaço de endereço**, insira o intervalo de endereços de ponto a site para o gateway de rede virtual no Azure.

    ![Espaço de endereço ponto a site no gateway de rede local do Hub Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. Selecione **salvar** para validar e salvar a configuração.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configurar o DNS para dimensionamento entre nuvem

Ao configurar corretamente o DNS para aplicativos de nuvem cruzada, os usuários podem acessar as instâncias globais do Azure e do hub de Azure Stack do seu aplicativo Web. A configuração de DNS para este tutorial também permite que o Gerenciador de tráfego do Azure encaminhe o tráfego quando a carga aumenta ou diminui.

Este tutorial usa o DNS do Azure para gerenciar o DNS porque os domínios do serviço de aplicativo não funcionarão.

### <a name="create-subdomains"></a>Criar subdomínios

Como o Gerenciador de tráfego depende de CNAMEs DNS, um subdomínio é necessário para rotear corretamente o tráfego para os pontos de extremidade. Para obter mais informações sobre os registros DNS e o mapeamento de domínio, consulte [mapear domínios com o Gerenciador de tráfego](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Para o ponto de extremidade do Azure, você criará um subdomínio que os usuários podem usar para acessar seu aplicativo Web. Para este tutorial, pode usar **app.Northwind.com**, mas você deve personalizar esse valor com base em seu próprio domínio.

Você também precisará criar um subdomínio com um registro A para o ponto de extremidade do Hub Azure Stack. Você pode usar **azurestack.Northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Configurar um domínio personalizado no Azure

1. Adicione o nome de host **app.Northwind.com** ao aplicativo Web do Azure [mapeando um CNAME para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Configurar domínios personalizados no Hub de Azure Stack

1. Adicione o nome de host **azurestack.Northwind.com** ao aplicativo Web Hub de Azure Stack [mapeando um registro a para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Use o endereço IP roteável da Internet para o aplicativo do serviço de aplicativo.

2. Adicione o nome de host **app.Northwind.com** ao aplicativo Web Hub de Azure Stack [mapeando um CNAME para Azure app serviço](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Use o nome de host que você configurou na etapa anterior (1) como o destino para o CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configurar certificados SSL para dimensionamento entre nuvem

É importante garantir que os dados confidenciais coletados pelo seu aplicativo Web sejam protegidos em trânsito e quando armazenados no banco de dados SQL.

Você configurará seus aplicativos Web do Azure e do Azure Stack Hub para usar certificados SSL para todo o tráfego de entrada.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Adicionar SSL ao Azure e ao Hub de Azure Stack

Para adicionar SSL ao Azure:

1. Certifique-se de que o certificado SSL obtido é válido para o subdomínio que você criou. (Não há problema em usar certificados curinga.)

2. No Azure, siga as instruções nas seções **preparar seu aplicativo Web** e **associar seu certificado SSL** do artigo [associar um certificado SSL personalizado existente a aplicativos Web do Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) . Selecione **SSL baseado em SNI** como o **tipo de SSL**.

3. Redirecionar todo o tráfego para a porta HTTPS. Siga as instruções na seção **impor https** do artigo [associar um certificado SSL personalizado existente a aplicativos Web do Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .

Para adicionar SSL ao Hub Azure Stack:

1. Repita as etapas a 1-3 que você usou para o Azure.

## <a name="configure-and-deploy-the-web-app"></a>Configurar e implantar o aplicativo Web

Você configurará o código do aplicativo para relatar telemetria para a instância de Application Insights correta e configurará os aplicativos Web com as cadeias de conexão corretas. Para saber mais sobre Application Insights, confira [o que é Application insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Adicionar Application Insights

1. Abra seu aplicativo Web no Microsoft Visual Studio.

2. [Adicione Application insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application insights usa para criar alertas quando o tráfego da Web aumenta ou diminui.

### <a name="configure-dynamic-connection-strings"></a>Configurar cadeias de conexão dinâmicas

Cada instância do aplicativo Web usará um método diferente para se conectar ao banco de dados SQL. O aplicativo no Azure usa o endereço IP privado da VM SQL Server e o aplicativo no Hub Azure Stack usa o endereço IP público da VM SQL Server.

> [!Note]  
> Em um sistema integrado de Hub Azure Stack, o endereço IP público não deve ser roteável pela Internet. Em um ASDK, o endereço IP público não é roteável fora do ASDK.

Você pode usar variáveis de ambiente do serviço de aplicativo para passar uma cadeia de conexão diferente para cada instância do aplicativo.

1. Abra o aplicativo no Visual Studio.

2. Abra Startup.cs e localize o seguinte bloco de código:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Substitua o bloco de código anterior pelo código a seguir, que usa uma cadeia de conexão definida no *appsettings.jsno* arquivo:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Definir configurações do aplicativo do serviço de aplicativo

1. Crie cadeias de conexão para o Azure e o Hub de Azure Stack. As cadeias de caracteres devem ser as mesmas, exceto os endereços IP que são usados.

2. No Azure e no Hub de Azure Stack, adicione a cadeia de conexão apropriada [como uma configuração de aplicativo](https://docs.microsoft.com/azure/app-service/web-sites-configure) no aplicativo Web, usando `SQLCONNSTR\_` como um prefixo no nome.

3. **Salve** as configurações do aplicativo Web e reinicie o aplicativo.

## <a name="enable-automatic-scaling-in-global-azure"></a>Habilitar o dimensionamento automático no Azure global

Quando você cria seu aplicativo Web em um ambiente do serviço de aplicativo, ele começa com uma instância. Você pode escalar horizontalmente automaticamente para adicionar instâncias para fornecer mais recursos de computação para seu aplicativo. Da mesma forma, você pode dimensionar e reduzir automaticamente o número de instâncias de que seu aplicativo precisa.

> [!Note]  
> Você precisa ter um plano do serviço de aplicativo para configurar o scale out e reduzir horizontalmente. Se você não tiver um plano, crie um antes de iniciar as próximas etapas.

### <a name="enable-automatic-scale-out"></a>Habilitar expansão automática

1. No Azure, localize o plano do serviço de aplicativo para os sites que você deseja escalar horizontalmente e selecione **escalar horizontalmente (plano do serviço de aplicativo)**.

    ![Escalar horizontalmente Azure App serviço](media/solution-deployment-guide-hybrid/image16.png)

2. Selecione **Habilitar dimensionamento automático**.

    ![Habilitar dimensionamento automático no serviço Azure App](media/solution-deployment-guide-hybrid/image17.png)

3. Insira um nome para o **nome da configuração de dimensionamento automático**. Para a regra de dimensionamento automático **padrão** , selecione **escala com base em uma métrica**. Defina os **limites da instância** como **mínimo: 1**, **máximo: 10**e **padrão: 1**.

    ![Configurar o dimensionamento automático no serviço Azure App](media/solution-deployment-guide-hybrid/image18.png)

4. Selecione **+ Adicionar uma regra**.

5. Em **origem da métrica**, selecione **recurso atual**. Use os critérios e as ações a seguir para a regra.

#### <a name="criteria"></a>Critérios

1. Em **agregação de tempo,** selecione **média**.

2. Em **nome da métrica**, selecione **percentual de CPU**.

3. Em **operador**, selecione **maior que**.

   - Defina o **limite** como **50**.
   - Defina a **duração** como **10**.

#### <a name="action"></a>Ação

1. Em **operação**, selecione **aumentar contagem por**.

2. Defina a **contagem de instâncias** como **2**.

3. Defina o **resfriamento** como **5**.

4. Selecione **Adicionar**.

5. Selecione **+ Adicionar uma regra**.

6. Em **origem da métrica**, selecione **recurso atual.**

   > [!Note]  
   > O recurso atual conterá o nome/GUID do plano do serviço de aplicativo e as listas suspensas **tipo de recurso** e **recurso** não estarão disponíveis.

### <a name="enable-automatic-scale-in"></a>Habilitar escala automática em

Quando o tráfego diminui, o aplicativo Web do Azure pode reduzir automaticamente o número de instâncias ativas para reduzir os custos. Essa ação é menos agressiva do que a expansão e minimiza o impacto nos usuários do aplicativo.

1. Vá para a condição de expansão **padrão** e, em seguida, selecione **+ Adicionar uma regra**. Use os critérios e as ações a seguir para a regra.

#### <a name="criteria"></a>Critérios

1. Em **agregação de tempo,** selecione **média**.

2. Em **nome da métrica**, selecione **percentual de CPU**.

3. Em **operador**, selecione **menor que**.

   - Defina o **limite** como **30**.
   - Defina a **duração** como **10**.

#### <a name="action"></a>Ação

1. Em **operação**, selecione **diminuir contagem por**.

   - Defina a **contagem de instâncias** como **1**.
   - Defina o **resfriamento** como **5**.

2. Selecione **Adicionar**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Criar um perfil do Gerenciador de tráfego e configurar o dimensionamento entre nuvens

Crie um perfil do Gerenciador de tráfego no Azure e, em seguida, configure pontos de extremidade para habilitar o dimensionamento entre nuvens.

### <a name="create-traffic-manager-profile"></a>Criar perfil do Gerenciador de tráfego

1. Selecione **Criar um recurso**.
2. Selecione **rede**.
3. Selecione **perfil do Gerenciador de tráfego** e defina as seguintes configurações:

   - Em **nome**, insira um nome para o seu perfil. Esse nome **deve** ser exclusivo na zona trafficmanager.net e é usado para criar um novo nome DNS (por exemplo, northwindstore.trafficmanager.net).
   - Para o **método de roteamento**, selecione o **peso**.
   - Para **assinatura**, selecione a assinatura na qual você deseja criar esse perfil.
   - Em **grupo de recursos**, crie um novo grupo de recursos para esse perfil.
   - Em **Local do grupo de recursos**, selecione o local do grupo de recursos. Essa configuração refere-se ao local do grupo de recursos e não tem impacto sobre o perfil do Gerenciador de tráfego implantado globalmente.

4. Selecione **Criar**.

    ![Criar perfil do Gerenciador de tráfego](media/solution-deployment-guide-hybrid/image19.png)

   Quando a implantação global do seu perfil do Gerenciador de tráfego for concluída, ela será mostrada na lista de recursos para o grupo de recursos no qual você o criou.

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos de extremidade do Gerenciador de Tráfego

1. Pesquise o perfil do Gerenciador de tráfego que você criou. Se você navegou até o grupo de recursos para o perfil, selecione o perfil.

2. No **perfil do Gerenciador de tráfego**, em **configurações**, selecione **pontos de extremidade**.

3. Selecione **Adicionar**.

4. Em **Adicionar ponto de extremidade**, use as seguintes configurações para Azure Stack Hub:

   - Para **tipo**, selecione **ponto de extremidade externo**.
   - Insira um **nome** para o ponto de extremidade.
   - Para **nome de domínio totalmente qualificado (FQDN) ou IP**, insira a URL externa para seu aplicativo Web de Hub de Azure Stack.
   - Para **peso**, mantenha o padrão, **1**. Esse peso resulta em todo o tráfego indo para esse ponto de extremidade se ele estiver íntegro.
   - Deixe **Adicionar como desabilitado** desmarcado.

5. Selecione **OK** para salvar o ponto de extremidade do Hub de Azure Stack.

Você configurará o ponto de extremidade do Azure em seguida.

1. No **perfil do Gerenciador de tráfego**, selecione **pontos de extremidade**.
2. Selecione **+ Adicionar**.
3. Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure:

   - Para **tipo**, selecione **ponto de extremidade do Azure**.
   - Insira um **nome** para o ponto de extremidade.
   - Para **tipo de recurso de destino**, selecione **serviço de aplicativo**.
   - Para **recurso de destino**, selecione **escolher um serviço de aplicativo** para ver uma lista de aplicativos Web na mesma assinatura.
   - Em **Recursos**, escolha o Serviço de Aplicativo que deseja adicionar como o primeiro ponto de extremidade.
   - Para **peso**, selecione **2**. Essa configuração resulta em todo o tráfego indo para esse ponto de extremidade se o ponto de extremidade primário não estiver íntegro ou se você tiver uma regra/alerta que redireciona o tráfego quando disparado.
   - Deixe **Adicionar como desabilitado** desmarcado.

4. Selecione **OK** para salvar o ponto de extremidade do Azure.

Depois que os dois pontos de extremidade são configurados, eles são listados no **perfil do Gerenciador de tráfego** quando você seleciona pontos de **extremidade**. O exemplo na captura de tela a seguir mostra dois pontos de extremidade, com status e informações de configuração para cada um.

![Pontos de extremidade no perfil do Gerenciador de tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Configurar Application Insights monitoramento e alertas

Aplicativo Azure insights permite monitorar seu aplicativo e enviar alertas com base nas condições que você configurar. Alguns exemplos são: o aplicativo está indisponível, está apresentando falhas ou está mostrando problemas de desempenho.

Você usará Application Insights métricas para criar alertas. Quando esses alertas forem disparados, a instância do aplicativo Web mudará automaticamente do Hub Azure Stack para o Azure para escalar horizontalmente e, em seguida, de volta para o Hub de Azure Stack para reduzir horizontalmente.

### <a name="create-an-alert-from-metrics"></a>Crie um alerta de métricas

Vá para o grupo de recursos deste tutorial e selecione a instância de Application Insights para abrir **Application insights**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Você usará essa exibição para criar um alerta de expansão e um alerta de redução.

### <a name="create-the-scale-out-alert"></a>Criar o alerta de expansão

1. Em **Configurar**, selecione **alertas (clássico)**.
2. Selecione **adicionar alerta de métrica (clássico)**.
3. Em **Adicionar regra**, defina as seguintes configurações:

   - Para **nome**, insira **intermitência na nuvem do Azure**.
   - Uma **Descrição** é opcional.
   - Em **Source**  >  **alerta de origem em**, selecione **métricas**.
   - Em **critérios**, selecione sua assinatura, o grupo de recursos para seu perfil do Gerenciador de tráfego e o nome do perfil do Gerenciador de tráfego para o recurso.

4. Para **métrica**, selecione **taxa de solicitação**.
5. Para **condição**, selecione **maior que**.
6. Para **limite**, digite **2**.
7. Para **período**, selecione **nos últimos 5 minutos**.
8. Em **notificar via**:
   - Marque a caixa de seleção de **proprietários, colaboradores e leitores de email**.
   - Insira seu endereço de email para **emails de administrador adicionais**.

9. Na barra de menus, selecione **salvar**.

### <a name="create-the-scale-in-alert"></a>Criar o alerta de redução

1. Em **Configurar**, selecione **alertas (clássico)**.
2. Selecione **adicionar alerta de métrica (clássico)**.
3. Em **Adicionar regra**, defina as seguintes configurações:

   - Para **nome**, digite **dimensionar novamente no Hub de Azure Stack**.
   - Uma **Descrição** é opcional.
   - Em **Source**  >  **alerta de origem em**, selecione **métricas**.
   - Em **critérios**, selecione sua assinatura, o grupo de recursos para seu perfil do Gerenciador de tráfego e o nome do perfil do Gerenciador de tráfego para o recurso.

4. Para **métrica**, selecione **taxa de solicitação**.
5. Para **condição**, selecione **menor que**.
6. Para **limite**, digite **2**.
7. Para **período**, selecione **nos últimos 5 minutos**.
8. Em **notificar via**:
   - Marque a caixa de seleção de **proprietários, colaboradores e leitores de email**.
   - Insira seu endereço de email para **emails de administrador adicionais**.

9. Na barra de menus, selecione **salvar**.

A captura de tela a seguir mostra os alertas de expansão e redução horizontal.

   ![Alertas de Application Insights (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Redirecionar o tráfego entre o Azure e o Hub de Azure Stack

Você pode configurar a alternância manual ou automática de seu tráfego de aplicativo Web entre o Azure e o Hub de Azure Stack.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Configurar a alternância manual entre o Azure e o Hub de Azure Stack

Quando o site atingir os limites que você configurar, você receberá um alerta. Use as etapas a seguir para redirecionar manualmente o tráfego para o Azure.

1. No portal do Azure, selecione seu perfil do Gerenciador de tráfego.

    ![Pontos de extremidade do Gerenciador de tráfego no portal do Azure](media/solution-deployment-guide-hybrid/image20.png)

2. Selecione **Pontos de extremidade**.
3. Selecione o **ponto de extremidade do Azure**.
4. Em **status**, selecione **habilitado**e, em seguida, selecione **salvar**.

    ![Habilitar o ponto de extremidade do Azure no portal do Azure](media/solution-deployment-guide-hybrid/image23.png)

5. Em **pontos** de extremidade para o perfil do Gerenciador de tráfego, selecione **ponto de extremidade externo**.
6. Em **status**, selecione **desabilitado**e, em seguida, selecione **salvar**.

    ![Desabilitar ponto de extremidade do hub de Azure Stack em portal do Azure](media/solution-deployment-guide-hybrid/image24.png)

Depois que os pontos de extremidade são configurados, o tráfego do aplicativo vai para o aplicativo Web de expansão do Azure em vez do aplicativo Web do hub de Azure Stack.

 ![Pontos de extremidade alterados no tráfego do aplicativo Web do Azure](media/solution-deployment-guide-hybrid/image25.png)

Para reverter o fluxo de volta para Azure Stack Hub, use as etapas anteriores para:

- Habilite o ponto de extremidade do hub de Azure Stack.
- Desabilite o ponto de extremidade do Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Configurar a alternância automática entre o Azure e o Hub de Azure Stack

Você também pode usar o monitoramento de Application Insights se seu aplicativo for executado em um ambiente sem [servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelo Azure functions.

Nesse cenário, você pode configurar Application Insights para usar um webhook que chama um aplicativo de funções. Esse aplicativo habilita ou desabilita automaticamente um ponto de extremidade em resposta a um alerta.

Use as etapas a seguir como um guia para configurar a alternância automática de tráfego.

1. Crie um aplicativo de funções do Azure.
2. Crie uma função disparada por HTTP.
3. Importe os SDKs do Azure para o Gerenciador de recursos, aplicativos Web e Gerenciador de tráfego.
4. Desenvolver código para:

   - Autentique em sua assinatura do Azure.
   - Use um parâmetro que alterna os pontos de extremidade do Gerenciador de tráfego para direcionar o tráfego para o Azure ou Azure Stack Hub.

5. Salve seu código e adicione a URL do aplicativo de funções com os parâmetros apropriados à seção **webhook** do Application insights configurações da regra de alerta.
6. O tráfego é redirecionado automaticamente quando um alerta de Application Insights é disparado.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).
