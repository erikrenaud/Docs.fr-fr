---
title: "Vue d’ensemble des API de consommateur"
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: f69beb9d-a519-43a8-857c-f6b01886a903
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/overview
ms.openlocfilehash: d23a6ce50eef71f393124b9420f4ba473904d8b4
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/11/2017
---
# <a name="consumer-apis-overview"></a><span data-ttu-id="9db57-103">Vue d’ensemble des API de consommateur</span><span class="sxs-lookup"><span data-stu-id="9db57-103">Consumer APIs overview</span></span>

<span data-ttu-id="9db57-104">Les interfaces IDataProtectionProvider et IDataProtector sont les interfaces de base par le biais duquel les consommateurs utilisent le système de protection des données.</span><span class="sxs-lookup"><span data-stu-id="9db57-104">The IDataProtectionProvider and IDataProtector interfaces are the basic interfaces through which consumers use the data protection system.</span></span> <span data-ttu-id="9db57-105">Ils se trouvent dans le package Microsoft.AspNetCore.DataProtection.Abstractions.</span><span class="sxs-lookup"><span data-stu-id="9db57-105">They are located in the Microsoft.AspNetCore.DataProtection.Abstractions package.</span></span>

## <a name="idataprotectionprovider"></a><span data-ttu-id="9db57-106">IDataProtectionProvider</span><span class="sxs-lookup"><span data-stu-id="9db57-106">IDataProtectionProvider</span></span>

<span data-ttu-id="9db57-107">L’interface du fournisseur représente la racine du système de protection des données.</span><span class="sxs-lookup"><span data-stu-id="9db57-107">The provider interface represents the root of the data protection system.</span></span> <span data-ttu-id="9db57-108">Il ne peut pas être utilisé directement pour protéger ou ôter la protection des données.</span><span class="sxs-lookup"><span data-stu-id="9db57-108">It cannot directly be used to protect or unprotect data.</span></span> <span data-ttu-id="9db57-109">Au lieu de cela, le consommateur doit obtenir une référence à un IDataProtector en appelant IDataProtectionProvider.CreateProtector(purpose), où objectif est une chaîne qui décrit le cas d’utilisation prévue de consommateur.</span><span class="sxs-lookup"><span data-stu-id="9db57-109">Instead, the consumer must get a reference to an IDataProtector by calling IDataProtectionProvider.CreateProtector(purpose), where purpose is a string that describes the intended consumer use case.</span></span> <span data-ttu-id="9db57-110">Consultez [objectif chaînes](purpose-strings.md) pour plus d’informations sur l’intention de ce paramètre et comment choisir une valeur appropriée.</span><span class="sxs-lookup"><span data-stu-id="9db57-110">See [Purpose Strings](purpose-strings.md) for much more information on the intent of this parameter and how to choose an appropriate value.</span></span>

## <a name="idataprotector"></a><span data-ttu-id="9db57-111">IDataProtector</span><span class="sxs-lookup"><span data-stu-id="9db57-111">IDataProtector</span></span>

<span data-ttu-id="9db57-112">L’interface de protecteur est retourné par un appel à CreateProtector, et c’est cette interface dans laquelle les utilisateurs peuvent utiliser pour effectuer protéger et déprotéger les opérations.</span><span class="sxs-lookup"><span data-stu-id="9db57-112">The protector interface is returned by a call to CreateProtector, and it is this interface which consumers can use to perform protect and unprotect operations.</span></span>

<span data-ttu-id="9db57-113">Pour protéger une partie des données, passer les données à la méthode de protection.</span><span class="sxs-lookup"><span data-stu-id="9db57-113">To protect a piece of data, pass the data to the Protect method.</span></span> <span data-ttu-id="9db57-114">L’interface de base définit une méthode qui convertit byte [-] -> byte [], mais il existe également une surcharge (fournie en tant qu’une méthode d’extension) qui convertit la chaîne -> chaîne.</span><span class="sxs-lookup"><span data-stu-id="9db57-114">The basic interface defines a method which converts byte[] -> byte[], but there is also an overload (provided as an extension method) which converts string -> string.</span></span> <span data-ttu-id="9db57-115">La sécurité offerte par les deux méthodes est identique. le développeur doit choisir quelle que soit la surcharge est plus pratique pour les cas d’utilisation.</span><span class="sxs-lookup"><span data-stu-id="9db57-115">The security offered by the two methods is identical; the developer should choose whichever overload is most convenient for their use case.</span></span> <span data-ttu-id="9db57-116">Quelle que soit la surcharge choisie, la valeur retournée par la protection méthode est désormais protégée (enciphered et répondre à tous les systèmes), et l’application peut envoyer à un client non fiable.</span><span class="sxs-lookup"><span data-stu-id="9db57-116">Irrespective of the overload chosen, the value returned by the Protect method is now protected (enciphered and tamper-proofed), and the application can send it to an untrusted client.</span></span>

<span data-ttu-id="9db57-117">Pour ôter la protection d’une donnée précédemment protégé, passer les données protégées à la méthode de suppression de la protection.</span><span class="sxs-lookup"><span data-stu-id="9db57-117">To unprotect a previously-protected piece of data, pass the protected data to the Unprotect method.</span></span> <span data-ttu-id="9db57-118">(Il n’y byte []-basé sur chaîne et en fonction des surcharges pour des raisons pratiques de développement.) Si la charge utile protégée a été générée par un appel précédent à protéger sur ce même IDataProtector, la méthode Unprotect retournera la charge non protégée d’origine.</span><span class="sxs-lookup"><span data-stu-id="9db57-118">(There are byte[]-based and string-based overloads for developer convenience.) If the protected payload was generated by an earlier call to Protect on this same IDataProtector, the Unprotect method will return the original unprotected payload.</span></span> <span data-ttu-id="9db57-119">Si la charge utile protégée a été falsifiée ou a été créée par un IDataProtector différents, la méthode Unprotect lèvera CryptographicException.</span><span class="sxs-lookup"><span data-stu-id="9db57-119">If the protected payload has been tampered with or was produced by a different IDataProtector, the Unprotect method will throw CryptographicException.</span></span>

<span data-ttu-id="9db57-120">Le concept de même et différents IDataProtector se réfère au concept d’objectif.</span><span class="sxs-lookup"><span data-stu-id="9db57-120">The concept of same vs. different IDataProtector ties back to the concept of purpose.</span></span> <span data-ttu-id="9db57-121">Si deux instances de IDataProtector ont été générés à partir de la même racine IDataProtectionProvider mais via des chaînes d’objectif différent dans l’appel à IDataProtectionProvider.CreateProtector, ils sont considérés comme [différents protecteurs](purpose-strings.md), et une ne pourrez pas ôter la protection des charges utiles générées par l’autre.</span><span class="sxs-lookup"><span data-stu-id="9db57-121">If two IDataProtector instances were generated from the same root IDataProtectionProvider but via different purpose strings in the call to IDataProtectionProvider.CreateProtector, then they are considered [different protectors](purpose-strings.md), and one will not be able to unprotect payloads generated by the other.</span></span>

## <a name="consuming-these-interfaces"></a><span data-ttu-id="9db57-122">Consommation de ces interfaces.</span><span class="sxs-lookup"><span data-stu-id="9db57-122">Consuming these interfaces</span></span>

<span data-ttu-id="9db57-123">Pour un composant prenant en charge DI, l’utilisation prévue est que le composant prendre un paramètre de IDataProtectionProvider dans son constructeur, et que le système DI fournit automatiquement ce service lorsque le composant est instancié.</span><span class="sxs-lookup"><span data-stu-id="9db57-123">For a DI-aware component, the intended usage is that the component take an IDataProtectionProvider parameter in its constructor and that the DI system automatically provides this service when the component is instantiated.</span></span>

> [!NOTE]
> <span data-ttu-id="9db57-124">Certaines applications (telles que les applications console ou les applications ASP.NET 4.x) ne peuvent pas être DI prenant en charge ne pouvez pas utiliser le mécanisme décrit ici.</span><span class="sxs-lookup"><span data-stu-id="9db57-124">Some applications (such as console applications or ASP.NET 4.x applications) might not be DI-aware so cannot use the mechanism described here.</span></span> <span data-ttu-id="9db57-125">Pour ces scénarios, consultez le [Non scénarios DI](../configuration/non-di-scenarios.md) document pour plus d’informations sur l’obtention d’une instance d’un fournisseur d’IDataProtection sans passer par DI.</span><span class="sxs-lookup"><span data-stu-id="9db57-125">For these scenarios consult the [Non DI Aware Scenarios](../configuration/non-di-scenarios.md) document for more information on getting an instance of an IDataProtection provider without going through DI.</span></span>

<span data-ttu-id="9db57-126">L’exemple suivant illustre trois concepts :</span><span class="sxs-lookup"><span data-stu-id="9db57-126">The following sample demonstrates three concepts:</span></span>

1. <span data-ttu-id="9db57-127">[Ajout du système de protection des données](../configuration/overview.md) au conteneur de service,</span><span class="sxs-lookup"><span data-stu-id="9db57-127">[Adding the data protection system](../configuration/overview.md) to the service container,</span></span>

2. <span data-ttu-id="9db57-128">À l’aide de DI pour recevoir une instance d’un IDataProtectionProvider, et</span><span class="sxs-lookup"><span data-stu-id="9db57-128">Using DI to receive an instance of an IDataProtectionProvider, and</span></span>

3. <span data-ttu-id="9db57-129">Création d’un IDataProtector à partir d’un IDataProtectionProvider et son utilisation pour protéger et déprotéger les données.</span><span class="sxs-lookup"><span data-stu-id="9db57-129">Creating an IDataProtector from an IDataProtectionProvider and using it to protect and unprotect data.</span></span>

<span data-ttu-id="9db57-130">[!code-csharp[Main](../using-data-protection/samples/protectunprotect.cs?highlight=26,34,35,36,37,38,39,40)]</span><span class="sxs-lookup"><span data-stu-id="9db57-130">[!code-csharp[Main](../using-data-protection/samples/protectunprotect.cs?highlight=26,34,35,36,37,38,39,40)]</span></span>

<span data-ttu-id="9db57-131">Le package Microsoft.AspNetCore.DataProtection.Abstractions contient une méthode d’extension IServiceProvider.GetDataProtector en tant que développeur pour des raisons pratiques.</span><span class="sxs-lookup"><span data-stu-id="9db57-131">The package Microsoft.AspNetCore.DataProtection.Abstractions contains an extension method IServiceProvider.GetDataProtector as a developer convenience.</span></span> <span data-ttu-id="9db57-132">Elle encapsule comme une seule opération à la fois la récupération d’un IDataProtectionProvider à partir du fournisseur de service et en appelant IDataProtectionProvider.CreateProtector.</span><span class="sxs-lookup"><span data-stu-id="9db57-132">It encapsulates as a single operation both retrieving an IDataProtectionProvider from the service provider and calling IDataProtectionProvider.CreateProtector.</span></span> <span data-ttu-id="9db57-133">L’exemple suivant illustre son utilisation.</span><span class="sxs-lookup"><span data-stu-id="9db57-133">The following sample demonstrates its usage.</span></span>

<span data-ttu-id="9db57-134">[!code-csharp[Main](./overview/samples/getdataprotector.cs?highlight=15)]</span><span class="sxs-lookup"><span data-stu-id="9db57-134">[!code-csharp[Main](./overview/samples/getdataprotector.cs?highlight=15)]</span></span>

>[!TIP]
> <span data-ttu-id="9db57-135">Instances de IDataProtectionProvider et IDataProtector sont thread-safe pour les appelants plusieurs.</span><span class="sxs-lookup"><span data-stu-id="9db57-135">Instances of IDataProtectionProvider and IDataProtector are thread-safe for multiple callers.</span></span> <span data-ttu-id="9db57-136">Il est prévu qu’une fois qu’un composant obtient une référence à un IDataProtector via un appel à CreateProtector, il utilisera cette référence pour les appels multiples à protéger et Unprotect Unprotect.A appel lève CryptographicException si la charge utile protégée ne peut pas être vérifié ou déchiffrés.</span><span class="sxs-lookup"><span data-stu-id="9db57-136">It is intended that once a component gets a reference to an IDataProtector via a call to CreateProtector, it will use that reference for multiple calls to Protect and Unprotect.A call to Unprotect will throw CryptographicException if the protected payload cannot be verified or deciphered.</span></span> <span data-ttu-id="9db57-137">Certains composants peuvent souhaiter ignorer les erreurs pendant les opérations ; ôter la protection un composant qui lit les cookies d’authentification peut gérer cette erreur et traiter la demande comme s’il n’avait aucun cookie tout plutôt qu’échouer la requête ferme.</span><span class="sxs-lookup"><span data-stu-id="9db57-137">Some components may wish to ignore errors during unprotect operations; a component which reads authentication cookies might handle this error and treat the request as if it had no cookie at all rather than fail the request outright.</span></span> <span data-ttu-id="9db57-138">Les composants dont vous souhaitez que ce comportement doivent spécifiquement intercepter CryptographicException au lieu d’absorber toutes les exceptions.</span><span class="sxs-lookup"><span data-stu-id="9db57-138">Components which want this behavior should specifically catch CryptographicException instead of swallowing all exceptions.</span></span>