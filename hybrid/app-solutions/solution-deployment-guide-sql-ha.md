---
title: Implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack
description: Saiba como implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909808"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Implantar um grupo de disponibilidade SQL Server 2016 no Azure e no Hub de Azure Stack

Este artigo o orientará em uma implantação automatizada de um cluster de HA (alta disponibilidade) SQL Server 2016 Enterprise com um site de DR (recuperação de desastre assíncrono) em dois ambientes de Hub de Azure Stack. Para saber mais sobre o SQL Server 2016 e a alta disponibilidade, consulte [grupos de disponibilidade Always on: uma solução de alta disponibilidade e recuperação de desastres](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Orquestrar uma implantação em dois hubs de Azure Stack.
> - Use o Docker para minimizar problemas de dependência com perfis de API do Azure.
> - Implante um cluster do SQL Server 2016 Enterprise altamente disponível básico com um site de recuperação de desastre.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="architecture-for-sql-server-2016"></a>Arquitetura para SQL Server 2016

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Pré-requisitos para o SQL Server 2016

- Dois sistemas integrados de Hub de Azure Stack (Hub de Azure Stack) conectados. Essa implantação não funciona no Kit de Desenvolvimento do Azure Stack (ASDK). Para saber mais sobre o Hub de Azure Stack, consulte a [visão geral Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Uma assinatura de locatário em cada Hub de Azure Stack.
  - **Anote cada ID de assinatura e o ponto de extremidade de Azure Resource Manager para cada Hub de Azure Stack.**
- Uma entidade de serviço Azure Active Directory (AD do Azure) que tem permissões para a assinatura de locatário em cada Hub de Azure Stack. Talvez seja necessário criar duas entidades de serviço se os hubs de Azure Stack forem implantados em diferentes locatários do Azure AD. Para saber como criar uma entidade de serviço para Azure Stack Hub, consulte [criar entidades de serviço para fornecer aos aplicativos acesso aos recursos do Hub Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).
  - **Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) da entidade de serviço.**
- SQL Server 2016 Enterprise distribuído para cada Marketplace de Azure Stack do Hub. Para saber mais sobre a distribuição do Marketplace, confira [baixar itens do Marketplace para Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Verifique se sua organização tem as licenças SQL apropriadas.**
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.

## <a name="get-the-docker-image"></a>Obter a imagem do Docker

As imagens do Docker para cada implantação eliminam os problemas de dependência entre diferentes versões do Azure PowerShell.

1. Verifique se Docker for Windows está usando contêineres do Windows.
2. Execute o script a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Implantar o grupo de disponibilidade

1. Depois que a imagem de contêiner tiver sido recebida com êxito, inicie a imagem.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Depois que o contêiner for iniciado, você receberá um terminal elevado do PowerShell no contêiner. Altere os diretórios para obter o script de implantação.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Execute a implantação. Forneça credenciais e nomes de recursos quando necessário. HA refere-se ao Hub de Azure Stack onde o cluster de alta disponibilidade será implantado. A recuperação de desastre refere-se ao Hub de Azure Stack onde o cluster de DR será implantado.

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

5. Aguarde a conclusão da implantação de recursos.

6. Após a conclusão da implantação de recursos de recuperação de desastre, saia do contêiner.

      ```powershell
      exit
      ```

7. Inspecione a implantação exibindo os recursos em cada portal do hub de Azure Stack. Conecte-se a uma das instâncias do SQL no ambiente de HA e inspecione o grupo de disponibilidade por meio do SQL Server Management Studio (SSMS).

    ![2016 SQL HA de SQL Server](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Próximas etapas

- Use SQL Server Management Studio para fazer failover manual do cluster. Consulte [executar um failover manual forçado de um grupo de disponibilidade de Always on (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Saiba mais sobre os aplicativos de nuvem híbrida. Consulte [soluções de nuvem híbrida.](https://aka.ms/azsdevtutorials)
- Use seus próprios dados ou modifique o código para este exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
