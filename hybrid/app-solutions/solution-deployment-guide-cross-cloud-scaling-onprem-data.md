---
title: Implantar o aplicativo híbrido com dados locais que escalam entre nuvens
description: Saiba como implantar um aplicativo que usa dados locais e escala entre nuvens usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895390"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Implantar o aplicativo híbrido com dados locais que escalam entre nuvens

Este guia de solução mostra como implantar um aplicativo híbrido que abranja o Azure e o Azure Stack Hub e use uma única fonte de dados local.

Usando uma solução de nuvem híbrida, você pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública. Seus desenvolvedores também podem aproveitar o ecossistema de desenvolvimento da Microsoft e aplicar suas habilidades aos ambientes locais e na nuvem.

## <a name="overview-and-assumptions"></a>Visão geral e suposições

Siga este tutorial para configurar um fluxo de trabalho que permita aos desenvolvedores implantar um aplicativo Web idêntico em uma nuvem pública e em uma nuvem privada. Esse aplicativo pode acessar uma rede roteável fora da Internet que esteja hospedada na nuvem privada. Esses aplicativos Web são monitorados e, quando há um pico de tráfego, um programa modifica os registros DNS para redirecionar o tráfego para a nuvem pública. Quando o tráfego cai para o nível anterior ao pico, ele é roteado de volta para a nuvem privada.

Este tutorial cobre as seguintes tarefas:

> [!div class="checklist"]
> - Implante um servidor de banco de dados do SQL Server com conexão híbrida.
> - Conecte um aplicativo Web no Azure global a uma rede híbrida.
> - Configure o DNS para escala entre nuvens.
> - Configure certificados SSL para escala entre nuvens.
> - Configure e implante o aplicativo Web.
> - Crie um perfil do Gerenciador de Tráfego e configure-o para escala entre nuvens.
> - Configure o monitoramento e os alertas do Application Insights para aumentar o tráfego.
> - Configure a alternância automática de tráfego entre o Azure global e o Azure Stack Hub.

> [!Tip]  
> ![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

### <a name="assumptions"></a>Suposições

Este tutorial pressupõe que você tenha um conhecimento básico do Azure global e do Azure Stack Hub. Se você quiser saber mais antes de iniciar o tutorial, confira estes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave do Azure Stack Hub](/azure-stack/operator/azure-stack-overview)

Este tutorial pressupõe que você tenha uma assinatura do Azure. Se você não tem uma assinatura, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar esta solução, verifique se você atende aos seguintes requisitos:

- Um ASDK (Kit de Desenvolvimento do Azure Stack) ou uma assinatura em um sistema integrado do Azure Stack Hub. Para implantar o ASDK, siga as instruções em [Implantar o ASDK usando o instalador](/azure-stack/asdk/asdk-install).
- A instalação do Azure Stack Hub deve ter instalado:
  - O Serviço de Aplicativo do Azure. Trabalhe com seu operador do Azure Stack Hub para implantar e configurar o Serviço de Aplicativo do Azure no seu ambiente. Este tutorial requer que o Serviço de Aplicativo tenha pelo menos uma (1) função de trabalho dedicada disponível.
  - Uma imagem do Windows Server 2016.
  - Um Windows Server 2016 com uma imagem do Microsoft SQL Server.
  - Os planos e as ofertas apropriados.
  - Um nome de domínio para seu aplicativo Web. Se você não tem um, é possível comprá-lo de um provedor de domínio, como GoDaddy, Bluehost e InMotion.
- Um certificado SSL para seu domínio de uma autoridade de certificação confiável, como o LetsEncrypt.
- Um aplicativo Web que se comunica com um banco de dados do SQL Server e oferece suporte ao Application Insights. Você pode baixar o aplicativo de exemplo [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do GitHub.
- Uma rede híbrida entre uma rede virtual do Azure e outra do Azure Stack Hub. Para obter instruções detalhadas, confira [Configurar a conectividade de nuvem híbrida com o Azure e o Azure Stack Hub](solution-deployment-guide-connectivity.md).

- Um pipeline de CI/CD (integração contínua/implantação contínua) com um agente de compilação particular no Azure Stack Hub. Para obter instruções detalhadas, confira [Configurar a identidade de nuvem híbrida com os aplicativos do Azure e do Azure Stack Hub](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Implantar um servidor de banco de dados do SQL Server com conexão híbrida

1. Entre no portal do usuário do Azure Stack Hub.

2. No **Dashboard**, selecione **Marketplace**.

    ![Marketplace do Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. Em **Marketplace**, selecione **Computação** e, em seguida, escolha **Mais**. Em **Mais**, selecione a **Licença gratuita do SQL Server: Imagem do SQL Server 2017 Developer no Windows Server**.

    ![Selecionar uma imagem de máquina virtual no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. Em **Licença gratuita do SQL Server: Em SQL Server 2017 Developer no Windows Server**, selecione **Criar**.

5. Em **Noções básicas > Definir configurações básicas**, forneça um **Nome** para a VM (máquina virtual), um **Nome de usuário** para o SA do SQL Server e uma **Senha** para o SA.  Na lista suspensa **Assinatura**, selecione a assinatura na qual você está efetuando a implantação. Para **Grupo de recursos**, use **Escolher existente** e coloque a VM no mesmo grupo de recursos que o seu aplicativo Web do Azure Stack Hub.

    ![Definir as configurações básicas para VM no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. Em **Tamanho**, escolha um para a sua VM. Para este tutorial, recomendamos A2_Standard ou DS2_V2_Standard.

7. Em **Configurações > Configurar recursos opcionais**, defina as seguintes configurações:

   - **Conta de armazenamento**: se precisar, crie outra conta.
   - **Rede virtual**:

     > [!Important]  
     > verifique se sua VM do SQL Server está implantada na mesma rede virtual que os gateways de VPN.

   - **Endereço IP público**: use as configurações padrão.
   - **Grupo de segurança de rede**: (NSG). Crie um NSG.
   - **Extensões e Monitoramento**: Mantenha as configurações padrão.
   - **Conta de armazenamento de diagnóstico**: se precisar, crie outra conta.
   - Selecione **OK** para salvar a configuração.

     ![Configurar recursos opcionais de VM no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. Em **Configurações do SQL Server**, defina as seguintes configurações:

   - Em **Conectividade do SQL**, selecione **Pública (Internet)** .
   - Em **Porta**, mantenha o padrão **1433**.
   - Em **Autenticação do SQL**, selecione **Habilitar**.

     > [!Note]  
     > Quando você habilita a autenticação do SQL, ela deve preencher automaticamente as informações de “SQLAdmin” que você configurou em **Noções básicas**.

   - Para o restante das configurações, mantenha as opções padrão. Selecione **OK**.

     ![Definir as configurações básicas do SQL Server no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. Em **Resumo**, examine a configuração da VM e, em seguida, selecione **OK** para iniciar a implantação.

    ![Resumo das configurações no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. A criação da VM leva algum tempo. Você pode exibir o STATUS de suas VMs em **Máquinas virtuais**.

    ![Status das máquinas virtuais no portal do usuário do Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Criar aplicativos Web no Azure e no Azure Stack Hub

O Serviço de Aplicativo do Azure simplifica a execução e o gerenciamento de um aplicativo Web. Como o Azure Stack Hub é consistente com o Azure, o Serviço de Aplicativo pode ser executado em ambos os ambientes. Você usará o Serviço de Aplicativo para hospedar seu aplicativo.

### <a name="create-web-apps"></a>Criar aplicativos Web

1. Crie um aplicativo Web no Azure seguindo as instruções em [Gerenciar um Plano do Serviço de Aplicativo no Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Coloque o aplicativo Web na mesma assinatura e defina o grupo de recursos como sua rede híbrida.

2. Repita a etapa anterior (1) no Hub Azure Stack.

### <a name="add-route-for-azure-stack-hub"></a>Adicionar rota para o Azure Stack Hub

É necessário que o Serviço de Aplicativo no Azure Stack Hub possa ser roteado da Internet pública para permitir que os usuários acessem seu aplicativo. Caso o Azure Stack Hub esteja acessível pela Internet, anote o endereço IP ou a URL voltada para o público para o aplicativo Web do Azure Stack Hub.

Se você está usando um ASDK, é possível [configurar um mapeamento NAT estático](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o Serviço de Aplicativo fora do ambiente virtual.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Conectar um aplicativo Web no Azure a uma rede híbrida

Para fornecer conectividade entre o front-end da Web no Azure e o banco de dados do SQL Server no Azure Stack Hub, é necessário que o aplicativo Web esteja conectado à rede híbrida entre o Azure e o Azure Stack Hub. Para habilitar a conectividade, você terá que:

- Configurar conectividade ponto a site.
- Configurar o aplicativo Web.
- Modificar o gateway de rede local no Azure Stack Hub.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configurar a rede virtual do Azure para conectividade ponto a site

É necessário que o gateway de rede virtual no lado do Azure da rede híbrida permita que conexões ponto a site sejam integradas ao Serviço de Aplicativo do Azure.

1. No portal do Azure, acesse a página do gateway de rede virtual. Em **Configurações**, selecione **Configuração ponto a site**.

    ![Opção ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Selecione **Configurar agora** para configurar o ponto a site.

    ![Iniciar a configuração ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Na página de configuração **Ponto a site**, insira o intervalo de endereços IP privado que você deseja usar no **Pool de endereços**.

   > [!Note]  
   > Verifique se o intervalo especificado não se sobrepõe a nenhum intervalo de endereços já usado por sub-redes nos componentes globais da rede híbrida do Azure ou do Azure Stack Hub.

   Em **Tipo de Túnel**, desmarque **IKEv2 VPN**. Selecione **Salvar** para concluir a configuração de ponto a site.

   ![Configurações de ponto a site no gateway de rede virtual do Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrar o aplicativo do Serviço de Aplicativo do Azure com a rede híbrida

1. Para conectar o aplicativo à VNet do Azure, siga as instruções em [Integração VNet exigida pelo gateway](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Acesse **Configurações** para ver o plano do Serviço de Aplicativo que hospeda o aplicativo Web. Em **Configurações**, selecione **Rede**.

    ![Configurar a Rede para o Plano do Serviço de Aplicativo](media/solution-deployment-guide-hybrid/image11.png)

3. Em **Integração VNET**, selecione **Clique aqui para gerenciar**.

    ![Gerenciar a integração VNET para o Plano do Serviço de Aplicativo](media/solution-deployment-guide-hybrid/image12.png)

4. Selecione o VNET que você deseja configurar. Em **ENDEREÇOS IP ROTEADOS PARA VNET**, insira o intervalo de endereços IP para a VNet do Azure, a do Azure Stack Hub e os espaços de endereço ponto a site. Selecione **Salvar** para validar e salvar essas configurações.

    ![Intervalos de endereços IP a serem roteados na integração da Rede Virtual](media/solution-deployment-guide-hybrid/image13.png)

Para saber mais sobre como o Serviço de Aplicativo se integra às VNets do Azure, confira [Integrar seu aplicativo a uma Rede Virtual do Azure](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Configurar a rede virtual do Azure Stack Hub

O gateway de rede local na rede virtual do Azure Stack Hub precisa ser configurado para rotear o tráfego pelo intervalo de endereços ponto a site do Serviço de Aplicativo.

1. No portal do Azure Stack Hub, acesse **Gateway de rede local**. Em **Configurações**, escolha **Configuração**.

    ![Opção de configuração de gateway no gateway de rede local do Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. Em **Espaço de endereço**, insira o intervalo de endereços ponto a site para o gateway de rede virtual no Azure.

    ![Espaço de endereço ponto a site no gateway de rede local do Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. Selecione **Salvar** para validar e salvar a configuração.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configurar o DNS para escala entre nuvens

Ao configurar corretamente o DNS para aplicativos entre nuvens, os usuários podem acessar as instâncias globais do Azure e do Azure Stack Hub do seu aplicativo Web. A configuração de DNS para este tutorial também permite que o Gerenciador de Tráfego do Azure roteie o tráfego quando a carga aumenta ou diminui.

Este tutorial usa o DNS do Azure para gerenciar o DNS porque os domínios do Serviço de Aplicativo não funcionarão.

### <a name="create-subdomains"></a>Criar subdomínios

Como o Gerenciador de Tráfego depende de CNAMEs de DNS, um subdomínio é necessário para rotear corretamente o tráfego para os pontos de extremidade. Para obter mais informações sobre os registros DNS e o mapeamento de domínio, confira [mapear domínios com o Gerenciador de Tráfego](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Para o ponto de extremidade do Azure, você criará um subdomínio que os usuários possam usar para acessar seu aplicativo Web. Neste tutorial, você pode usar **app.northwind.com**, mas deve personalizar esse valor com base no seu domínio.

Também será necessário criar um subdomínio com um registro A para o ponto de extremidade do Azure Stack Hub. Você pode usar **azurestack.northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Configurar um domínio personalizado no Azure

1. Adicione o nome do host **app.northwind.com** ao aplicativo Web do Azure [mapeando um CNAME para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Configurar domínios personalizados no Azure Stack Hub

1. Adicione o nome do host **azurestack.northwind.com** ao aplicativo Web do Azure Stack Hub [mapeando um registro A para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Use o endereço IP roteável da Internet para o aplicativo do Serviço de Aplicativo.

2. Adicione o nome do host **app.northwind.com** ao aplicativo Web do Azure Stack Hub [mapeando um CNAME para o Serviço de Aplicativo do Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Use o nome de host que você configurou na etapa anterior (1) como o destino para o CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configure certificados SSL para escala entre nuvens

É importante garantir que os dados confidenciais coletados pelo seu aplicativo Web estejam protegidos enquanto estão em trânsito e quando são armazenados no banco de dados SQL.

Você configurará seus aplicativos Web do Azure e do Azure Stack Hub para usar certificados SSL para todo o tráfego de entrada.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Adicionar SSL ao Azure e ao Azure Stack Hub

Para adicionar SSL ao Azure:

1. Verifique se o certificado SSL obtido é válido para o subdomínio que você criou. Não há problema em usar certificados curinga.

2. No portal do Azure, siga as instruções nas seções **Preparar o aplicativo Web** e **Associar certificado SSL** do artigo [Associar um certificado SSL personalizado existente a aplicativos Web do Azure](/azure/app-service/app-service-web-tutorial-custom-ssl). Selecione **SSL baseado em SNI** como o **Tipo de SSL**.

3. Redirecione todo o tráfego para a porta HTTPS. Siga as instruções na seção **Impor HTTPS** do artigo [Associar um certificado SSL personalizado existente a aplicativos Web do Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).

Para adicionar SSL ao Azure Stack Hub:

1. Repita as etapas de 1 a 3 que você usou para o Azure usando o portal do Azure Stack Hub.

## <a name="configure-and-deploy-the-web-app"></a>Configurar e implantar o aplicativo Web

Você configurará o código do aplicativo para relatar telemetria para a instância correta do Application Insights e configurará os aplicativos Web com as cadeias de conexão corretas. Para saber mais sobre o Application Insights, confira [O que é o Application Insights?](/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Adicionar o Application Insights

1. Abra seu aplicativo Web no Microsoft Visual Studio.

2. [Adicione o Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application Insights usa para criar alertas quando o tráfego da Web aumenta ou diminui.

### <a name="configure-dynamic-connection-strings"></a>Configurar cadeias de conexão dinâmicas

Cada instância do aplicativo Web usará um método diferente para se conectar ao banco de dados SQL. O aplicativo no Azure usa o endereço IP privado da VM do SQL Server, e o aplicativo no Azure Stack Hub usa o endereço IP público da VM do SQL Server.

> [!Note]  
> Em um sistema integrado do Azure Stack Hub, o endereço IP público não deve ser roteável pela Internet. Em um ASDK, o endereço IP público não é roteável fora do ASDK.

Você pode usar variáveis de ambiente do Serviço de Aplicativo para passar uma cadeia de conexão diferente para cada instância do aplicativo.

1. Abra o aplicativo no Visual Studio.

2. Abra Startup.cs e localize o seguinte bloco de código:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Substitua o bloco de código anterior pelo código a seguir, que usa uma cadeia de conexão definida no arquivo *appsettings.json*:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Definir as configurações do Serviço de Aplicativo

1. Crie cadeias de conexão para o Azure e o Azure Stack Hub. As cadeias de caracteres devem ser as mesmas, exceto para os endereços IP que são usados.

2. No Azure e no Azure Stack Hub, adicione a cadeia de conexão apropriada [como uma configuração de aplicativo](/azure/app-service/web-sites-configure) no aplicativo Web, usando `SQLCONNSTR\_` como um prefixo no nome.

3. **Salve** as configurações do aplicativo Web e reinicie o aplicativo.

## <a name="enable-automatic-scaling-in-global-azure"></a>Habilitar a colocação em escala automática no Azure global

Quando você cria seu aplicativo Web em um ambiente do Serviço de Aplicativo, ele começa com uma instância. Você pode escalar horizontalmente para adicionar instâncias e fornecer mais recursos de computação para o seu aplicativo. Da mesma forma, você pode reduzir horizontalmente o número de instâncias de que seu aplicativo precisa.

> [!Note]  
> Você precisa ter um Plano do Serviço de Aplicativo para configurar a ação de escalar e reduzir horizontalmente. Se você não tem um plano, crie um antes de iniciar as próximas etapas.

### <a name="enable-automatic-scale-out"></a>Habilitar a expansão automática

1. No portal do Azure, localize o Plano do Serviço de Aplicativo para os sites que você deseja escalar horizontalmente e, em seguida, selecione **Expansão (Plano do Serviço de Aplicativo)** .

    ![Escalar horizontalmente o Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image16.png)

2. Clique em **Habilitar dimensionamento automático**.

    ![Habilitar o dimensionamento automático no Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image17.png)

3. Forneça um nome para o **Nome de configuração do dimensionamento automático**. Na regra de dimensionamento automático **Padrão**, selecione **Escala com base em uma métrica**. Defina os **Limites da instância** como **Mínimo: 1**, **Máximo: 10** e **Padrão: 1**.

    ![Configurar o dimensionamento automático no Serviço de Aplicativo do Azure](media/solution-deployment-guide-hybrid/image18.png)

4. Selecione **+Adicionar uma regra**.

5. Em **Origem da Métrica**, selecione **Recurso Atual**. Use os critérios e as ações a seguir para a regra.

#### <a name="criteria"></a>Critérios

1. Em **Agregação de Tempo,** selecione **Média**.

2. Em **Nome da Métrica**, selecione **Percentual de CPU**.

3. Em **Operador**, selecione **Maior que**.

   - Defina o **Limite** como **50**.
   - Defina a **Duração** como **10**.

#### <a name="action"></a>Ação

1. Em **Operação**, selecione **Aumentar Contagem por**.

2. Defina a **Contagem de Instâncias** como **2**.

3. Defina o **Resfriamento** como **5**.

4. Selecione **Adicionar**.

5. Selecione **+ Adicionar uma regra**.

6. Em **Origem da Métrica**, selecione **Recurso Atual.**

   > [!Note]  
   > O recurso atual conterá o nome/GUID do Plano do Serviço de Aplicativo, e as listas suspensas **Tipo de Recurso** e **Recurso** ficarão indisponíveis.

### <a name="enable-automatic-scale-in"></a>Habilitar a redução horizontal automática

Quando o tráfego diminui, o aplicativo Web do Azure pode reduzir automaticamente o número de instâncias ativas para reduzir os custos. Essa ação é menos agressiva do que a expansão e minimiza o impacto nos usuários do aplicativo.

1. Acesse a condição de expansão **Padrão** e selecione **+ Adicionar uma regra**. Use os critérios e as ações a seguir para a regra.

#### <a name="criteria"></a>Critérios

1. Em **Agregação de Tempo,** selecione **Média**.

2. Em **Nome da Métrica**, selecione **Percentual de CPU**.

3. Em **Operador**, selecione **Menor que**.

   - Defina o **Limite** como **30**.
   - Defina a **Duração** como **10**.

#### <a name="action"></a>Ação

1. Em **Operação**, selecione **Diminuir contagem por**.

   - Defina a **Contagem de Instâncias** como **1**.
   - Defina o **Resfriamento** como **5**.

2. Selecione **Adicionar**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Crie um perfil do Gerenciador de Tráfego e configure a escala entre nuvens.

Crie um perfil do Gerenciador de Tráfego usando o portal do Azure e, em seguida, configure os pontos de extremidade para habilitar a escala entre nuvens.

### <a name="create-traffic-manager-profile"></a>Criar perfil do Gerenciador de Tráfego

1. Selecione **Criar um recurso**.
2. Selecione **Rede**.
3. Selecione **perfil do Gerenciador de Tráfego** e defina as seguintes configurações:

   - Em **Nome**, insira um nome para o seu perfil. Esse nome **deve** ser exclusivo na zona trafficmanager.net e usado para criar um nome DNS (por exemplo, northwindstore.trafficmanager.net).
   - Para **Método de roteamento**, selecione o **Ponderado**.
   - Em **Assinatura**, selecione a aquela na qual deseja criar esse perfil.
   - Em **Grupo de Recursos**, crie um grupo de recursos para esse perfil.
   - Em **Local do grupo de recursos**, selecione o local do grupo de recursos. Essa configuração refere-se ao local do grupo de recursos e não tem impacto no perfil do Gerenciador de Tráfego implantado globalmente.

4. Selecione **Criar**.

    ![Criar perfil do Gerenciador de Tráfego](media/solution-deployment-guide-hybrid/image19.png)

   Quando a implantação global do seu perfil do Gerenciador de Tráfego estiver concluída, ela será mostrada na lista do grupo de recursos no qual você a criou.

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos de extremidade do Gerenciador de Tráfego

1. Pesquise o perfil do Gerenciador de Tráfego que você criou. Se você navegou até o grupo de recursos para pesquisar o perfil, selecione-o.

2. No **Perfil do Gerenciador de Tráfego**, em **CONFIGURAÇÕES**, selecione **Ponto de extremidade**.

3. Selecione **Adicionar**.

4. Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure Stack Hub:

   - Para **Tipo**, selecione **Ponto de extremidade externo**.
   - Insira um **Nome** para o ponto de extremidade.
   - Para **FQDN (Nome de domínio totalmente qualificado) ou IP**, insira a URL externa para o seu aplicativo Web do Azure Stack Hub.
   - Em **Peso**, mantenha o valor padrão **1**. Esse peso faz com que todo o tráfego passe para esse ponto de extremidade caso ele esteja íntegro.
   - Deixe a opção **Adicionar como desabilitado** desmarcada.

5. Selecione **OK** para salvar o ponto de extremidade do Azure Stack Hub.

Você configurará o ponto de extremidade do Azure em seguida.

1. No **perfil do Gerenciador de Tráfego**, selecione **Ponto de extremidade**.
2. Selecione **+Adicionar**.
3. Em **Adicionar ponto de extremidade**, use as seguintes configurações para o Azure:

   - Em **Tipo**, selecione **Ponto de extremidade do Azure**.
   - Insira um **Nome** para o ponto de extremidade.
   - Para **Tipo de recurso de destino**, selecione **Serviço de Aplicativo**.
   - Em **Recurso de destino**, selecione **Escolher um serviço de aplicativo** para ver uma lista de aplicativos Web na mesma assinatura.
   - Em **Recursos**, escolha o Serviço de Aplicativo que deseja adicionar como o primeiro ponto de extremidade.
   - Em **Peso**, selecione **2**. Essa configuração faz com que todo o tráfego vá para esse ponto de extremidade caso o ponto de extremidade primário não esteja íntegro ou caso você tenha uma regra/alerta que redirecione o tráfego ao ser disparado.
   - Deixe a opção **Adicionar como desabilitado** desmarcada.

4. Selecione **OK** para salvar o ponto de extremidade do Azure.

Depois que todos os pontos de extremidade são configurados, eles são listados no **Perfil do Gerenciador de Tráfego** quando você seleciona os **Pontos de extremidade**. O exemplo na captura de tela a seguir mostra dois pontos de extremidade, com status e informações de configuração para cada um deles.

![Pontos de extremidades no perfil do Gerenciador de Tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Configurar o monitoramento e alertas do Application Insights no Azure

O Application Insights do Azure permite que você monitore seu aplicativo e envie alertas com base nas condições que você configurar. Por exemplo: o aplicativo está indisponível, está apresentando falhas ou está mostrando problemas de desempenho.

Você usará as métricas do Azure Application Insights para criar alertas. Quando esses alertas são disparados, a instância do aplicativo Web alterna automaticamente do Azure Stack Hub para o Azure para escalar horizontalmente e, em seguida, alterna novamente para o Azure Stack Hub para reduzir horizontalmente.

### <a name="create-an-alert-from-metrics"></a>Crie um alerta de métricas

No portal do Azure, acesse o grupo de recursos deste tutorial e selecione a instância do Application Insights para abrir o **Application Insights**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Você usará essa exibição para criar um alerta de expansão e outro de redução.

### <a name="create-the-scale-out-alert"></a>Criar o alerta de expansão

1. Em **CONFIGURAR**, selecione **Alertas (clássico)** .
2. Selecione **adicionar alerta de métrica (clássico)** .
3. Em **Adicionar regra**, defina as seguintes configurações:

   - Em **Nome**, insira **Intermitência no Azure Cloud**.
   - A **Descrição** é opcional.
   - Em **Origem** > **Alerta no**, selecione **Métricas**.
   - Em **Critérios**, selecione sua assinatura, o grupo de recursos do seu perfil do Gerenciador de Tráfego e o nome do perfil do Gerenciador de Tráfego para o recurso.

4. Em **Métrica**, selecione **Taxa de Solicitação**.
5. Em **Condição**, selecione **Maior que**.
6. Em **Limite**, insira **2**.
7. Em **Período**, selecione **Nos últimos 5 minutos**.
8. Em **Notificar via**:
   - Marque a caixa de seleção **Proprietários, colaboradores e leitores de email**.
   - Insira seu endereço de email em **Emails adicionais do administrador**.

9. Na barra de menu, selecione **Salvar**.

### <a name="create-the-scale-in-alert"></a>Criar o alerta de redução

1. Em **CONFIGURAR**, selecione **Alertas (clássico)** .
2. Selecione **adicionar alerta de métrica (clássico)** .
3. Em **Adicionar regra**, defina as seguintes configurações:

   - Em **Nome**, insira **Escalar de volta para o Azure Stack Hub**.
   - A **Descrição** é opcional.
   - Em **Origem** > **Alerta no**, selecione **Métricas**.
   - Em **Critérios**, selecione sua assinatura, o grupo de recursos do seu perfil do Gerenciador de Tráfego e o nome do perfil do Gerenciador de Tráfego para o recurso.

4. Em **Métrica**, selecione **Taxa de Solicitação**.
5. Em **Condição**, selecione **Menor que**.
6. Em **Limite**, insira **2**.
7. Em **Período**, selecione **Nos últimos 5 minutos**.
8. Em **Notificar via**:
   - Marque a caixa de seleção **Proprietários, colaboradores e leitores de email**.
   - Insira seu endereço de email em **Emails adicionais do administrador**.

9. Na barra de menu, selecione **Salvar**.

A captura de tela a seguir mostra os alertas de expansão e redução horizontal.

   ![Alertas do Application Insights (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Redirecionar o tráfego entre o Azure e o Azure Stack Hub

Você pode configurar a alternância do tráfego de aplicativo Web para manual ou automática entre o Azure e o Azure Stack Hub.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Configurar a alternância manual entre o Azure e o Azure Stack Hub

Quando o site atingir os limites que você configurar, você receberá um alerta. Use as etapas a seguir para redirecionar manualmente o tráfego para o Azure.

1. No portal do Azure, selecione seu perfil do Gerenciador de Tráfego.

    ![Pontos de extremidade do Gerenciador de Tráfego no portal do Azure](media/solution-deployment-guide-hybrid/image20.png)

2. Selecione **Pontos de extremidade**.
3. Selecione o **Ponto de extremidade do Azure**.
4. Em **Status**, selecione **Habilitado** e, em seguida, selecione **Salvar**.

    ![Habilitar o ponto de extremidade do Azure no portal do Azure](media/solution-deployment-guide-hybrid/image23.png)

5. Em **Pontos de extremidade** para o perfil do Gerenciador de Tráfego, selecione **Ponto de extremidade externo**.
6. Em **Status**, selecione **Desabilitado** e, em seguida, selecione **Salvar**.

    ![Desabilitar ponto de extremidade do Azure Stack Hub no portal do Azure](media/solution-deployment-guide-hybrid/image24.png)

Depois que os pontos de extremidade são configurados, o tráfego do aplicativo vai para o aplicativo Web de expansão do Azure em vez do aplicativo Web do Azure Stack Hub.

 ![Pontos de extremidade alterados no tráfego do aplicativo Web do Azure](media/solution-deployment-guide-hybrid/image25.png)

Para reverter o fluxo de volta para o Azure Stack Hub, use as etapas anteriores para:

- Habilitar o ponto de extremidade do Azure Stack Hub.
- Desabilitar o ponto de extremidade do Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Configurar a alternância automática entre o Azure e o Azure Stack Hub

Também é possível usar o monitoramento do Application Insights se seu aplicativo é executado em um ambiente [sem servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelo Azure Functions.

Nesse cenário, você pode configurar o Application Insights para usar um webhook que chama um aplicativo de funções. Esse aplicativo habilita ou desabilita automaticamente um ponto de extremidade em resposta a um alerta.

Use as etapas a seguir como um guia para configurar a alternância automática de tráfego.

1. Crie um aplicativo de funções do Azure.
2. Crie uma função disparada por HTTP.
3. Importe os SDKs do Azure para o Resource Manager, aplicativos Web e Gerenciador de Tráfego.
4. Desenvolva código para:

   - Autenticar-se na sua assinatura do Azure.
   - Usar um parâmetro que alterna os pontos de extremidade do Gerenciador de Tráfego para direcionar o tráfego para o Azure ou o Azure Stack Hub.

5. Salvar seu código e adicionar a URL do aplicativo de funções com os parâmetros apropriados à seção **Webhook** das configurações da regra de alerta do Application Insights.
6. O tráfego é redirecionado automaticamente quando um alerta do Application Insights é disparado.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).
