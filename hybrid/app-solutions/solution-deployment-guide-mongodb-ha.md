---
title: Implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub
description: Saiba como implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477262"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Implantar uma solução MongoDB de alta disponibilidade no Azure e Azure Stack Hub

Este artigo guiará você na implantação automatizada de um cluster MongoDB básico de alta disponibilidade (HA) com um site de DR (recuperação de desastre) em dois ambientes do Azure Stack Hub. Para saber mais sobre o MongoDB e a alta disponibilidade, confira [Membros do conjunto de réplicas](https://docs.mongodb.com/manual/core/replica-set-members/).

Nesta solução, você criará um exemplo de ambiente para:

> [!div class="checklist"]
> - Coordenar uma implantação em dois Azure Stack Hubs.
> - Usar o Docker para minimizar problemas de dependência com perfis de API do Azure.
> - Implantar um cluster MongoDB básico e de alta disponibilidade com um site de recuperação de desastre.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) analisa os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Arquitetura para o MongoDB com o Azure Stack Hub

![arquitetura de alta disponibilidade do MongoDB no Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Pré-requisitos do MongoDB com o Azure Stack Hub

- Dois sistemas integrados e conectados do Azure Stack Hub (Azure Stack Hub). Essa implantação não funciona no ASDK (Kit de Desenvolvimento do Azure Stack). Para saber mais sobre o Azure Stack Hub, confira [O que é o Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Uma assinatura de locatário em cada Azure Stack Hub. 
  - **Anote cada ID de assinatura e o ponto de extremidade do Azure Resource Manager para cada Azure Stack Hub.**
- Uma entidade de serviço do Azure Active Directory (Azure AD) com permissões para a assinatura de locatário em cada Azure Stack Hub. Você pode ter que criar duas entidades de serviço se os Azure Stack Hubs forem implantados em diferentes locatários do Azure AD. Para saber como criar uma entidade de serviço para o Azure Stack Hub, confira [Usar uma identidade de aplicativo para acessar recursos do Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).
  - **Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) de cada entidade de serviço.**
- O Ubuntu 16.04 é sindicalizado para o marketplace de cada Azure Stack Hub. Para saber mais sobre a sindicalização de marketplace, confira [Baixar itens do marketplace para o Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.

## <a name="get-the-docker-image"></a>Obter a imagem do Docker

Ter imagens do Docker para cada implantação elimina os problemas de dependência entre diferentes versões do Azure PowerShell.

1. Verifique se o Docker for Windows está usando contêineres do Windows.
2. Execute o comando a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Implante os clusters

1. Inicie a imagem de contêiner após ela ser recebida.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Após o contêiner ser iniciado, você verá um terminal elevado do PowerShell. Mude de diretório para acessar o script de implantação.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Execute a implantação. Forneça credenciais e nomes de recursos quando necessário. HA se refere ao Azure Stack Hub onde o cluster de alta disponibilidade será implantado. DR se refere ao Azure Stack Hub onde o cluster de recuperação de desastre será implantado.

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

4. Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-hybrid" a serem instalados.

5. Os recursos de HA serão implantados primeiro. Monitore a implantação e aguarde sua conclusão. Quando você receber a mensagem informando que a implantação HA foi concluída, poderá verificar no portal do Azure Stack Hub de alta disponibilidade para ver os recursos implantados.

6. Continue com a implantação dos recursos de DR e decida se deseja habilitar uma caixa de atalhos no Azure Stack Hub de recuperação de desastre para interagir com o cluster.

7. Aguarde até que a implantação do recurso de recuperação de desastre seja concluída.

8. Após a implantação do recurso de recuperação de desastre ser concluída, saia do contêiner.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Próximas etapas

- Se você habilitou a VM de caixa de atalhos no Azure Stack Hub de recuperação de desastre, poderá se conectar via SSH e interagir com o cluster do MongoDB instalando a CLI do Mongo. Para saber mais sobre como interagir com o MongoDB, confira [O Mongo Shell](https://docs.mongodb.com/manual/mongo/).
- Para saber mais sobre os aplicativos de nuvem híbrida, confira [Soluções de Nuvem Híbrida.](https://aka.ms/azsdevtutorials)
- Modifique o código deste exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
