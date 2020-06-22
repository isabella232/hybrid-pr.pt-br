---
title: Configurar a conectividade de nuvem híbrida no Azure e no Hub de Azure Stack
description: Saiba como configurar a conectividade de nuvem híbrida usando o Azure e o Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0e1a0fc4fb4110fdb406d4b4b2e72abb8f5412c9
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909783"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Configurar a conectividade de nuvem híbrida usando o Azure e o Hub de Azure Stack

Você pode acessar recursos com segurança no Azure global e Azure Stack Hub usando o padrão de conectividade híbrida.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Mantenha os dados locais para atender aos requisitos de privacidade ou normativos, mas mantenha o acesso aos recursos globais do Azure.
> - Mantenha um sistema herdado ao usar recursos e implantação de aplicativo em escala de nuvem no Azure global.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Alguns componentes são necessários para criar uma implantação de conectividade híbrida. Alguns desses componentes levam tempo para se preparar, portanto, planeje adequadamente.

### <a name="azure"></a>Azure

- Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Crie um [aplicativo Web](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts) no Azure. Anote a URL do aplicativo Web porque você precisará dela na solução.

### <a name="azure-stack-hub"></a>Hub de Azure Stack

Um parceiro de hardware/OEM do Azure pode implantar um hub de Azure Stack de produção e todos os usuários podem implantar um Kit de Desenvolvimento do Azure Stack (ASDK).

- Use o Hub de Azure Stack de produção ou implante o ASDK.
   >[!Note]
   >A implantação do ASDK pode levar até 7 horas, portanto, planeje de acordo.

- Implante serviços de PaaS do [serviço de aplicativo](/azure-stack/operator/azure-stack-app-service-deploy.md) para Azure Stack Hub.
- [Crie planos e ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) no ambiente de Hub de Azure Stack.
- [Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro do ambiente de Hub de Azure Stack.

### <a name="azure-stack-hub-components"></a>Componentes do hub de Azure Stack

Um operador de Hub de Azure Stack deve implantar o serviço de aplicativo, criar planos e ofertas, criar uma assinatura de locatário e adicionar a imagem do Windows Server 2016. Se você já tiver esses componentes, verifique se eles atendem aos requisitos antes de iniciar essa solução.

Este exemplo de solução pressupõe que você tenha algum conhecimento básico do Azure e do hub de Azure Stack. Para saber mais antes de iniciar a solução, leia os seguintes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave de Hub de Azure Stack](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Antes de começar

Verifique se você atende aos seguintes critérios antes de começar a configurar a conectividade de nuvem híbrida:

- Você precisa de um endereço IPv4 público de volta externa para seu dispositivo VPN. Esse endereço IP não pode ser localizado atrás de um NAT (conversão de endereços de rede).
- Todos os recursos são implantados na mesma região/local.

#### <a name="solution-example-values"></a>Valores de exemplo de solução

Os exemplos nesta solução usam os valores a seguir. Você pode usar esses valores para criar um ambiente de teste ou consultá-los para uma melhor compreensão dos exemplos. Para obter mais informações sobre as configurações de gateway de VPN, consulte [sobre as configurações de gateway de VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Especificações de conexão:

- **Tipo de VPN**: baseada em rota
- **Tipo de conexão**: site a site (IPSec)
- **Tipo de gateway**: VPN
- **Nome da conexão do Azure**: Azure-gateway-AzureStack-S2SGateway (o portal fará o preenchimento automático desse valor)
- **Nome de conexão do hub de Azure Stack**: AzureStack-gateway-Azure-S2SGateway (o portal fará o preenchimento automático desse valor)
- **Chave compartilhada**: qualquer compatível com hardware VPN, com valores correspondentes em ambos os lados da conexão
- **Assinatura**: qualquer assinatura preferencial
- **Grupo de recursos**: teste-infraestrutura

Endereços IP de rede e de sub-rede:

| Conexão do Hub do Azure/Azure Stack | Nome | Sub-rede | Endereço IP |
|---|---|---|---|
| VNet do Azure | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| VNet do hub de Azure Stack | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Gateway de rede virtual do Azure | Azure-gateway |  |  |
| Gateway de rede virtual do hub de Azure Stack | AzureStack-gateway |  |  |
| IP público do Azure | Azure-GatewayPublicIP |  | Determinado na criação |
| IP público do hub de Azure Stack | AzureStack-GatewayPublicIP |  | Determinado na criação |
| Gateway de rede local do Azure | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack valor IP público do Hub |
| Gateway de rede local de Hub Azure Stack | Azure-S2SGateway<br>10.100.102.0/23 |  | Valor de IP público do Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Criar uma rede virtual no Azure global e no Hub de Azure Stack

Use as etapas a seguir para criar uma rede virtual usando o Portal. Você pode usar esses [valores de exemplo](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) se estiver usando este artigo como apenas uma solução. Se você estiver usando este artigo para configurar um ambiente de produção, substitua as configurações de exemplo pelos seus próprios valores.

> [!IMPORTANT]
> Você deve garantir que não haja uma sobreposição de endereços IP no Azure ou Azure Stack espaços de endereço de vNet do Hub.

Para criar uma vNet no Azure:

1. Use seu navegador para se conectar ao [portal do Azure](https://portal.azure.com/) e entre com sua conta do Azure.
2. Selecione **Criar um recurso**. No campo **Pesquisar no Marketplace** , insira "rede virtual". Selecione **rede virtual** nos resultados.
3. Na lista **selecionar um modelo de implantação** , selecione **Gerenciador de recursos**e, em seguida, selecione **criar**.
4. Em **criar rede virtual**, defina as configurações de VNet. Os nomes dos campos obrigatórios são prefixados com um asterisco vermelho.  Quando você insere um valor válido, o asterisco é alterado para uma marca de seleção verde.

Para criar uma vNet no Hub de Azure Stack:

1. Repita as etapas acima (1-4) usando o portal de **locatário**do Hub Azure Stack.

## <a name="add-a-gateway-subnet"></a>Adicionar uma sub-rede de gateway

Antes de conectar sua rede virtual a um gateway, você precisa criar a sub-rede de gateway para a rede virtual à qual você deseja se conectar. Os serviços de gateway usam os endereços IP que você especifica na sub-rede de gateway.

Na [portal do Azure](https://portal.azure.com/), navegue até a rede virtual do Gerenciador de recursos em que você deseja criar um gateway de rede virtual.

1. Selecione a vNet para abrir a página **rede virtual** .
2. Em **configurações**, selecione **sub-redes**.
3. Na página **sub-redes** , selecione **+ sub-rede de gateway** para abrir a página **Adicionar sub-rede** .

    ![Adicionar sub-rede de gateway](media/solution-deployment-guide-connectivity/image4.png)

4. O **nome** da sub-rede é preenchido automaticamente com o valor ' GatewaySubnet '. Esse valor é necessário para o Azure reconhecer a sub-rede como a sub-rede do gateway.
5. Altere os valores de **intervalo de endereços** que são fornecidos para corresponder aos seus requisitos de configuração e selecione **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Criar um gateway de rede virtual no Azure e Azure Stack

Use as etapas a seguir para criar um gateway de rede virtual no Azure.

1. No lado esquerdo da página do portal, selecione **+** e insira ' gateway de rede virtual ' no campo de pesquisa.
2. Em **resultados**, selecione **Gateway de rede virtual**.
3. Em **Gateway de rede virtual**, selecione **criar** para abrir a página **criar gateway de rede virtual** .
4. Em **criar gateway de rede virtual**, especifique os valores para o seu gateway de rede usando nossos **valores de exemplo de tutorial**. Inclua os seguintes valores adicionais:

   - **SKU**: básico
   - **Rede virtual**: selecione a rede virtual que você criou anteriormente. A sub-rede de gateway que você criou é selecionada automaticamente.
   - **Primeira configuração de IP**: o IP público do seu gateway.
     - Selecione **criar configuração de IP de gateway**, que leva você para a página **escolher endereço IP público** .
     - Selecione **+ criar novo** para abrir a página **criar endereço IP público** .
     - Insira um **nome** para seu endereço IP público. Deixe a SKU como **básica**e, em seguida, selecione **OK** para salvar suas alterações.

       > [!Note]
       > Atualmente, o gateway de VPN dá suporte apenas à alocação de endereços IP públicos dinâmicos. No entanto, isso não significa que o endereço IP seja alterado após ser atribuído ao seu gateway de VPN. A única vez em que o endereço IP Público é alterado é quando o gateway é excluído e recriado. O redimensionamento, a redefinição ou outras atualizações/manutenção internas para o gateway de VPN não alteram o endereço IP.

5. Verifique as configurações do gateway.
6. Selecione **criar** para criar o gateway de VPN. As configurações do gateway são validadas e o bloco "Implantando o gateway de rede virtual" é mostrado no seu painel.

   >[!Note]
   >A criação de um gateway pode levar até 45 minutos. Talvez seja necessário atualizar a página do portal para ver o status concluído.

    Depois que o gateway for criado, você poderá ver o endereço IP atribuído a ele examinando a rede virtual no Portal. O gateway aparecerá como um dispositivo conectado. Para ver mais informações sobre o gateway, selecione o dispositivo.

7. Repita as etapas anteriores (1-5) em sua implantação de Hub de Azure Stack.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Criar o gateway de rede local no Azure e no Hub de Azure Stack

O gateway de rede local geralmente se refere ao seu local. Você dá ao site um nome ao qual o Hub do Azure ou Azure Stack pode se referir e, em seguida, especifica:

- O endereço IP do dispositivo VPN local para o qual você está criando uma conexão.
- Os prefixos de endereço IP que serão roteados por meio do gateway de VPN para o dispositivo VPN. Os prefixos de endereço que você especifica são os prefixos localizados em sua rede local.

  >[!Note]
  >Se sua rede local for alterada ou você precisar alterar o endereço IP público para o dispositivo VPN, você poderá atualizar esses valores posteriormente.

1. No portal, selecione **+ criar um recurso**.
2. Na caixa de pesquisa, insira **Gateway de rede local**e, em seguida, selecione **Enter** para pesquisar. Uma lista de resultados será exibida.
3. Selecione **Gateway de rede local**e, em seguida, selecione **criar** para abrir a página **criar gateway de rede local** .
4. Em **criar gateway de rede local**, especifique os valores para seu gateway de rede local usando nossos **valores de exemplo de tutorial**. Inclua os seguintes valores adicionais:

    - **Endereço IP**: o endereço IP público do dispositivo VPN ao qual você deseja que o Azure ou Azure Stack Hub se conecte. Especifique um endereço IP público válido que não esteja atrás de um NAT para que o Azure possa acessar o endereço. Se você não tiver o endereço IP no momento, poderá usar um valor do exemplo como um espaço reservado. Você precisará voltar e substituir o espaço reservado pelo endereço IP público do seu dispositivo VPN. O Azure não pode se conectar ao dispositivo até que você forneça um endereço válido.
    - **Espaço de endereço**: o intervalo de endereços para a rede que essa rede local representa. Você pode adicionar vários intervalos de espaço de endereço. Certifique-se de que os intervalos especificados não se sobreponham a intervalos de outras redes às quais você deseja se conectar. O Azure roteará o intervalo de endereços especificado para o endereço IP do dispositivo VPN local. Use seus próprios valores se desejar se conectar ao site local, não um valor de exemplo.
    - **Definir configurações de BGP**: Use somente ao configurar o BGP. Caso contrário, não selecione essa opção.
    - **Assinatura**: Verifique se a assinatura correta está sendo exibida.
    - **Grupo de recursos**: selecione o grupo de recursos que você deseja usar. Você pode criar um novo grupo de recursos ou selecionar um que já tenha criado.
    - **Local**: selecione o local em que esse objeto será criado. Talvez você queira selecionar o mesmo local em que sua VNet reside, mas não é necessário fazer isso.
5. Quando terminar de especificar os valores necessários, selecione **criar** para criar o gateway de rede local.
6. Repita essas etapas (1-5) em sua implantação de Hub de Azure Stack.

## <a name="configure-your-connection"></a>Configurar sua conexão

As conexões site a site com uma rede local exigem um dispositivo VPN. O dispositivo VPN que você configura é conhecido como uma conexão. Para configurar sua conexão, você precisará de:

- Uma chave compartilhada. Essa chave é a mesma chave compartilhada que você especifica ao criar a conexão VPN site a site. Em nossos exemplos, usamos uma chave compartilhada básica. Recomendamos gerar uma chave mais complexa para uso.
- O endereço IP público do seu gateway de rede virtual. Você pode exibir o endereço IP público usando o portal do Azure, o PowerShell ou a CLI. Para localizar o endereço IP público do seu gateway de VPN usando o portal do Azure, vá para gateways de rede virtual e selecione o nome do seu gateway.

Use as etapas a seguir para criar uma conexão VPN site a site entre o gateway de rede virtual e o dispositivo VPN local.

1. Na portal do Azure, selecione **+ criar um recurso**.
2. Procure **conexões**.
3. Em **resultados**, selecione **conexões**.
4. Em **conexão**, selecione **criar**.
5. Em **criar conexão**, defina as seguintes configurações:

    - **Tipo de conexão**: selecione site a site (IPSec).
    - **Grupo de recursos**: selecione o grupo de recursos de teste.
    - **Gateway de rede virtual**: selecione o gateway de rede virtual que você criou.
    - **Gateway de rede local**: selecione o gateway de rede local que você criou.
    - **Nome da conexão**: esse nome é preenchido automaticamente usando os valores dos dois gateways.
    - **Chave compartilhada**: esse valor deve corresponder ao valor que você está usando para o dispositivo VPN local. O exemplo de tutorial usa ' abc123 ', mas você deve usar algo mais complexo. O importante é que esse valor *deve* ser o mesmo valor que você especificar ao configurar seu dispositivo VPN.
    - Os valores para **Assinatura**, **Grupo de recursos** e **Local** são fixos.

6. Selecione **OK** para criar sua conexão.

Você pode ver a conexão na página **conexões** do gateway de rede virtual. O status será de *desconhecido* para *se conectar*e, em seguida, para *êxito*.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).
