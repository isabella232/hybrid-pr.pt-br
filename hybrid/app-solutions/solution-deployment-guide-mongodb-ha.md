---
title: Implantar uma solução MongoDB altamente disponível no Azure e no Hub de Azure Stack
description: Saiba como implantar uma solução MongoDB altamente disponível para o Azure e o Hub de Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909866"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Implantar uma solução MongoDB altamente disponível no Azure e no Hub de Azure Stack

Este artigo o orientará em uma implantação automatizada de um cluster MongoDB de alta disponibilidade (HA) básico com um site de DR (recuperação de desastre) em dois ambientes de Hub de Azure Stack. Para saber mais sobre o MongoDB e a alta disponibilidade, consulte [membros do conjunto de réplicas](https://docs.mongodb.com/manual/core/replica-set-members/).

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Orquestrar uma implantação em dois hubs de Azure Stack.
> - Use o Docker para minimizar problemas de dependência com perfis de API do Azure.
> - Implante um cluster MongoDB básico de alta disponibilidade com um site de recuperação de desastre.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Arquitetura para MongoDB com hub de Azure Stack

![arquitetura do MongoDB altamente disponível no Hub de Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Pré-requisitos para o MongoDB com hub de Azure Stack

- Dois sistemas integrados de Hub de Azure Stack (Hub de Azure Stack) conectados. Essa implantação não funciona no Kit de Desenvolvimento do Azure Stack (ASDK). Para saber mais sobre o Hub de Azure Stack, confira [o que é o Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Uma assinatura de locatário em cada Hub de Azure Stack. 
  - **Anote cada ID de assinatura e o ponto de extremidade de Azure Resource Manager para cada Hub de Azure Stack.**
- Uma entidade de serviço Azure Active Directory (AD do Azure) que tem permissões para a assinatura de locatário em cada Hub de Azure Stack. Talvez seja necessário criar duas entidades de serviço se os hubs de Azure Stack forem implantados em diferentes locatários do Azure AD. Para saber como criar uma entidade de serviço para Azure Stack Hub, consulte [usar uma identidade de aplicativo para acessar recursos de Azure Stack Hub](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).
  - **Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) da entidade de serviço.**
- O Ubuntu 16, 4 é agregado ao Marketplace de cada Azure Stack do Hub. Para saber mais sobre a distribuição do Marketplace, confira [baixar itens do Marketplace para Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.

## <a name="get-the-docker-image"></a>Obter a imagem do Docker

As imagens do Docker para cada implantação eliminam os problemas de dependência entre diferentes versões do Azure PowerShell.

1. Verifique se Docker for Windows está usando contêineres do Windows.
2. Execute o comando a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Implantar os clusters

1. Depois que a imagem de contêiner tiver sido recebida com êxito, inicie a imagem.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Depois que o contêiner for iniciado, você receberá um terminal elevado do PowerShell no contêiner. Altere os diretórios para obter o script de implantação.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Execute a implantação. Forneça credenciais e nomes de recursos quando necessário. HA refere-se ao Hub de Azure Stack onde o cluster de alta disponibilidade será implantado. A recuperação de desastre refere-se ao Hub de Azure Stack onde o cluster de DR será implantado.

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-Hybrid" a serem instalados.

5. Os recursos de HA serão implantados primeiro. Monitore a implantação e aguarde sua conclusão. Quando você tiver a mensagem informando que a implantação de alta disponibilidade foi concluída, você poderá verificar o portal do hub de Azure Stack de HA para ver os recursos implantados.

6. Continue com a implantação de recursos de DR e decida se deseja habilitar uma caixa de salto no Hub de Azure Stack de recuperação de desastre para interagir com o cluster.

7. Aguarde a conclusão da implantação de recursos de recuperação de desastre.

8. Depois que a implantação de recursos de recuperação de desastre for concluída, saia do contêiner.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Próximas etapas

- Se você habilitou a VM da caixa de salto no Hub de Azure Stack de recuperação de desastre, poderá se conectar via SSH e interagir com o cluster do MongoDB instalando a CLI do Mongo. Para saber mais sobre como interagir com o MongoDB, consulte [o Shell Mongo](https://docs.mongodb.com/manual/mongo/).
- Para saber mais sobre os aplicativos de nuvem híbrida, consulte [soluções de nuvem híbrida.](https://aka.ms/azsdevtutorials)
- Modifique o código para este exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
