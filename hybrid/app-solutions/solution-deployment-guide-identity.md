---
title: Configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub
description: Saiba como configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895339"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurar a identidade de nuvem híbrida para aplicativos do Azure e do Azure Stack Hub

Saiba como configurar uma identidade de nuvem híbrida para seus aplicativos do Azure e do Azure Stack Hub.

Você tem duas opções para conceder acesso aos seus aplicativos tanto no Azure global quanto no Azure Stack Hub.

 * Quando o Azure Stack Hub tiver uma conexão contínua com a Internet, você poderá usar o Azure Active Directory (Azure AD).
 * Quando o Azure Stack Hub estiver desconectado da Internet, use o AD FS (Serviços de Federação do Azure Directory).

Você pode usar entidades de serviço para conceder acesso aos seus aplicativos do Azure Stack Hub para fins de implantação ou configuração usando o Azure Resource Manager no Azure Stack Hub.

Nesta solução, você criará um ambiente de exemplo para:

> [!div class="checklist"]
> - Estabelecer uma identidade híbrida no Azure global e no Azure Stack Hub
> - Recuperar um token para acessar a API do Azure Stack Hub.

Você precisa ter permissões de operador do Azure Stack Hub para executar as etapas desta solução.

> [!Tip]  
> ![Diagrama dos pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> O Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e a inovação da computação em nuvem para seu ambiente local, fornecendo a única nuvem híbrida que permite criar e implantar aplicativos híbridos em qualquer lugar.  
> 
> O artigo [Considerações de design de aplicativos híbridos](overview-app-design-considerations.md) examina os pilares da qualidade de software (posicionamento, escalabilidade, disponibilidade, resiliência, capacidade de gerenciamento e segurança) relativos ao design, à implantação e à operação de aplicativos híbridos. As considerações de design ajudam na otimização do design de aplicativos híbridos, reduzindo os desafios nos ambientes de produção.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Crie uma entidade de serviço para o Azure AD no portal

Se você implantou o Azure Stack Hub usando o Azure AD como armazenamento de identidade, pode criar entidades de serviço da mesma forma que faz para o Azure. [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) mostra como executar as etapas por meio do portal. Verifique se você tem as [permissões necessárias do Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Crie uma entidade de serviço para o AD FS usando o PowerShell

Se você implantou o Azure Stack Hub com o AD FS, pode usar o PowerShell para criar uma entidade de serviço, atribuir uma função de acesso e entrar a partir do PowerShell usando essa identidade. [Usar uma identidade de aplicativo para acessar recursos](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) mostra como executar as etapas necessárias usando o PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Usando a API do Azure Stack Hub API

A solução [API do Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use) guiará você pelo processo de recuperação de um token para acessar a API do Azure Stack Hub.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Conecte-se com o Azure Stack Hub usando o PowerShell

O início rápido [para começar a executar o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) guiará você pelas etapas necessárias para instalar o Azure PowerShell e conectar-se à instalação do Azure Stack Hub.

### <a name="prerequisites"></a>Pré-requisitos

É preciso uma instalação do Azure Stack Hub conectada ao Azure AD com uma assinatura que você possa acessar. Se você não tiver uma instalação do Azure Stack Hub, poderá usar estas instruções para configurar um [ASDK (Kit de Desenvolvimento do Azure Stack)](/azure-stack/asdk/asdk-install).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Conecte-se com o Azure Stack Hub usando código

Para conectar-se com o Azure Stack Hub usando código, empregue a API de pontos de extremidade do Azure Resource Manager para obter os pontos de extremidade de autenticação e de grafo de sua instalação do Azure Stack Hub. Em seguida, autentique-se usando solicitações REST. Você pode encontrar um exemplo de aplicativo cliente no[GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>A menos que o SDK do Azure para sua linguagem de escolha dê suporte a perfis de API do Azure, o SDK pode não funcionar com o Azure Stack Hub. Para saber mais sobre os perfis de API do Azure, confira o artigo [gerenciar perfis de versão de API](/azure-stack/user/azure-stack-version-profiles).

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre como a identidade é manipulada no Azure Stack Hub, confira [Arquitetura de identidade do Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).
- Para saber mais sobre os padrões de nuvem do Azure, confira [Padrões de Design de Nuvem](/azure/architecture/patterns).
