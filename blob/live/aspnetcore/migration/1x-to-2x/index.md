---
title: Migrazione da ASP.NET Core 1.x a 2.0
author: scottaddie
description: "In questo articolo vengono illustrati i prerequisiti e i passaggi più comuni per la migrazione di un progetto di ASP.NET Core 1.x su ASP.NET 2.0 Core."
keywords: ASP.NET Core, eseguire la migrazione
ms.author: scaddie
manager: wpickett
ms.date: 10/03/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: migration/1x-to-2x/index
ms.openlocfilehash: 12734504953f2942458c3bfe1fe146f48d8f24ff
ms.sourcegitcommit: 8f42ab93402c1b8044815e1e48d0bb84c81f8b59
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/29/2017
---
# <a name="migrating-from-aspnet-core-1x-to-aspnet-core-20"></a><span data-ttu-id="f141c-104">Migrazione da ASP.NET Core 1.x a 2.0</span><span class="sxs-lookup"><span data-stu-id="f141c-104">Migrating from ASP.NET Core 1.x to ASP.NET Core 2.0</span></span>

<span data-ttu-id="f141c-105">Di [Scott Addie](https://github.com/scottaddie)</span><span class="sxs-lookup"><span data-stu-id="f141c-105">By [Scott Addie](https://github.com/scottaddie)</span></span>

<span data-ttu-id="f141c-106">In questo articolo si procederà con l'aggiornamento di un progetto ASP.NET Core 1.x esistente ad ASP.NET 2.0 Core.</span><span class="sxs-lookup"><span data-stu-id="f141c-106">In this article, we'll walk you through updating an existing ASP.NET Core 1.x project to ASP.NET Core 2.0.</span></span> <span data-ttu-id="f141c-107">La migrazione dell'applicazione ad ASP.NET 2.0 Core consente di sfruttare le [molte nuove caratteristiche e i miglioramenti delle prestazioni](https://docs.microsoft.com/aspnet/core/aspnetcore-2.0).</span><span class="sxs-lookup"><span data-stu-id="f141c-107">Migrating your application to ASP.NET Core 2.0 enables you to take advantage of [many new features and performance improvements](https://docs.microsoft.com/aspnet/core/aspnetcore-2.0).</span></span> 

<span data-ttu-id="f141c-108">Le applicazioni esistenti di ASP.NET Core 1.x si basano su modelli di progetto specifico della versione.</span><span class="sxs-lookup"><span data-stu-id="f141c-108">Existing ASP.NET Core 1.x applications are based off of version-specific project templates.</span></span> <span data-ttu-id="f141c-109">Quando il framework di ASP.NET Core si evolve, i modelli del progetto fanno la stessa cosa, così come il codice di avvio in essi contenuti.</span><span class="sxs-lookup"><span data-stu-id="f141c-109">As the ASP.NET Core framework evolves, so do the project templates and the starter code contained within them.</span></span> <span data-ttu-id="f141c-110">Oltre ad aggiornare il framework di ASP.NET Core, è necessario aggiornare il codice dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="f141c-110">In addition to updating the ASP.NET Core framework, you need to update the code for your application.</span></span>

<a name="prerequisites"></a>

## <a name="prerequisites"></a><span data-ttu-id="f141c-111">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f141c-111">Prerequisites</span></span>
<span data-ttu-id="f141c-112">Vedere [Introduzione ad ASP.NET Core](xref:getting-started).</span><span class="sxs-lookup"><span data-stu-id="f141c-112">Please see [Getting Started with ASP.NET Core](xref:getting-started).</span></span>

<a name="tfm"></a>

## <a name="update-target-framework-moniker-tfm"></a><span data-ttu-id="f141c-113">Aggiornare Moniker della versione di .NET Framework di destinazione (TFM, Target Framework Moniker)</span><span class="sxs-lookup"><span data-stu-id="f141c-113">Update Target Framework Moniker (TFM)</span></span>
<span data-ttu-id="f141c-114">I progetti destinati a .NET Core devono usare il [TFM](/dotnet/standard/frameworks#referring-to-frameworks) di una versione successiva o uguale a .NET Core 2.0.</span><span class="sxs-lookup"><span data-stu-id="f141c-114">Projects targeting .NET Core should use the [TFM](/dotnet/standard/frameworks#referring-to-frameworks) of a version greater than or equal to .NET Core 2.0.</span></span> <span data-ttu-id="f141c-115">Cercare il nodo `<TargetFramework>` nel file *csproj*, e sostituire il testo interno con `netcoreapp2.0`:</span><span class="sxs-lookup"><span data-stu-id="f141c-115">Search for the `<TargetFramework>` node in the *.csproj* file, and replace its inner text with `netcoreapp2.0`:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=3)]

<span data-ttu-id="f141c-116">I progetti destinati a .NET Framework devono usare il TFM di una versione successiva o uguale a .NET Framework 4.6.1.</span><span class="sxs-lookup"><span data-stu-id="f141c-116">Projects targeting .NET Framework should use the TFM of a version greater than or equal to .NET Framework 4.6.1.</span></span> <span data-ttu-id="f141c-117">Cercare il nodo `<TargetFramework>` nel file *csproj*, e sostituire il testo interno con `net461`:</span><span class="sxs-lookup"><span data-stu-id="f141c-117">Search for the `<TargetFramework>` node in the *.csproj* file, and replace its inner text with `net461`:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=4)]

> [!NOTE]
> <span data-ttu-id="f141c-118">.NET core 2.0 offre una quantità di superficie maggiore rispetto a .NET Core 1.x.</span><span class="sxs-lookup"><span data-stu-id="f141c-118">.NET Core 2.0 offers a much larger surface area than .NET Core 1.x.</span></span> <span data-ttu-id="f141c-119">Se il destinatario è .NET Framework esclusivamente a causa della mancanza di API in .NET Core 1.x, avere .NET Core 2.0 come destinatario potrebbe funzionare.</span><span class="sxs-lookup"><span data-stu-id="f141c-119">If you're targeting .NET Framework solely because of missing APIs in .NET Core 1.x, targeting .NET Core 2.0 is likely to work.</span></span>

<a name="global-json"></a>

## <a name="update-net-core-sdk-version-in-globaljson"></a><span data-ttu-id="f141c-120">Aggiornare la versione di .NET Core SDK in global.json</span><span class="sxs-lookup"><span data-stu-id="f141c-120">Update .NET Core SDK version in global.json</span></span>
<span data-ttu-id="f141c-121">Se la soluzione si basa su un file [*global.json*](https://docs.microsoft.com/dotnet/core/tools/global-json) per avere come destinatario una specifica versione di .NET Core SDK, aggiornare la relativa proprietà `version` per usare la versione 2.0 installata nel computer:</span><span class="sxs-lookup"><span data-stu-id="f141c-121">If your solution relies upon a [*global.json*](https://docs.microsoft.com/dotnet/core/tools/global-json) file to target a specific .NET Core SDK version, update its `version` property to use the 2.0 version installed on your machine:</span></span>

[!code-json[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/global.json?highlight=3)]

<a name="package-reference"></a>

## <a name="update-package-references"></a><span data-ttu-id="f141c-122">Aggiornare i riferimenti del pacchetto</span><span class="sxs-lookup"><span data-stu-id="f141c-122">Update package references</span></span>
<span data-ttu-id="f141c-123">Il file *csproj* in un progetto di 1.x elenca ogni pacchetto NuGet usato dal progetto.</span><span class="sxs-lookup"><span data-stu-id="f141c-123">The *.csproj* file in a 1.x project lists each NuGet package used by the project.</span></span>

<span data-ttu-id="f141c-124">In un progetto ASP.NET Core 2.0 destinato a .NET Core 2.0, un singolo riferimento del [metapacchetto](xref:fundamentals/metapackage) nel file *csproj* sostituisce la raccolta di pacchetti:</span><span class="sxs-lookup"><span data-stu-id="f141c-124">In an ASP.NET Core 2.0 project targeting .NET Core 2.0, a single [metapackage](xref:fundamentals/metapackage) reference in the *.csproj* file replaces the collection of packages:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=8-10)]

<span data-ttu-id="f141c-125">Tutte le funzionalità di ASP.NET Core 2.0 e di Entity Framework Core 2.0 sono incluse nel metapacchetto.</span><span class="sxs-lookup"><span data-stu-id="f141c-125">All the features of ASP.NET Core 2.0 and Entity Framework Core 2.0 are included in the metapackage.</span></span>

<span data-ttu-id="f141c-126">I progetti di ASP.NET Core 2.0 con destinazione .NET Framework devono fare riferimento a singoli pacchetti NuGet.</span><span class="sxs-lookup"><span data-stu-id="f141c-126">ASP.NET Core 2.0 projects targeting .NET Framework should continue to reference individual NuGet packages.</span></span> <span data-ttu-id="f141c-127">Aggiornamento dell'attributo `Version` di ogni `<PackageReference />` nodo a 2.0.0.</span><span class="sxs-lookup"><span data-stu-id="f141c-127">Update the `Version` attribute of each `<PackageReference />` node to 2.0.0.</span></span>

<span data-ttu-id="f141c-128">Ad esempio, ecco l'elenco dei `<PackageReference />` nodi usati in un progetto ASP.NET Core 2.0 tipico destinato a .NET Framework:</span><span class="sxs-lookup"><span data-stu-id="f141c-128">For example, here's the list of `<PackageReference />` nodes used in a typical ASP.NET Core 2.0 project targeting .NET Framework:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=9-22)]

<a name="dot-net-cli-tool-reference"></a>

## <a name="update-net-core-cli-tools"></a><span data-ttu-id="f141c-129">Strumenti dell'interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="f141c-129">Update .NET Core CLI tools</span></span>
<span data-ttu-id="f141c-130">Nel file *.csproj* aggiornare l'attributo `Version` di ogni nodo `<DotNetCliToolReference />` a 2.0.0.</span><span class="sxs-lookup"><span data-stu-id="f141c-130">In the *.csproj* file, update the `Version` attribute of each `<DotNetCliToolReference />` node to 2.0.0.</span></span>

<span data-ttu-id="f141c-131">Ad esempio, ecco l'elenco degli strumenti dell'interfaccia della riga di comando usati in un progetto ASP.NET Core 2.0 tipico destinato a .NET Core 2.0:</span><span class="sxs-lookup"><span data-stu-id="f141c-131">For example, here's the list of CLI tools used in a typical ASP.NET Core 2.0 project targeting .NET Core 2.0:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=12-16)]

<a name="package-target-fallback"></a>

## <a name="rename-package-target-fallback-property"></a><span data-ttu-id="f141c-132">Rinominare la proprietà di Fallback di destinazione del pacchetto</span><span class="sxs-lookup"><span data-stu-id="f141c-132">Rename Package Target Fallback property</span></span>
<span data-ttu-id="f141c-133">Il file *csproj* di un progetto 1.x ha usato un nodo `PackageTargetFallback` e una variabile:</span><span class="sxs-lookup"><span data-stu-id="f141c-133">The *.csproj* file of a 1.x project used a `PackageTargetFallback` node and variable:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App.csproj?range=5)]

<span data-ttu-id="f141c-134">Rinominare il nodo e la variabile a `AssetTargetFallback`:</span><span class="sxs-lookup"><span data-stu-id="f141c-134">Rename both the node and variable to `AssetTargetFallback`:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App.csproj?range=4)]

<a name="program-cs"></a>

## <a name="update-main-method-in-programcs"></a><span data-ttu-id="f141c-135">Aggiornare il metodo Main in Program.cs</span><span class="sxs-lookup"><span data-stu-id="f141c-135">Update Main method in Program.cs</span></span>
<span data-ttu-id="f141c-136">Nei progetti di 1.x, il metodo `Main` di *Program.cs* era simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="f141c-136">In 1.x projects, the `Main` method of *Program.cs* looked like this:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Program.cs?name=snippet_ProgramCs&highlight=8-19)]

<span data-ttu-id="f141c-137">Nei progetti 2.0, il metodo `Main` di *Program.cs* è stato semplificato:</span><span class="sxs-lookup"><span data-stu-id="f141c-137">In 2.0 projects, the `Main` method of *Program.cs* has been simplified:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Program.cs?highlight=8-11)]

<span data-ttu-id="f141c-138">L'adozione di questo nuovo modello 2.0 è altamente consigliabile ed è necessaria affinché le funzionalità di prodotto come le [Migrazioni Entity Framework (EF) Core](xref:data/ef-mvc/migrations) funzionino.</span><span class="sxs-lookup"><span data-stu-id="f141c-138">The adoption of this new 2.0 pattern is highly recommended and is required for product features like [Entity Framework (EF) Core Migrations](xref:data/ef-mvc/migrations) to work.</span></span> <span data-ttu-id="f141c-139">Ad esempio, l'esecuzione di `Update-Database` dalla finestra della Console di Gestione pacchetti o di `dotnet ef database update` dalla riga di comando (nei progetti convertiti in ASP.NET Core 2.0) genera l'errore seguente:</span><span class="sxs-lookup"><span data-stu-id="f141c-139">For example, running `Update-Database` from the Package Manager Console window or `dotnet ef database update` from the command line (on projects converted to ASP.NET Core 2.0) generates the following error:</span></span>

```
Unable to create an object of type '<Context>'. Add an implementation of 'IDesignTimeDbContextFactory<Context>' to the project, or see https://go.microsoft.com/fwlink/?linkid=851728 for additional patterns supported at design time.
```

<a name="add-modify-configuration"></a>

## <a name="add-configuration-providers"></a><span data-ttu-id="f141c-140">Aggiungere provider di configurazione</span><span class="sxs-lookup"><span data-stu-id="f141c-140">Add configuration providers</span></span>
<span data-ttu-id="f141c-141">Nei progetti di 1. x viene usato il costruttore `Startup` per aggiungere i provider di configurazione a un'app.</span><span class="sxs-lookup"><span data-stu-id="f141c-141">In 1.x projects, adding configuration providers to an app was accomplished via the `Startup` constructor.</span></span> <span data-ttu-id="f141c-142">La procedura implica la creazione di un'istanza di `ConfigurationBuilder`, il caricamento di provider applicabili (variabili di ambiente, impostazioni dell'app e così via) e l'inizializzazione di un membro di `IConfigurationRoot`.</span><span class="sxs-lookup"><span data-stu-id="f141c-142">The steps involved creating an instance of `ConfigurationBuilder`, loading applicable providers (environment variables, app settings, etc.), and initializing a member of `IConfigurationRoot`.</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Startup.cs?name=snippet_1xStartup)]

<span data-ttu-id="f141c-143">Nell'esempio precedente il membro `Configuration` viene caricato con le impostazioni di configurazione da *appsettings.json* e da un qualsiasi file *appsettings.\<NomeAmbiente\>.json* corrispondente alla proprietà `IHostingEnvironment.EnvironmentName`.</span><span class="sxs-lookup"><span data-stu-id="f141c-143">The preceding example loads the `Configuration` member with configuration settings from *appsettings.json* as well as any *appsettings.\<EnvironmentName\>.json* file matching the `IHostingEnvironment.EnvironmentName` property.</span></span> <span data-ttu-id="f141c-144">Questi file si trovano nello stesso percorso di *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="f141c-144">The location of these files is at the same path as *Startup.cs*.</span></span>

<span data-ttu-id="f141c-145">Nei progetti 2.0 il codice di configurazione boilerplate relativo ai progetti 1. x viene eseguito in background.</span><span class="sxs-lookup"><span data-stu-id="f141c-145">In 2.0 projects, the boilerplate configuration code inherent to 1.x projects runs behind-the-scenes.</span></span> <span data-ttu-id="f141c-146">Le variabili di ambiente e le impostazioni dell'app vengono ad esempio caricate all'avvio.</span><span class="sxs-lookup"><span data-stu-id="f141c-146">For example, environment variables and app settings are loaded at startup.</span></span> <span data-ttu-id="f141c-147">Il codice *Startup.cs* equivalente viene ridotto all'inizializzazione `IConfiguration` con l'istanza inserita:</span><span class="sxs-lookup"><span data-stu-id="f141c-147">The equivalent *Startup.cs* code is reduced to `IConfiguration` initialization with the injected instance:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/Startup.cs?name=snippet_2xStartup)]

<span data-ttu-id="f141c-148">Per rimuovere i provider predefiniti aggiunti da `WebHostBuilder.CreateDefaultBuilder`, richiamare il metodo `Clear` nella proprietà `IConfigurationBuilder.Sources` all'interno di `ConfigureAppConfiguration`.</span><span class="sxs-lookup"><span data-stu-id="f141c-148">To remove the default providers added by `WebHostBuilder.CreateDefaultBuilder`, invoke the `Clear` method on the `IConfigurationBuilder.Sources` property inside of `ConfigureAppConfiguration`.</span></span> <span data-ttu-id="f141c-149">Per aggiungere di nuovo i provider, usare il metodo `ConfigureAppConfiguration`in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="f141c-149">To add providers back, utilize the `ConfigureAppConfiguration` method in *Program.cs*:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/Program.cs?name=snippet_ProgramMainConfigProviders&highlight=9-14)]

<span data-ttu-id="f141c-150">La configurazione usata dal metodo `CreateDefaultBuilder` nel frammento di codice precedente può essere visualizzata [qui](https://github.com/aspnet/MetaPackages/blob/rel/2.0.0/src/Microsoft.AspNetCore/WebHost.cs#L152).</span><span class="sxs-lookup"><span data-stu-id="f141c-150">The configuration used by the `CreateDefaultBuilder` method in the preceding code snippet can be seen [here](https://github.com/aspnet/MetaPackages/blob/rel/2.0.0/src/Microsoft.AspNetCore/WebHost.cs#L152).</span></span>

<span data-ttu-id="f141c-151">Per altre informazioni, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="f141c-151">For more information, see [Configuration in ASP.NET Core](xref:fundamentals/configuration/index).</span></span>

<a name="db-init-code"></a>

## <a name="move-database-initialization-code"></a><span data-ttu-id="f141c-152">Spostare il codice di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="f141c-152">Move database initialization code</span></span>
<span data-ttu-id="f141c-153">Nei progetti 1.x che usano EF Core 1.x un comando come ad esempio `dotnet ef migrations add` esegue le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="f141c-153">In 1.x projects using EF Core 1.x, a command such as `dotnet ef migrations add` does the following:</span></span>
1. <span data-ttu-id="f141c-154">Crea un'istanza dell'istanza `Startup`</span><span class="sxs-lookup"><span data-stu-id="f141c-154">Instantiates a `Startup` instance</span></span>
2. <span data-ttu-id="f141c-155">Richiama il metodo `ConfigureServices` per registrare tutti i servizi con inserimento di dipendenze, inclusi i tipi `DbContext`</span><span class="sxs-lookup"><span data-stu-id="f141c-155">Invokes the `ConfigureServices` method to register all services with dependency injection (including `DbContext` types)</span></span>
3. <span data-ttu-id="f141c-156">Esegue le attività necessarie</span><span class="sxs-lookup"><span data-stu-id="f141c-156">Performs its requisite tasks</span></span>

<span data-ttu-id="f141c-157">Nei progetti 2.0 che usano EF Core 2.0 viene richiamato `Program.BuildWebHost` per ottenere i servizi delle applicazioni.</span><span class="sxs-lookup"><span data-stu-id="f141c-157">In 2.0 projects using EF Core 2.0, `Program.BuildWebHost` is invoked to obtain the application services.</span></span> <span data-ttu-id="f141c-158">A differenza della versione 1.x, questo ha l'effetto collaterale di richiamare `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="f141c-158">Unlike 1.x, this has the additional side effect of invoking `Startup.Configure`.</span></span> <span data-ttu-id="f141c-159">Se l'app 1.x ha richiamato il codice di inizializzazione del database nel metodo `Configure`, possono verificarsi problemi imprevisti.</span><span class="sxs-lookup"><span data-stu-id="f141c-159">If your 1.x app invoked database initialization code in its `Configure` method, unexpected problems can occur.</span></span> <span data-ttu-id="f141c-160">Ad esempio, se il database non esiste ancora, il codice di seeding viene eseguito prima dell'esecuzione del comando delle migrazioni di EF Core.</span><span class="sxs-lookup"><span data-stu-id="f141c-160">For example, if the database doesn't yet exist, the seeding code runs before the EF Core Migrations command execution.</span></span> <span data-ttu-id="f141c-161">Questo problema causa l'esito negativo del comando `dotnet ef migrations list` se il database non esiste ancora.</span><span class="sxs-lookup"><span data-stu-id="f141c-161">This problem causes a `dotnet ef migrations list` command to fail if the database doesn't yet exist.</span></span>

<span data-ttu-id="f141c-162">Si consideri il codice di inizializzazione del seed 1.x seguente nel metodo `Configure` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="f141c-162">Consider the following 1.x seed initialization code in the `Configure` method of *Startup.cs*:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Startup.cs?name=snippet_ConfigureSeedData&highlight=8)]

<span data-ttu-id="f141c-163">Nei progetti 2.0 spostare la chiamata `SeedData.Initialize` al metodo `Main` di *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="f141c-163">In 2.0 projects, move the `SeedData.Initialize` call to the `Main` method of *Program.cs*:</span></span>

[!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Program2.cs?name=snippet_Main2Code&highlight=10)]

<span data-ttu-id="f141c-164">A partire dalla versione 2.0, non è una buona prassi eseguire alcuna operazione in `BuildWebHost` ad eccezione della compilazione e della configurazione dell'host Web.</span><span class="sxs-lookup"><span data-stu-id="f141c-164">As of 2.0, it's bad practice to do anything in `BuildWebHost` except build and configure the web host.</span></span> <span data-ttu-id="f141c-165">Tutto ciò che riguarda l'esecuzione dell'applicazione deve essere gestito all'esterno di `BuildWebHost`, in genere nel metodo `Main` di *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="f141c-165">Anything that is about running the application should be handled outside of `BuildWebHost` &mdash; typically in the `Main` method of *Program.cs*.</span></span>

<a name="view-compilation"></a>

## <a name="review-your-razor-view-compilation-setting"></a><span data-ttu-id="f141c-166">Revisionare le impostazioni di compilazione della vista Razor</span><span class="sxs-lookup"><span data-stu-id="f141c-166">Review your Razor View Compilation setting</span></span>
<span data-ttu-id="f141c-167">Tempi di avvio dell'applicazione più rapidi e aggregazioni pubblicate più piccole sono di importanza fondamentale per l'utente.</span><span class="sxs-lookup"><span data-stu-id="f141c-167">Faster application startup time and smaller published bundles are of utmost importance to you.</span></span> <span data-ttu-id="f141c-168">Per questi motivi, [la compilazione della vista Razor](xref:mvc/views/view-compilation) è abilitata per impostazione predefinita in ASP.NET Core 2.0.</span><span class="sxs-lookup"><span data-stu-id="f141c-168">For these reasons, [Razor view compilation](xref:mvc/views/view-compilation) is enabled by default in ASP.NET Core 2.0.</span></span>

<span data-ttu-id="f141c-169">L'impostazione della proprietà `MvcRazorCompileOnPublish` su true non è più necessaria.</span><span class="sxs-lookup"><span data-stu-id="f141c-169">Setting the `MvcRazorCompileOnPublish` property to true is no longer required.</span></span> <span data-ttu-id="f141c-170">A meno che non si stia disattivando la compilazione della vista, la proprietà può essere rimossa dal file *csproj*.</span><span class="sxs-lookup"><span data-stu-id="f141c-170">Unless you're disabling view compilation, the property may be removed from the *.csproj* file.</span></span>

<span data-ttu-id="f141c-171">Quando la destinazione è .NET Framework, è necessario fare comunque riferimento in modo esplicito al pacchetto NuGet [Microsoft.AspNetCore.Mvc.Razor.ViewCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.ViewCompilation) nel file *csproj*:</span><span class="sxs-lookup"><span data-stu-id="f141c-171">When targeting .NET Framework, you still need to explicitly reference the [Microsoft.AspNetCore.Mvc.Razor.ViewCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.ViewCompilation) NuGet package in your *.csproj* file:</span></span>

[!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App/AspNetCoreDotNetFx2.0App.csproj?range=15)]

<a name="app-insights"></a>

## <a name="rely-on-application-insights-light-up-features"></a><span data-ttu-id="f141c-172">Basarsi sulle funzionalità di Application Insights "Light-Up"</span><span class="sxs-lookup"><span data-stu-id="f141c-172">Rely on Application Insights "Light-Up" features</span></span>
<span data-ttu-id="f141c-173">Un semplice programma di installazione di strumentazione delle prestazioni dell'applicazione è importante.</span><span class="sxs-lookup"><span data-stu-id="f141c-173">Effortless setup of application performance instrumentation is important.</span></span> <span data-ttu-id="f141c-174">È ora possibile basarsi sulle nuove funzionalità "light-up" di [Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview) disponibili negli strumenti di Visual Studio 2017.</span><span class="sxs-lookup"><span data-stu-id="f141c-174">You can now rely on the new [Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview) "light-up" features available in the Visual Studio 2017 tooling.</span></span>

<span data-ttu-id="f141c-175">Per impostazione predefinita, i progetti ASP.NET Core 1.1 creati in Visual Studio 2017 hanno aggiunto Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f141c-175">ASP.NET Core 1.1 projects created in Visual Studio 2017 added Application Insights by default.</span></span> <span data-ttu-id="f141c-176">Se non si usa Application Insights SDK direttamente, di fuori di *Program.cs* e *Startup.cs*, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="f141c-176">If you're not using the Application Insights SDK directly, outside of *Program.cs* and *Startup.cs*, follow these steps:</span></span>

1. <span data-ttu-id="f141c-177">Se la destinazione è .NET Core, rimuovere il nodo `<PackageReference />` seguente dal file *.csproj*:</span><span class="sxs-lookup"><span data-stu-id="f141c-177">If targeting .NET Core, remove the following `<PackageReference />` node from the *.csproj* file:</span></span>
    
    [!code-xml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App.csproj?range=10)]

2. <span data-ttu-id="f141c-178">Se la destinazione è .NET Core, rimuovere la chiamata del metodo di estensione `UseApplicationInsights` da *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="f141c-178">If targeting .NET Core, remove the `UseApplicationInsights` extension method invocation from *Program.cs*:</span></span>

    [!code-csharp[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Program.cs?name=snippet_ProgramCsMain&highlight=8)]

3. <span data-ttu-id="f141c-179">Rimuovere la chiamata di API sul lato client di Application Insights da *_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="f141c-179">Remove the Application Insights client-side API call from *_Layout.cshtml*.</span></span> <span data-ttu-id="f141c-180">Comprende le due righe di codice seguenti:</span><span class="sxs-lookup"><span data-stu-id="f141c-180">It comprises the following two lines of code:</span></span>

    [!code-cshtml[Main](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Views/Shared/_Layout.cshtml?range=1,19&dedent=4)]

<span data-ttu-id="f141c-181">Se si usa Application Insights SDK direttamente, continuare a farlo.</span><span class="sxs-lookup"><span data-stu-id="f141c-181">If you are using the Application Insights SDK directly, continue to do so.</span></span> <span data-ttu-id="f141c-182">Il [metapacchetto](xref:fundamentals/metapackage) 2.0 comprende la versione più recente di Application Insights, pertanto viene visualizzato un errore di downgrade del pacchetto se si fa riferimento a una versione precedente.</span><span class="sxs-lookup"><span data-stu-id="f141c-182">The 2.0 [metapackage](xref:fundamentals/metapackage) includes the latest version of Application Insights, so a package downgrade error appears if you're referencing an older version.</span></span>

<a name="auth-and-identity"></a>

## <a name="adopt-authentication--identity-improvements"></a><span data-ttu-id="f141c-183">Adottare i miglioramenti di Autenticazione/Identità</span><span class="sxs-lookup"><span data-stu-id="f141c-183">Adopt Authentication / Identity Improvements</span></span>
<span data-ttu-id="f141c-184">ASP.NET Core 2.0 ha un nuovo modello di autenticazione e un numero di modifiche significative per ASP.NET Identity Core.</span><span class="sxs-lookup"><span data-stu-id="f141c-184">ASP.NET Core 2.0 has a new authentication model and a number of significant changes to ASP.NET Core Identity.</span></span> <span data-ttu-id="f141c-185">Se il progetto è stato creato con Account utente abilitati o se è stata aggiunta manualmente l'autenticazione o l'identità, vedere [Migrazione di autenticazione e identità ad ASP.NET 2.0 Core](xref:migration/1x-to-2x/identity-2x).</span><span class="sxs-lookup"><span data-stu-id="f141c-185">If you created your project with Individual User Accounts enabled, or if you have manually added authentication or Identity, see [Migrating Authentication and Identity to ASP.NET Core 2.0](xref:migration/1x-to-2x/identity-2x).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="f141c-186">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="f141c-186">Additional Resources</span></span>
- [<span data-ttu-id="f141c-187">Modifiche importanti apportate in ASP.NET Core 2.0</span><span class="sxs-lookup"><span data-stu-id="f141c-187">Breaking Changes in ASP.NET Core 2.0</span></span>](https://github.com/aspnet/announcements/issues?page=1&q=is%3Aissue+is%3Aopen+label%3A2.0.0+label%3A%22Breaking+change%22&utf8=%E2%9C%93)