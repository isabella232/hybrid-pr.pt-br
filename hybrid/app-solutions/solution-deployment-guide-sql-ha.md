---
title: Implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub
description: Saiba como implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852466"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Implantar um grupo de disponibilidade do SQL Server 2016 no Azure e Azure Stack Hub

Este artigo guiará você na implantação automatizada de um cluster SQL Server 2016 Enterprise básico de alta disponibilidade (HA) com um site de DR (recuperação de desastre) assíncrona em dois ambientes do Azure Stack Hub. Para saber mais sobre o SQL Server 2016 e a alta disponibilidade, confira [Grupos de disponibilidade Always On: uma solução de alta disponibilidade e recuperação de desastre](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Coordenar uma implantação em dois Azure Stack Hubs.
> - Usar o Docker para minimizar problemas de dependência com perfis de API do Azure.
> - Implantar um cluster SQL Server 2016 Enterprise básico e de alta disponibilidade com um site de recuperação de desastre.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) analisa os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="architecture-for-sql-server-2016"></a>Arquitetura para o SQL Server2016

![Azure Stack Hub de alta disponibilidade para o SQL Server 2016](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Pré -requisitos do SQL Server 2016

- Dois sistemas integrados e conectados do Azure Stack Hub (Azure Stack Hub). Essa implantação não funciona no ASDK (Kit de Desenvolvimento do Azure Stack). Para saber mais sobre o Azure Stack Hub, confira [Visão geral do Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Uma assinatura de locatário em cada Azure Stack Hub.
  - **Anote cada ID de assinatura e o ponto de extremidade do Azure Resource Manager para cada Azure Stack Hub.**
- Uma entidade de serviço do Azure Active Directory (Azure AD) com permissões para a assinatura de locatário em cada Azure Stack Hub. Você pode ter que criar duas entidades de serviço se os Azure Stack Hubs forem implantados em diferentes locatários do Azure AD. Para saber como criar uma entidade de serviço para o Azure Stack Hub, confira [Criar entidades de serviço para permitir que os aplicativos acessem recursos do Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).
  - **Anote a ID do aplicativo, o segredo do cliente e o nome do locatário (xxxxx.onmicrosoft.com) de cada entidade de serviço.**
- O SQL Server 2016 Enterprise é sindicalizado para o marketplace de cada Azure Stack Hub. Para saber mais sobre a sindicalização de marketplace, confira [Baixar itens do marketplace para o Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Verifique se sua organização tem as licenças do SQL apropriadas.**
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) instalado no computador local.

## <a name="get-the-docker-image"></a>Obter a imagem do Docker

Ter imagens do Docker para cada implantação elimina os problemas de dependência entre diferentes versões do Azure PowerShell.

1. Verifique se o Docker for Windows está usando contêineres do Windows.
2. Execute o script a seguir em um prompt de comando elevado para obter o contêiner do Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Implante o grupo de disponibilidade

1. Inicie a imagem de contêiner após ela ser recebida.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Após o contêiner ser iniciado, você verá um terminal elevado do PowerShell. Mude de diretório para acessar o script de implantação.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Execute a implantação. Forneça credenciais e nomes de recursos quando necessário. HA se refere ao Azure Stack Hub onde o cluster de alta disponibilidade será implantado. DR se refere ao Azure Stack Hub onde o cluster de recuperação de desastre será implantado.

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

4. Digite `Y` para permitir que o provedor do NuGet seja instalado, o que iniciará os módulos do perfil de API "2018-03-01-hybrid" a serem instalados.

5. Aguarde até que a implantação do recurso seja concluída.

6. Após a implantação do recurso de recuperação de desastre ser concluída, saia do contêiner.

      ```powershell
      exit
      ```

7. Inspecione a implantação visualizando os recursos no portal de cada Azure Stack Hub. Conecte-se a uma das instâncias do SQL no ambiente de HA e inspecione o grupo de disponibilidade por meio do SSMS (SQL Server Management Studio).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Próximas etapas

- Use o SQL Server Management Studio para fazer o failover manual do cluster. Confira [Executar um failover manual forçado de um grupo de disponibilidade Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Saiba mais sobre aplicativos de nuvem híbrida. Confira [Soluções de Nuvem Híbrida.](/azure-stack/user/)
- Use seus próprios dados ou modifique o código deste exemplo no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).