---
title: Risolvere i problemi di localizzazione di ASP.NET Core
author: hishamco
description: Informazioni su come diagnosticare i problemi di localizzazione nelle app ASP.NET Core.
ms.date: 01/24/2019
uid: fundamentals/troubleshoot-aspnet-core-localization
ms.openlocfilehash: 73405f539c89d79096e7b536407cd9730679d478
ms.sourcegitcommit: 687ffb15ebe65379f75c84739ea851d5a0d788b7
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 03/26/2019
ms.locfileid: "58488994"
---
# <a name="troubleshoot-aspnet-core-localization"></a><span data-ttu-id="931fe-103">Risolvere i problemi di localizzazione di ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="931fe-103">Troubleshoot ASP.NET Core Localization</span></span>

<span data-ttu-id="931fe-104">Di [Hisham Bin Ateya](https://github.com/hishamco)</span><span class="sxs-lookup"><span data-stu-id="931fe-104">By [Hisham Bin Ateya](https://github.com/hishamco)</span></span>

<span data-ttu-id="931fe-105">Questo articolo include istruzioni per la diagnostica di problemi di localizzazione delle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="931fe-105">This article provides instructions on how to diagnose ASP.NET Core app localization issues.</span></span>

## <a name="localization-configuration-issues"></a><span data-ttu-id="931fe-106">Problemi di configurazione della localizzazione</span><span class="sxs-lookup"><span data-stu-id="931fe-106">Localization configuration issues</span></span>

<span data-ttu-id="931fe-107">**Ordine del middleware di localizzazione**</span><span class="sxs-lookup"><span data-stu-id="931fe-107">**Localization middleware order**</span></span>  
<span data-ttu-id="931fe-108">L'app potrebbe non essere localizzata perché il middleware di localizzazione non è ordinato come previsto.</span><span class="sxs-lookup"><span data-stu-id="931fe-108">The app may not localize because the localization middleware isn't ordered as expected.</span></span>

<span data-ttu-id="931fe-109">Per risolvere questo problema assicurarsi che il middleware di localizzazione sia registrato prima del middleware MVC.</span><span class="sxs-lookup"><span data-stu-id="931fe-109">To resolve this issue, ensure that localization middleware is registered before MVC middleware.</span></span> <span data-ttu-id="931fe-110">In caso contrario il middleware di localizzazione non viene applicato.</span><span class="sxs-lookup"><span data-stu-id="931fe-110">Otherwise, the localization middleware isn't applied.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddLocalization(options => options.ResourcesPath = "Resources");

    services.AddMvc();
}
```

<span data-ttu-id="931fe-111">**Percorso delle risorse di localizzazione non trovato**</span><span class="sxs-lookup"><span data-stu-id="931fe-111">**Localization resources path not found**</span></span>

<span data-ttu-id="931fe-112">**Le impostazioni cultura supportate in RequestCultureProvider non corrispondono con quelle registrate**</span><span class="sxs-lookup"><span data-stu-id="931fe-112">**Supported Cultures in RequestCultureProvider don't match with registered once**</span></span>  

## <a name="resource-file-naming-issues"></a><span data-ttu-id="931fe-113">Problemi di denominazione dei file di risorse</span><span class="sxs-lookup"><span data-stu-id="931fe-113">Resource file naming issues</span></span>

<span data-ttu-id="931fe-114">ASP.NET Core include regole predefinite e linee guida per la denominazione dei file di risorse di localizzazione, descritte in dettaglio [qui](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming).</span><span class="sxs-lookup"><span data-stu-id="931fe-114">ASP.NET Core has predefined rules and guidelines for localization resources file naming, which are described in detail [here](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming).</span></span>

## <a name="missing-resources"></a><span data-ttu-id="931fe-115">Risorse mancanti</span><span class="sxs-lookup"><span data-stu-id="931fe-115">Missing resources</span></span>

<span data-ttu-id="931fe-116">Le cause comuni per le quali non è possibile trovare le risorse sono:</span><span class="sxs-lookup"><span data-stu-id="931fe-116">Common causes of resources not being found include:</span></span>

- <span data-ttu-id="931fe-117">I nomi delle risorse presentano errori di ortografia nel file `resx` o nella richiesta del localizzatore.</span><span class="sxs-lookup"><span data-stu-id="931fe-117">Resource names are misspelled in either the `resx` file or the localizer request.</span></span>
- <span data-ttu-id="931fe-118">La risorsa non è presente in `resx` per alcune lingue, ma è presente per altre.</span><span class="sxs-lookup"><span data-stu-id="931fe-118">The resource is missing from the `resx` for some languages, but exists in others.</span></span>
- <span data-ttu-id="931fe-119">Se si verificano ancora problemi, controllare i messaggi del log di localizzazione (al livello di log `Debug`) per altre informazioni sulle risorse mancanti.</span><span class="sxs-lookup"><span data-stu-id="931fe-119">If you're still having trouble, check the localization log messages (which are at `Debug` log level) for more details about the missing resources.</span></span>

<span data-ttu-id="931fe-120">_**Suggerimento:** Quando si usa `CookieRequestCultureProvider`, verificare che le virgolette singole non vengano usate con le impostazioni cultura all'interno del valore del cookie di localizzazione. Ad esempio `c='en-UK'|uic='en-US'` è un valore di cookie non valido, mentre `c=en-UK|uic=en-US` è valido._</span><span class="sxs-lookup"><span data-stu-id="931fe-120">_**Hint:** When using `CookieRequestCultureProvider`, verify single quotes are not used with the cultures inside the localization cookie value. For example, `c='en-UK'|uic='en-US'` is an invalid cookie value, while `c=en-UK|uic=en-US` is a valid._</span></span>

## <a name="resources--class-libraries-issues"></a><span data-ttu-id="931fe-121">Problemi con le risorse e le librerie di classi</span><span class="sxs-lookup"><span data-stu-id="931fe-121">Resources & Class Libraries issues</span></span>

<span data-ttu-id="931fe-122">Per impostazione predefinita ASP.NET Core offre un modo per consentire alle librerie di classi di trovare i file di risorse tramite [ResourceLocationAttribute](/dotnet/api/microsoft.extensions.localization.resourcelocationattribute?view=aspnetcore-2.1).</span><span class="sxs-lookup"><span data-stu-id="931fe-122">ASP.NET Core by default provides a way to allow the class libraries to find their resource files via [ResourceLocationAttribute](/dotnet/api/microsoft.extensions.localization.resourcelocationattribute?view=aspnetcore-2.1).</span></span>

<span data-ttu-id="931fe-123">I problemi comuni con le librerie di classi includono:</span><span class="sxs-lookup"><span data-stu-id="931fe-123">Common issues with class libraries include:</span></span>
- <span data-ttu-id="931fe-124">L'assenza di `ResourceLocationAttribute` in una libreria di classi impedisce a `ResourceManagerStringLocalizerFactory` di trovare le risorse.</span><span class="sxs-lookup"><span data-stu-id="931fe-124">Missing the `ResourceLocationAttribute` in a class library will prevent `ResourceManagerStringLocalizerFactory` from discovering the resources.</span></span>
- <span data-ttu-id="931fe-125">Denominazione dei file di risorse.</span><span class="sxs-lookup"><span data-stu-id="931fe-125">Resource file naming.</span></span> <span data-ttu-id="931fe-126">Per altre informazioni, vedere la sezione [Problemi di denominazione dei file di risorse](#resource-file-naming-issues).</span><span class="sxs-lookup"><span data-stu-id="931fe-126">For more information, see [Resource file naming issues](#resource-file-naming-issues) section.</span></span>
- <span data-ttu-id="931fe-127">Modifica dello spazio dei nomi radice della libreria di classi.</span><span class="sxs-lookup"><span data-stu-id="931fe-127">Changing the root namespace of the class library.</span></span> <span data-ttu-id="931fe-128">Per altre informazioni, vedere la sezione [Problemi con lo spazio dei nomi radice](#root-namespace-issues).</span><span class="sxs-lookup"><span data-stu-id="931fe-128">For more information, see [Root Namespace issues](#root-namespace-issues) section.</span></span>

## <a name="customrequestcultureprovider-doesnt-work-as-expected"></a><span data-ttu-id="931fe-129">CustomRequestCultureProvider non funziona come previsto</span><span class="sxs-lookup"><span data-stu-id="931fe-129">CustomRequestCultureProvider doesn't work as expected</span></span>

<span data-ttu-id="931fe-130">La classe `RequestLocalizationOptions` dispone di tre provider predefiniti:</span><span class="sxs-lookup"><span data-stu-id="931fe-130">The `RequestLocalizationOptions` class has three default providers:</span></span>

1. `QueryStringRequestCultureProvider`
2. `CookieRequestCultureProvider`
3. `AcceptLanguageHeaderRequestCultureProvider`

<span data-ttu-id="931fe-131">[CustomRequestCultureProvider](/dotnet/api/microsoft.aspnetcore.localization.customrequestcultureprovider?view=aspnetcore-2.1) consente di personalizzare la modalità di fornitura delle impostazioni cultura di localizzazione nell'app.</span><span class="sxs-lookup"><span data-stu-id="931fe-131">The [CustomRequestCultureProvider](/dotnet/api/microsoft.aspnetcore.localization.customrequestcultureprovider?view=aspnetcore-2.1) allows you to customize how the localization culture is provided in your app.</span></span> <span data-ttu-id="931fe-132">`CustomRequestCultureProvider` viene usato quando i provider predefiniti non soddisfano i requisiti.</span><span class="sxs-lookup"><span data-stu-id="931fe-132">The `CustomRequestCultureProvider` is used when the default providers don't meet your requirements.</span></span>

- <span data-ttu-id="931fe-133">Un motivo comune per cui i il provider personalizzato non funziona correttamente è che non è il primo provider nell'elenco `RequestCultureProviders`.</span><span class="sxs-lookup"><span data-stu-id="931fe-133">A common reason custom provider don't work properly is that it isn't the first provider in the `RequestCultureProviders` list.</span></span> <span data-ttu-id="931fe-134">Per risolvere il problema:</span><span class="sxs-lookup"><span data-stu-id="931fe-134">To resolve this issue:</span></span>

- <span data-ttu-id="931fe-135">Inserire il provider personalizzato nella posizione 0 dell'elenco `RequestCultureProviders` come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="931fe-135">Insert the custom provider at the position 0 in the `RequestCultureProviders` list as the following:</span></span>

```csharp
options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
```

- <span data-ttu-id="931fe-136">Usare il metodo di estensione `AddInitialRequestCultureProvider` per impostare il provider personalizzato come provider iniziale.</span><span class="sxs-lookup"><span data-stu-id="931fe-136">Use `AddInitialRequestCultureProvider` extension method to set the custom provider as initial provider.</span></span>

## <a name="root-namespace-issues"></a><span data-ttu-id="931fe-137">Problemi con lo spazio dei nomi radice</span><span class="sxs-lookup"><span data-stu-id="931fe-137">Root Namespace issues</span></span>

<span data-ttu-id="931fe-138">Quando lo spazio dei nomi radice di un assembly è diverso dal nome dell'assembly, la localizzazione non funziona per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="931fe-138">When the root namespace of an assembly is different than the assembly name, localization doesn't work by default.</span></span> <span data-ttu-id="931fe-139">Per evitare questo problema usare [RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1), descritto in dettaglio [qui](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming).</span><span class="sxs-lookup"><span data-stu-id="931fe-139">To avoid this issue use [RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1), which is described in detail [here](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming)</span></span>

## <a name="resources--build-action"></a><span data-ttu-id="931fe-140">Risorse e azione di compilazione</span><span class="sxs-lookup"><span data-stu-id="931fe-140">Resources & Build Action</span></span>

<span data-ttu-id="931fe-141">Se si usano file di risorse per la localizzazione, è importante che questi dispongano di un'azione di compilazione appropriata.</span><span class="sxs-lookup"><span data-stu-id="931fe-141">If you use resource files for localization, it's important that they have an appropriate build action.</span></span> <span data-ttu-id="931fe-142">Vanno impostati come **Risorsa incorporata**; in caso contrario `ResourceStringLocalizer` non è in grado di trovare tali risorse.</span><span class="sxs-lookup"><span data-stu-id="931fe-142">They should be **Embedded Resource**, otherwise the `ResourceStringLocalizer` is not able to find these resources.</span></span>