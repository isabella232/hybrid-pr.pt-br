---
title: Configurar a conectividade de nuvem híbrida no Azure e no Azure Stack Hub
description: Saiba como configurar a conectividade de nuvem híbrida usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4480f51b03082f2a0cbb7f2f213e05b7bf488646
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895363"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Configurar a conectividade de nuvem híbrida usando o Azure e o Azure Stack Hub

Você pode acessar recursos com segurança no Azure global e no Azure Stack Hub usando o padrão de conectividade híbrida.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Manter os dados locais a fim de atender aos requisitos de privacidade ou normativos e também manter o acesso aos recursos globais do Azure.
> - Manter um sistema herdado ao usar recursos e implantação de aplicativos em escala de nuvem no Azure global.

> [!Tip]  
> ![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Alguns componentes são necessários para criar uma implantação de conectividade híbrida. Alguns desses componentes levam tempo para preparar, portanto, planeje com antecedência.

### <a name="azure"></a>Azure

- Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Crie um [aplicativo Web](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs) no Azure. Anote a URL do aplicativo Web, porque você precisará dela na solução.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Um parceiro de hardware/OEM do Azure pode implantar um Azure Stack Hub de produção, e todos os usuários podem implantar um ASDK (Kit de Desenvolvimento do Azure Stack).

- Use o Azure Stack Hub de produção ou implante o ASDK.
   >[!Note]
   >A implantação do ASDK pode levar até sete horas, portanto, planeje com antecedência.

- Implante serviços PaaS do [Serviço de Aplicativo](/azure-stack/operator/azure-stack-app-service-deploy) no Azure Stack Hub.
- [Crie planos e ofertas](/azure-stack/operator/service-plan-offer-subscription-overview) no ambiente do Azure Stack Hub.
- [Crie uma assinatura de locatário](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) no ambiente do Azure Stack Hub.

### <a name="azure-stack-hub-components"></a>Componentes do Azure Stack Hub

Um operador do Azure Stack Hub deve implantar o Serviço de Aplicativo, criar planos e ofertas, criar uma assinatura de locatário e adicionar a imagem do Windows Server 2016. Se você já tem esses componentes, verifique se eles atendem aos requisitos antes de iniciar essa solução.

Este exemplo de solução pressupõe que você tenha algum conhecimento básico do Azure e do Azure Stack Hub. Para saber mais antes de iniciar a solução, leia os seguintes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave do Azure Stack Hub](/azure-stack/operator/azure-stack-overview)

### <a name="before-you-begin"></a>Antes de começar

Verifique se você atende aos seguintes critérios antes de começar a configurar a conectividade de nuvem híbrida:

- Você precisa de um endereço IPv4 público voltado para o exterior para o seu dispositivo VPN. Esse endereço IP não pode ser localizado atrás de um NAT (Conversão de Endereços de Rede).
- Todos os recursos são implantados na mesma região/local.

#### <a name="solution-example-values"></a>Valores de exemplo de solução

Os exemplos nesta solução usam os valores a seguir. Você pode usar esses valores para criar um ambiente de teste ou consultá-los para compreender melhor os exemplos. Para obter mais informações sobre configurações de gateway de VPN, confira [Sobre as configurações de Gateway de VPN](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Especificações de conexão:

- **Tipo de VPN**: baseada em rota
- **Tipo de Conexão**: site a site (IPsec)
- **Tipo de gateway**: VPN
- **Nome de conexão do Azure**: Azure-Gateway-AzureStack-S2SGateway (o portal fará o preenchimento automático desse valor)
- **Nome da conexão do Azure Stack Hub**: AzureStack-Gateway-Azure-S2SGateway (o portal fará o preenchimento automático desse valor)
- **Chave compartilhada**: todas as chaves compatíveis com o hardware de VPN, com valores correspondentes em ambos os lados da conexão
- **Assinatura**: todas as assinaturas preferenciais
- **Grupo de recursos**: Test-Infra

Endereços IP de rede e sub-rede:

| Conexão do Azure/Azure Stack Hub | Nome | Sub-rede | Endereço IP |
|---|---|---|---|
| VNet do Azure | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| VNet do Azure Stack Hub | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Gateway de Rede Virtual do Azure | Azure-Gateway |  |  |
| Gateway de Rede Virtual do Azure Stack Hub | AzureStack-Gateway |  |  |
| IP público do Azure | Azure-GatewayPublicIP |  | Determinado na criação |
| IP Público do Azure Stack Hub | AzureStack-GatewayPublicIP |  | Determinado na criação |
| Gateway de Rede Local do Azure | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Valor de IP Público do Azure Stack Hub |
| Gateway de Rede Local do Azure Stack Hub | Azure-S2SGateway<br>10.100.102.0/23 |  | Valor de IP Público do Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Criar uma rede virtual no Azure global e no Azure Stack Hub

Use as etapas a seguir para criar uma rede virtual usando o portal. Você pode usar esses [valores de exemplo](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) caso esteja usando este artigo apenas como uma solução. Se você está usando este artigo para configurar um ambiente de produção, substitua as configurações de exemplo pelos seus próprios valores.

> [!IMPORTANT]
> Você deve garantir que não haja uma sobreposição de endereços IP no Azure ou nos espaços de endereço da vNet do Azure Stack Hub.

Para criar uma vNet no Azure:

1. Use seu navegador para se conectar ao [portal do Azure](https://portal.azure.com/) e entrar na sua conta do Azure.
2. Selecione **Criar um recurso**. No campo **Pesquisar no marketplace**, insira “rede virtual”. Selecione **Rede virtual** nos resultados.
3. Na lista **Selecionar um modelo de implantação**, selecione **Gerenciador de Recursos** e, em seguida, **Criar**.
4. Em **Criar rede virtual**, defina as configurações da VNet. Os nomes dos campos obrigatórios são prefixados com um asterisco vermelho.  Ao inserir um valor válido, o asterisco é alterado para uma marca de seleção verde.

Para criar uma vNet no Azure Stack Hub:

1. Repita as etapas acima (1 a 4) usando o **portal de locatário** do Azure Stack Hub.

## <a name="add-a-gateway-subnet"></a>Adicionar uma sub-rede de gateway

Antes de conectar sua rede virtual a um gateway, é necessário criar a sub-rede de gateway para a rede virtual à qual você deseja se conectar. Os serviços de gateway usam os endereços IP que você especificou na sub-rede do gateway.

No [portal do Azure](https://portal.azure.com/), navegue até a rede virtual do Gerenciador de Recursos na qual você deseja criar um gateway de rede virtual.

1. Selecione vNet para abrir a página **Rede virtual**.
2. Em **CONFIGURAÇÕES**, selecione **Sub-redes**.
3. Na página **Sub-redes**, selecione **+Sub-rede de gateway** para abrir a página **Adicionar sub-rede**.

    ![Adicionar sub-rede de gateway](media/solution-deployment-guide-connectivity/image4.png)

4. O **Nome** da sub-rede será automaticamente preenchido com o valor “GatewaySubnet”. Esse valor é necessário para o Azure reconhecer a sub-rede como a sub-rede do gateway.
5. Altere os valores do **Intervalo de endereços** que são fornecidos para corresponder aos seus requisitos de configuração e, em seguida, selecione **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Criar um gateway de rede virtual no Azure e no Azure Stack

Use as etapas a seguir para criar um gateway de rede virtual no Azure.

1. No lado esquerdo da página do portal, selecione **+** e insira “gateway de rede virtual” no campo de pesquisa.
2. Em **Resultados**, selecione **Gateway de rede virtual**.
3. Em **Gateway de rede virtual**, selecione **Criar** para abrir a página **Criar gateway de rede virtual**.
4. Em **Criar gateway de rede virtual**, especifique os valores de seu gateway de rede usando os nossos **Valores de exemplo do tutorial**. Inclua os seguintes valores adicionais:

   - **SKU**: básico
   - **Rede virtual**: selecione a rede virtual que você criou anteriormente. A sub-rede de gateway que você criou é selecionada automaticamente.
   - **Primeira configuração de IP**:  o IP público do seu gateway.
     - Selecione **Criar configuração de IP do gateway**, que abrirá a página **Escolher endereço IP público**.
     - Selecione **+Criar novo** para abrir a página **Criar endereço IP público**.
     - Insira um **Nome** para o seu endereço IP público. Deixe a SKU como **Básica** e, em seguida, selecione **OK** para salvar as alterações.

       > [!Note]
       > No momento, o Gateway de VPN dá suporte apenas à alocação de endereços IP público dinâmico. No entanto, isso não significa que o endereço IP mude depois de eles ser atribuído ao seu gateway de VPN. A única vez em que o endereço IP Público é alterado é quando o gateway é excluído e recriado. O redimensionamento, a redefinição ou outras manutenções/atualizações internas do seu gateway de VPN não alteram o endereço IP.

5. Verifique as configurações do seu gateway.
6. Selecione **Criar** para criar o gateway de VPN. As configurações do gateway são validadas e o bloco “Implantando o gateway de rede virtual” é mostrado no seu painel.

   >[!Note]
   >A criação de um gateway pode levar até 45 minutos. Talvez seja necessário atualizar a página do portal para ver o status concluído.

    Após a criação do gateway, será possível ver o endereço IP atribuído a ele examinando a rede virtual no portal. O gateway aparecerá como um dispositivo conectado. Para ver mais informações sobre o gateway, selecione o dispositivo.

7. Repita as etapas anteriores (1 a 5) em sua implantação do Azure Stack Hub.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Criar o gateway de rede local no Azure e no Azure Stack Hub

O gateway de rede local geralmente se refere ao seu local. Atribua ao site um nome que pode ser consultado no Azure ou no Azure Stack Hub e, em seguida, especifique:

- O endereço IP do dispositivo VPN local para o qual você está criando uma conexão.
- Os prefixos de endereço IP que serão roteados por meio do gateway de VPN para o dispositivo VPN. Os prefixos de endereço que você especifica são os prefixos localizados em sua rede local.

  >[!Note]
  >Caso sua rede local seja alterada ou você precise alterar o endereço IP público para o dispositivo VPN, é possível atualizar esses valores posteriormente.

1. No portal, selecione **+Criar um recurso**.
2. Na caixa de pesquisa, insira **Gateway de rede local**, em seguida, selecione **Enter** para pesquisar. Uma lista de resultados será exibida.
3. Selecione **Gateway de rede local**, em seguida, selecione **Criar** para abrir a página **Criar gateway de rede local**.
4. Em **Criar gateway de rede local**, especifique os valores de seu gateway de rede local usando os nossos **Valores de exemplo do tutorial**. Inclua os seguintes valores adicionais:

    - **Endereço IP**: o endereço IP público do dispositivo VPN ao qual você deseja que o Azure Stack Hub se conecte. Especifique um endereço IP público válido que não esteja atrás de um NAT para que o Azure possa acessar o endereço. Caso você não tenha o endereço IP no momento, é possível usar um valor do exemplo como um espaço reservado. Você precisará voltar e substituir o espaço reservado pelo endereço IP público do seu dispositivo VPN. O Azure não pode se conectar ao dispositivo até que você forneça um endereço válido.
    - **Espaço de Endereço**: intervalo de endereços para a rede que é representada por esse local. Você pode adicionar vários intervalos de espaço de endereço. Verifique se os intervalos que você especifica não se sobrepõem aos intervalos de outras redes com as quais você deseja se conectar. O Azure roteará o intervalo de endereços especificado para o endereço IP do dispositivo VPN local. Se desejar se conectar ao site local, use seus valores, não um valor de exemplo.
    - **Configurar as definições de BGP ASN**: Use somente ao configurar o BGP. Caso contrário, não selecione essa opção.
    - **Assinatura**: Verifique se a assinatura correta está sendo exibida.
    - **Grupo de Recursos**: Selecione o grupo de recursos que você deseja usar. Você pode criar um novo grupo de recursos ou selecionar um que você já criou.
    - **Localização**: Selecione o local em que esse objeto será criado. É possível selecionar o mesmo local onde reside a sua rede virtual, mas não é necessário fazê-lo.
5. Quando terminar de especificar os valores necessários, selecione **Criar** para criar o gateway de rede local.
6. Repita essas etapas (1 a 5) em sua implantação do Azure Stack Hub.

## <a name="configure-your-connection"></a>Configure sua conexão

As conexões site a site para uma rede local exigem um dispositivo VPN. O dispositivo VPN que você configura é conhecido como conexão. Para configurar sua conexão, você precisará de:

- Uma chave compartilhada. Essa é a mesma chave compartilhada que você especificou ao criar a conexão VPN site a site. Em nossos exemplos, usamos uma chave compartilhada básica. Recomendamos gerar uma chave mais complexa para uso.
- O endereço IP público do seu gateway de rede virtual. Você pode exibir o endereço IP público usando o portal do Azure, o PowerShell ou a CLI. Para localizar o endereço IP público do seu gateway de VPN usando o portal do Azure, navegue até gateways de rede virtual e selecione o nome do seu gateway.

Use as seguintes etapas para criar a conexão VPN site a site entre o gateway de rede virtual e o dispositivo VPN local.

1. No portal do Azure, selecione **+Criar um recurso**.
2. Pesquise por **conexões**.
3. Em **Resultados**, selecione **Conexões**.
4. Em **Conexão**, selecione **Criar**.
5. Em **Criar Conexão**, defina as seguintes configurações:

    - **Tipo de conexão**: selecione site a site (IPSec).
    - **Grupo de Recursos**: selecione o grupo de recursos de teste.
    - **Gateway de Rede Virtual**: selecione o gateway de rede virtual que você criou.
    - **Gateway de Rede Local**: selecione o gateway de rede local que você criou.
    - **Nome da Conexão**: esse nome é preenchido automaticamente usando os valores dos dois gateways.
    - **Chave compartilhada:** esse valor deve corresponder ao valor que você está usando para o dispositivo VPN local. O exemplo de tutorial usa “abc123”, mas você deve usar algo mais complexo. O importante é que esse valor *deve* ser o mesmo valor especificado ao configurar seu dispositivo VPN.
    - Os valores para **Assinatura**, **Grupo de recursos** e **Local** são fixos.

6. Selecione **OK** para criar a conexão.

Você pode ver a conexão na página **Conexões** do seu gateway de rede virtual. O status será alterado de *Desconhecido* para *Conectando* e então para *Êxito*.

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).
