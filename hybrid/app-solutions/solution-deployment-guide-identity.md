---
title: Configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack
description: Saiba como configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909847"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurar a identidade de nuvem híbrida para aplicativos do Azure e do hub de Azure Stack

Saiba como configurar uma identidade de nuvem híbrida para seus aplicativos do Azure e do hub de Azure Stack.

Você tem duas opções para conceder acesso aos seus aplicativos no Azure global e no Hub de Azure Stack.

 * Quando Azure Stack Hub tem uma conexão contínua com a Internet, você pode usar o Azure Active Directory (AD do Azure).
 * Quando Azure Stack Hub é desconectado da Internet, você pode usar o AD FS (Serviços Federados do Azure Directory).

Você usa entidades de serviço para conceder acesso aos seus aplicativos de Hub de Azure Stack para implantação ou configuração usando o Azure Resource Manager no Hub de Azure Stack.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Estabelecer uma identidade híbrida no Azure global e no Hub de Azure Stack
> - Recupere um token para acessar a API do hub de Azure Stack.

Você deve ter Azure Stack permissões de operador de Hub para as etapas nesta solução.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, habilitando a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [considerações de design de aplicativo híbrido](overview-app-design-considerations.md) revisa os pilares da qualidade do software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) para projetar, implantar e operar aplicativos híbridos. As considerações de design auxiliam na otimização do design de aplicativos híbridos, minimizando os desafios em ambientes de produção.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Criar uma entidade de serviço para o Azure AD no portal

Se você implantou o Hub de Azure Stack usando o Azure AD como o armazenamento de identidade, você pode criar entidades de serviço da mesma forma que faz para o Azure. [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) mostra como executar as etapas por meio do Portal. Verifique se você tem as [permissões necessárias do Azure ad](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Criar uma entidade de serviço para AD FS usando o PowerShell

Se você implantou Azure Stack Hub com AD FS, você pode usar o PowerShell para criar uma entidade de serviço, atribuir uma função de acesso e entrar do PowerShell usando essa identidade. [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) mostra como executar as etapas necessárias usando o PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Usando a API do hub de Azure Stack

A solução de [API do hub Azure Stack](/azure-stack/user/azure-stack-rest-api-use.md) orienta você pelo processo de recuperação de um token para acessar a API do hub de Azure Stack.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Conectar-se ao Hub de Azure Stack usando o PowerShell

O início rápido [para começar a executar o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) orienta você pelas etapas necessárias para instalar o Azure PowerShell e conectar-se à instalação do Hub do Azure Stack.

### <a name="prerequisites"></a>Pré-requisitos

Você precisa de uma instalação de Hub de Azure Stack conectada ao Azure AD com uma assinatura que você pode acessar. Se você não tiver uma instalação de Hub de Azure Stack, poderá usar estas instruções para configurar um [Kit de desenvolvimento do Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Conectar-se ao Hub de Azure Stack usando código

Para se conectar ao Hub de Azure Stack usando código, use a API de pontos de extremidade de Azure Resource Manager para obter os pontos de extremidade de autenticação e de grafo para a instalação do hub de Azure Stack. Em seguida, autentique usando solicitações REST. Você pode encontrar um aplicativo cliente de exemplo no [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>A menos que o SDK do Azure para sua linguagem de escolha dê suporte a perfis de API do Azure, o SDK pode não funcionar com o Hub de Azure Stack. Para saber mais sobre os perfis de API do Azure, consulte o artigo [gerenciar perfis de versão da API](/azure-stack/user/azure-stack-version-profiles.md) .

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre como a identidade é manipulada no Hub Azure Stack, consulte [arquitetura de identidade para Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).
- Para saber mais sobre os padrões de nuvem do Azure, consulte [padrões de design de nuvem](https://docs.microsoft.com/azure/architecture/patterns).
