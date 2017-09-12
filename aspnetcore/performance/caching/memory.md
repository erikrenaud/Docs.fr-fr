---
title: Mise en cache dans ASP.NET Core
author: rick-anderson
description: "Montre comment mettre en cache les données en mémoire dans ASP.NET Core."
keywords: "ASP.NET Core, les performances en mémoire, de cache,"
ms.author: riande
manager: wpickett
ms.date: 12/14/2016
ms.topic: article
ms.assetid: 819511cf-d33e-410a-b5a9-bef7fa64d2f3
ms.technology: aspnet
ms.prod: asp.net-core
uid: performance/caching/memory
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1e2d43d837ba76c6ef8b5136f3751edb44d6606a
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/12/2017
---
# <a name="introduction-to-in-memory-caching-in-aspnet-core"></a><span data-ttu-id="a535a-104">Introduction à la mise en cache dans ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a535a-104">Introduction to in-memory caching in ASP.NET Core</span></span>

<span data-ttu-id="a535a-105">Par [Rick Anderson](https://twitter.com/RickAndMSFT), [John Luo](https://github.com/JunTaoLuo), et [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="a535a-105">By [Rick Anderson](https://twitter.com/RickAndMSFT), [John Luo](https://github.com/JunTaoLuo), and [Steve Smith](https://ardalis.com/)</span></span>

[<span data-ttu-id="a535a-106">Afficher ou télécharger l’exemple de code</span><span class="sxs-lookup"><span data-stu-id="a535a-106">View or download sample code</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/memory/sample)

## <a name="caching-basics"></a><span data-ttu-id="a535a-107">Principes fondamentaux de la mise en cache</span><span class="sxs-lookup"><span data-stu-id="a535a-107">Caching basics</span></span>

<span data-ttu-id="a535a-108">La mise en cache peut améliorer considérablement les performances et l’évolutivité d’une application en réduisant le travail requis pour générer le contenu.</span><span class="sxs-lookup"><span data-stu-id="a535a-108">Caching can significantly improve the performance and scalability of an app by reducing the work required to generate content.</span></span> <span data-ttu-id="a535a-109">Mise en cache fonctionne mieux avec les données qui sont rarement modifiées.</span><span class="sxs-lookup"><span data-stu-id="a535a-109">Caching works best with data that changes infrequently.</span></span> <span data-ttu-id="a535a-110">Mise en cache permet une copie des données qui peuvent être renvoyées beaucoup plus rapidement qu’à partir de la source d’origine.</span><span class="sxs-lookup"><span data-stu-id="a535a-110">Caching makes a copy of data that can be returned much faster than from the original source.</span></span> <span data-ttu-id="a535a-111">Vous devez écrire et tester votre application pour jamais dépendent des données mises en cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-111">You should write and test your app to never depend on cached data.</span></span>

<span data-ttu-id="a535a-112">ASP.NET Core prend en charge plusieurs caches différents.</span><span class="sxs-lookup"><span data-stu-id="a535a-112">ASP.NET Core supports several different caches.</span></span> <span data-ttu-id="a535a-113">Le cache de la plus simple est basé sur le [IMemoryCache](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache), qui représente un cache stocké dans la mémoire du serveur web.</span><span class="sxs-lookup"><span data-stu-id="a535a-113">The simplest cache is based on the [IMemoryCache](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache), which represents a cache stored in the memory of the web server.</span></span> <span data-ttu-id="a535a-114">Les applications qui s’exécutent sur une batterie de serveurs de plusieurs serveurs devraient vous assurer que les sessions rémanentes lors de l’utilisation du cache en mémoire.</span><span class="sxs-lookup"><span data-stu-id="a535a-114">Apps which run on a server farm of multiple servers should ensure that sessions are sticky when using the in-memory cache.</span></span> <span data-ttu-id="a535a-115">Sessions rémanentes Assurez-vous que les requêtes ultérieures à partir d’un client tous les dirigé vers le même serveur.</span><span class="sxs-lookup"><span data-stu-id="a535a-115">Sticky sessions ensure that subsequent requests from a client all go to the same server.</span></span> <span data-ttu-id="a535a-116">Par exemple, les utilisation d’applications Web Azure [Application Request Routing](https://www.iis.net/learn/extensions/planning-for-arr) (ARR) pour router toutes les demandes ultérieures au même serveur.</span><span class="sxs-lookup"><span data-stu-id="a535a-116">For example, Azure Web apps use [Application Request Routing](https://www.iis.net/learn/extensions/planning-for-arr) (ARR) to route all subsequent requests to the same server.</span></span>

<span data-ttu-id="a535a-117">Sessions non persistantes dans une batterie de serveurs web nécessitent un [cache distribué](distributed.md) pour éviter les problèmes de cohérence du cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-117">Non-sticky sessions in a web farm require a [distributed cache](distributed.md) to avoid cache consistency problems.</span></span> <span data-ttu-id="a535a-118">Pour certaines applications, un cache distribué peut prendre en charge la plus élevée de montée en charge qu’un cache en mémoire.</span><span class="sxs-lookup"><span data-stu-id="a535a-118">For some apps, a distributed cache can support higher scale out than an in-memory cache.</span></span> <span data-ttu-id="a535a-119">À l’aide d’un cache distribué permet de décharger la mémoire cache pour un processus externe.</span><span class="sxs-lookup"><span data-stu-id="a535a-119">Using a distributed cache offloads the cache memory to an external process.</span></span> 

<span data-ttu-id="a535a-120">Le `IMemoryCache` cache supprimez les entrées du cache mémoire insuffisante, sauf si le [cache priorité](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheitempriority) a la valeur `CacheItemPriority.NeverRemove`.</span><span class="sxs-lookup"><span data-stu-id="a535a-120">The `IMemoryCache` cache will evict cache entries under memory pressure unless the [cache priority](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheitempriority) is set to `CacheItemPriority.NeverRemove`.</span></span> <span data-ttu-id="a535a-121">Vous pouvez définir le `CacheItemPriority` pour ajuster la priorité, le cache supprime des éléments de sollicitation de la mémoire.</span><span class="sxs-lookup"><span data-stu-id="a535a-121">You can set the `CacheItemPriority` to adjust the priority the cache evicts items under memory pressure.</span></span>

<span data-ttu-id="a535a-122">Le cache en mémoire permettre stocker n’importe quel objet ; l’interface de cache distribué est limité à `byte[]`.</span><span class="sxs-lookup"><span data-stu-id="a535a-122">The in-memory cache can store any object; the distributed cache interface is limited to `byte[]`.</span></span>

## <a name="using-imemorycache"></a><span data-ttu-id="a535a-123">À l’aide de IMemoryCache</span><span class="sxs-lookup"><span data-stu-id="a535a-123">Using IMemoryCache</span></span>

<span data-ttu-id="a535a-124">La mise en cache en mémoire est un *service* qui est référencé à partir de votre application à l’aide de [Injection de dépendance](../../fundamentals/dependency-injection.md).</span><span class="sxs-lookup"><span data-stu-id="a535a-124">In-memory caching is a *service* that is referenced from your app using [Dependency Injection](../../fundamentals/dependency-injection.md).</span></span> <span data-ttu-id="a535a-125">Appelez `AddMemoryCache` dans `ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="a535a-125">Call `AddMemoryCache` in `ConfigureServices`:</span></span>

<span data-ttu-id="a535a-126">[!code-csharp[Main](memory/sample/WebCache/Startup.cs?highlight=8)]</span><span class="sxs-lookup"><span data-stu-id="a535a-126">[!code-csharp[Main](memory/sample/WebCache/Startup.cs?highlight=8)]</span></span> 

<span data-ttu-id="a535a-127">Demander le `IMemoryCache` instance dans le constructeur :</span><span class="sxs-lookup"><span data-stu-id="a535a-127">Request the `IMemoryCache` instance in the constructor:</span></span>

<span data-ttu-id="a535a-128">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor&highlight=3,5-)]</span><span class="sxs-lookup"><span data-stu-id="a535a-128">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor&highlight=3,5-)]</span></span> 

<span data-ttu-id="a535a-129">`IMemoryCache`nécessite le package NuGet « Microsoft.Extensions.Caching.Memory ».</span><span class="sxs-lookup"><span data-stu-id="a535a-129">`IMemoryCache` requires NuGet package "Microsoft.Extensions.Caching.Memory".</span></span>

<span data-ttu-id="a535a-130">Le code suivant utilise [TryGetValue](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) pour vérifier si l’heure actuelle est dans le cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-130">The following code uses [TryGetValue](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) to check if the current time is in the cache.</span></span> <span data-ttu-id="a535a-131">Si l’élément n’est pas mis en cache, une nouvelle entrée est créée et ajoutée au cache avec [définir](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_).</span><span class="sxs-lookup"><span data-stu-id="a535a-131">If the item is not cached, a new entry is created and added to the cache with [Set](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_).</span></span>

<span data-ttu-id="a535a-132">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="a535a-132">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]</span></span>

<span data-ttu-id="a535a-133">L’heure actuelle et l’heure de mise en cache s’affiche :</span><span class="sxs-lookup"><span data-stu-id="a535a-133">The current time and the cached time is displayed:</span></span>

<span data-ttu-id="a535a-134">[!code-html[Main](memory/sample/WebCache/Views/Home/Cache.cshtml)]</span><span class="sxs-lookup"><span data-stu-id="a535a-134">[!code-html[Main](memory/sample/WebCache/Views/Home/Cache.cshtml)]</span></span>

<span data-ttu-id="a535a-135">La mise en cache `DateTime` valeur reste dans le cache s’il existe des demandes dans le délai d’expiration (et aucune suppression en raison d’une sollicitation de la mémoire).</span><span class="sxs-lookup"><span data-stu-id="a535a-135">The cached `DateTime` value will remain in the cache while there are requests within the timeout period (and no eviction due to memory pressure).</span></span> <span data-ttu-id="a535a-136">L’illustration ci-dessous indique l’heure actuelle et une heure antérieure récupérés du cache :</span><span class="sxs-lookup"><span data-stu-id="a535a-136">The image below shows the current time and an older time retrieved from cache:</span></span>

![Vue d’index avec deux fois différentes affichées](memory/_static/time.png)

<span data-ttu-id="a535a-138">Le code suivant utilise [GetOrCreate](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) et [GetOrCreateAsync](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) en cache des données.</span><span class="sxs-lookup"><span data-stu-id="a535a-138">The following code uses [GetOrCreate](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) and [GetOrCreateAsync](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) to cache data.</span></span> 

<span data-ttu-id="a535a-139">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]</span><span class="sxs-lookup"><span data-stu-id="a535a-139">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]</span></span>

<span data-ttu-id="a535a-140">Le code suivant appelle [obtenir](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) pour extraire l’heure de mise en cache :</span><span class="sxs-lookup"><span data-stu-id="a535a-140">The following code calls [Get](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) to fetch the cached time:</span></span>

<span data-ttu-id="a535a-141">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]</span><span class="sxs-lookup"><span data-stu-id="a535a-141">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]</span></span>

<span data-ttu-id="a535a-142">Consultez [IMemoryCache méthodes](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache) et [CacheExtensions méthodes](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions) pour obtenir une description des méthodes du cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-142">See [IMemoryCache methods](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.imemorycache) and [CacheExtensions methods](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.cacheextensions) for a description of the cache methods.</span></span>

## <a name="using-memorycacheentryoptions"></a><span data-ttu-id="a535a-143">À l’aide de MemoryCacheEntryOptions</span><span class="sxs-lookup"><span data-stu-id="a535a-143">Using MemoryCacheEntryOptions</span></span>

<span data-ttu-id="a535a-144">L’exemple suivant :</span><span class="sxs-lookup"><span data-stu-id="a535a-144">The following sample:</span></span>

- <span data-ttu-id="a535a-145">Définit l’heure d’expiration absolue.</span><span class="sxs-lookup"><span data-stu-id="a535a-145">Sets the absolute expiration time.</span></span> <span data-ttu-id="a535a-146">Ceci est la durée maximale que l’entrée peut être mis en cache et empêche l’élément de devenir trop périmées lors de l’expiration décalée est constamment renouvelée.</span><span class="sxs-lookup"><span data-stu-id="a535a-146">This is the maximum time the entry can be cached and prevents the item from becoming too stale when the sliding expiration is continuously renewed.</span></span>
- <span data-ttu-id="a535a-147">Définit un délai d’expiration décalée.</span><span class="sxs-lookup"><span data-stu-id="a535a-147">Sets a sliding expiration time.</span></span> <span data-ttu-id="a535a-148">Les requêtes qui accèdent à cet élément de mise en cache réinitialise l’horloge d’expiration décalée.</span><span class="sxs-lookup"><span data-stu-id="a535a-148">Requests that access this cached item will reset the sliding expiration clock.</span></span>
- <span data-ttu-id="a535a-149">Définit la priorité de cache `CacheItemPriority.NeverRemove`.</span><span class="sxs-lookup"><span data-stu-id="a535a-149">Sets the cache priority to `CacheItemPriority.NeverRemove`.</span></span> 
- <span data-ttu-id="a535a-150">Définit un [PostEvictionDelegate](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.postevictiondelegate) qui est appelée après la suppression de l’entrée du cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-150">Sets a [PostEvictionDelegate](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.memory.postevictiondelegate) that will be called after the entry is evicted from the cache.</span></span> <span data-ttu-id="a535a-151">Le rappel est exécuté sur un thread différent du code qui supprime l’élément à partir du cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-151">The callback is run on a different thread from the code that removes the item from the cache.</span></span>

<span data-ttu-id="a535a-152">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-20)]</span><span class="sxs-lookup"><span data-stu-id="a535a-152">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-20)]</span></span>

## <a name="cache-dependencies"></a><span data-ttu-id="a535a-153">Dépendances de cache</span><span class="sxs-lookup"><span data-stu-id="a535a-153">Cache dependencies</span></span>

<span data-ttu-id="a535a-154">L’exemple suivant montre comment le point d’expirer une entrée de cache si une entrée de dépendance expire.</span><span class="sxs-lookup"><span data-stu-id="a535a-154">The following sample shows how to expire a cache entry if a dependent entry expires.</span></span> <span data-ttu-id="a535a-155">A `CancellationChangeToken` est ajouté à l’élément mis en cache.</span><span class="sxs-lookup"><span data-stu-id="a535a-155">A `CancellationChangeToken` is added to the cached item.</span></span> <span data-ttu-id="a535a-156">Lorsque `Cancel` est appelée sur le `CancellationTokenSource`, les deux entrées du cache sont supprimées.</span><span class="sxs-lookup"><span data-stu-id="a535a-156">When `Cancel` is called on the `CancellationTokenSource`, both cache entries are evicted.</span></span> 

<span data-ttu-id="a535a-157">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]</span><span class="sxs-lookup"><span data-stu-id="a535a-157">[!code-csharp[Main](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]</span></span>

<span data-ttu-id="a535a-158">À l’aide un `CancellationTokenSource` permet à plusieurs entrées de cache à supprimer en tant que groupe.</span><span class="sxs-lookup"><span data-stu-id="a535a-158">Using a `CancellationTokenSource` allows multiple cache entries to be evicted as a group.</span></span> <span data-ttu-id="a535a-159">Avec la `using` modèle dans le code ci-dessus, les entrées de cache créées à l’intérieur du `using` bloc hériteront des déclencheurs et les paramètres d’expiration.</span><span class="sxs-lookup"><span data-stu-id="a535a-159">With the `using` pattern in the code above, cache entries created inside the `using` block will inherit triggers and expiration settings.</span></span>

### <a name="additional-notes"></a><span data-ttu-id="a535a-160">Remarques supplémentaires</span><span class="sxs-lookup"><span data-stu-id="a535a-160">Additional notes</span></span>

- <span data-ttu-id="a535a-161">Lorsque vous utilisez un rappel pour remplir un élément de cache à :</span><span class="sxs-lookup"><span data-stu-id="a535a-161">When using a callback to repopulate a cache item:</span></span>

  - <span data-ttu-id="a535a-162">La valeur de clé mise en cache peuvent trouver vide plusieurs demandes étant donné que le rappel n’est pas terminée.</span><span class="sxs-lookup"><span data-stu-id="a535a-162">Multiple requests can find the cached key value empty because the callback hasn't completed.</span></span> 
  - <span data-ttu-id="a535a-163">Cela peut entraîner le remplissage de l’élément mis en cache de plusieurs threads.</span><span class="sxs-lookup"><span data-stu-id="a535a-163">This can result in several threads repopulating the cached item.</span></span>

- <span data-ttu-id="a535a-164">Lorsqu’une entrée de cache est utilisée pour créer un autre, l’enfant copie de l’entrée parente des jetons d’expiration et les paramètres d’expiration basés sur le temps.</span><span class="sxs-lookup"><span data-stu-id="a535a-164">When one cache entry is used to create another, the child copies the parent entry's expiration tokens and time-based expiration settings.</span></span> <span data-ttu-id="a535a-165">L’enfant n’est pas expiré par la suppression manuelle ou mise à jour de l’entrée parente.</span><span class="sxs-lookup"><span data-stu-id="a535a-165">The child is not expired by manual removal or updating of the parent entry.</span></span>

### <a name="other-resources"></a><span data-ttu-id="a535a-166">Autres ressources</span><span class="sxs-lookup"><span data-stu-id="a535a-166">Other Resources</span></span>

* [<span data-ttu-id="a535a-167">Utilisation d’un cache distribué</span><span class="sxs-lookup"><span data-stu-id="a535a-167">Working with a Distributed Cache</span></span>](distributed.md)
* [<span data-ttu-id="a535a-168">Intergiciel (middleware) mise en cache de réponse</span><span class="sxs-lookup"><span data-stu-id="a535a-168">Response caching middleware</span></span>](middleware.md)