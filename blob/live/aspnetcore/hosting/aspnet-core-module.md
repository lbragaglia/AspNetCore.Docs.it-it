---
title: Riferimento di configurazione di ASP.NET modulo Core
author: guardrex
description: Come configurare il modulo di base di ASP.NET per ospitare applicazioni ASP.NET Core.
keywords: ASP.NET Core, ancm, modulo di base, iis, la registrazione di stdout, variabile di ambiente, var env, subapplication, subapp, appoffline, app_offline, 502, dello schema
ms.author: riande
manager: wpickett
ms.date: 03/07/2017
ms.topic: article
ms.assetid: 5de0c8f7-50ce-4e2c-b3d4-a1bd9fdfcff5
ms.technology: aspnet
ms.prod: asp.net-core
uid: hosting/aspnet-core-module
ms.openlocfilehash: 277e63a5663aca622e8252d6c6be1671e57cbf68
ms.sourcegitcommit: 44a62f59d4db39d685c4487a0345a486be18d7c7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 12/21/2017
---
# <a name="aspnet-core-module-configuration-reference"></a><span data-ttu-id="2af8a-104">Riferimento di configurazione di ASP.NET modulo Core</span><span class="sxs-lookup"><span data-stu-id="2af8a-104">ASP.NET Core Module configuration reference</span></span>

<span data-ttu-id="2af8a-105">Da [Luke Latham](https://github.com/guardrex), [Rick Anderson](https://twitter.com/RickAndMSFT), e [Sourabh Shirhatti](https://twitter.com/sshirhatti)</span><span class="sxs-lookup"><span data-stu-id="2af8a-105">By [Luke Latham](https://github.com/guardrex), [Rick Anderson](https://twitter.com/RickAndMSFT), and [Sourabh Shirhatti](https://twitter.com/sshirhatti)</span></span>

<span data-ttu-id="2af8a-106">Questo documento vengono fornite informazioni dettagliate su come configurare il modulo di base di ASP.NET per ospitare applicazioni ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2af8a-106">This document provides details on how to configure the ASP.NET Core Module for hosting ASP.NET Core applications.</span></span> <span data-ttu-id="2af8a-107">Per informazioni introduttive per il modulo di base di ASP.NET e le istruzioni di installazione, vedere il [Panoramica del modulo di ASP.NET Core](xref:fundamentals/servers/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="2af8a-107">For an introduction to the ASP.NET Core Module and installation instructions, see the [ASP.NET Core Module overview](xref:fundamentals/servers/aspnet-core-module).</span></span>

## <a name="configuration-via-webconfig"></a><span data-ttu-id="2af8a-108">Configurazione tramite Web. config</span><span class="sxs-lookup"><span data-stu-id="2af8a-108">Configuration via web.config</span></span>

<span data-ttu-id="2af8a-109">Il modulo di base di ASP.NET viene configurato tramite un sito o applicazione *Web. config* file e ha il proprio `aspNetCore` sezione di configurazione all'interno di `system.webServer`.</span><span class="sxs-lookup"><span data-stu-id="2af8a-109">The ASP.NET Core Module is configured via a site or application *web.config* file and has its own `aspNetCore` configuration section within `system.webServer`.</span></span> <span data-ttu-id="2af8a-110">Di seguito è riportato un esempio *Web. config* file che il `Microsoft.NET.Sdk.Web` SDK fornirà quando il progetto viene pubblicato per un [framework dipendente distribuzione](https://docs.microsoft.com/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) con segnaposto per il `processPath` e `arguments`:</span><span class="sxs-lookup"><span data-stu-id="2af8a-110">Here's an example *web.config* file that the `Microsoft.NET.Sdk.Web` SDK will provide when the project is published for a [framework-dependent deployment](https://docs.microsoft.com/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) with placeholders for the `processPath` and `arguments`:</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="%LAUNCHER_PATH%" 
        arguments="%LAUNCHER_ARGS%" 
        stdoutLogEnabled="false" 
        stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

<span data-ttu-id="2af8a-111">Il *Web. config* esempio riportato di seguito è per un [distribuzione indipendente](https://docs.microsoft.com/dotnet/articles/core/deploying/#self-contained-deployments-scd) per il [Azure App Service](https://azure.microsoft.com/services/app-service/).</span><span class="sxs-lookup"><span data-stu-id="2af8a-111">The *web.config* example below is for a [self-contained deployment](https://docs.microsoft.com/dotnet/articles/core/deploying/#self-contained-deployments-scd) to the [Azure App Service](https://azure.microsoft.com/services/app-service/).</span></span> <span data-ttu-id="2af8a-112">Per ulteriori informazioni, vedere [la pubblicazione in IIS](xref:publishing/iis).</span><span class="sxs-lookup"><span data-stu-id="2af8a-112">For more information, see [Publishing to IIS](xref:publishing/iis).</span></span> <span data-ttu-id="2af8a-113">Vedere [configurazione delle applicazioni secondario](xref:publishing/iis#configuration-of-sub-applications) per una nota importante relativi alla configurazione di *Web. config* file secondari di applicazioni.</span><span class="sxs-lookup"><span data-stu-id="2af8a-113">See [Configuration of sub-applications](xref:publishing/iis#configuration-of-sub-applications) for an important note pertaining to the configuration of *web.config* files in sub-applications.</span></span>

```xml
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath=".\MyApp.exe" 
        stdoutLogEnabled="false" 
        stdoutLogFile="\\?\%home%\LogFiles\stdout" />
  </system.webServer>
</configuration>
```

### <a name="attributes-of-the-aspnetcore-element"></a><span data-ttu-id="2af8a-114">Attributi dell'elemento aspNetCore</span><span class="sxs-lookup"><span data-stu-id="2af8a-114">Attributes of the aspNetCore element</span></span>

| <span data-ttu-id="2af8a-115">Attributo</span><span class="sxs-lookup"><span data-stu-id="2af8a-115">Attribute</span></span> | <span data-ttu-id="2af8a-116">Descrizione</span><span class="sxs-lookup"><span data-stu-id="2af8a-116">Description</span></span> |
| --- | --- |
| <span data-ttu-id="2af8a-117">processPath</span><span class="sxs-lookup"><span data-stu-id="2af8a-117">processPath</span></span> | <p><span data-ttu-id="2af8a-118">Attributo stringa obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="2af8a-118">Required string attribute.</span></span></p><p><span data-ttu-id="2af8a-119">Percorso dell'eseguibile da avvia un processo in ascolto delle richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="2af8a-119">Path to the executable that will launch a process listening for HTTP requests.</span></span> <span data-ttu-id="2af8a-120">I percorsi relativi sono supportati.</span><span class="sxs-lookup"><span data-stu-id="2af8a-120">Relative paths are supported.</span></span> <span data-ttu-id="2af8a-121">Se il percorso inizia con '.', il percorso viene considerato come relativo alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="2af8a-121">If the path begins with '.', the path is considered to be relative to the site root.</span></span></p><p><span data-ttu-id="2af8a-122">Non è previsto alcun valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="2af8a-122">There is no default value.</span></span></p> |
| <span data-ttu-id="2af8a-123">arguments</span><span class="sxs-lookup"><span data-stu-id="2af8a-123">arguments</span></span> | <p><span data-ttu-id="2af8a-124">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-124">Optional string attribute.</span></span></p><p><span data-ttu-id="2af8a-125">Argomenti per l'eseguibile specificato **processPath**.</span><span class="sxs-lookup"><span data-stu-id="2af8a-125">Arguments to the executable specified in **processPath**.</span></span></p><p><span data-ttu-id="2af8a-126">Il valore predefinito è una stringa vuota.</span><span class="sxs-lookup"><span data-stu-id="2af8a-126">The default value is an empty string.</span></span></p> |
| <span data-ttu-id="2af8a-127">startupTimeLimit</span><span class="sxs-lookup"><span data-stu-id="2af8a-127">startupTimeLimit</span></span> | <p><span data-ttu-id="2af8a-128">Attributo intero facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-128">Optional integer attribute.</span></span></p><p><span data-ttu-id="2af8a-129">Durata in secondi di attesa per l'eseguibile da avviare un processo in ascolto sulla porta il modulo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-129">Duration in seconds that the module will wait for the executable to start a process listening on the port.</span></span> <span data-ttu-id="2af8a-130">Se viene superato questo limite di tempo, il modulo verrà terminare il processo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-130">If this time limit is exceeded, the module will kill the process.</span></span> <span data-ttu-id="2af8a-131">Il modulo tenterà di avviare nuovamente il processo quando riceve una richiesta di nuovo e continuerà a tentare di riavviare il processo successivo delle richieste in ingresso, a meno che non viene avviato l'applicazione **rapidFailsPerMinute** numero di volte in cui nell'ultimo minuto in sequenza.</span><span class="sxs-lookup"><span data-stu-id="2af8a-131">The module will attempt to launch the process again when it receives a new request and will continue to attempt to restart the process on subsequent incoming requests unless the application fails to start **rapidFailsPerMinute** number of times in the last rolling minute.</span></span></p><p><span data-ttu-id="2af8a-132">Il valore predefinito è 120.</span><span class="sxs-lookup"><span data-stu-id="2af8a-132">The default value is 120.</span></span></p> |
| <span data-ttu-id="2af8a-133">shutdownTimeLimit</span><span class="sxs-lookup"><span data-stu-id="2af8a-133">shutdownTimeLimit</span></span> | <p><span data-ttu-id="2af8a-134">Attributo intero facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-134">Optional integer attribute.</span></span></p><p><span data-ttu-id="2af8a-135">Durata in secondi per cui il modulo di attesa per il file eseguibile per arrestare normalmente quando il *app_offline.htm* file è stato rilevato.</span><span class="sxs-lookup"><span data-stu-id="2af8a-135">Duration in seconds for which the module will wait for the executable to gracefully shutdown when the *app_offline.htm* file is detected.</span></span></p><p><span data-ttu-id="2af8a-136">Il valore predefinito è 10.</span><span class="sxs-lookup"><span data-stu-id="2af8a-136">The default value is 10.</span></span></p> |
| <span data-ttu-id="2af8a-137">rapidFailsPerMinute</span><span class="sxs-lookup"><span data-stu-id="2af8a-137">rapidFailsPerMinute</span></span> | <p><span data-ttu-id="2af8a-138">Attributo intero facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-138">Optional integer attribute.</span></span></p><p><span data-ttu-id="2af8a-139">Specifica il numero di volte specificato dal processo in **processPath** può arrestarsi al minuto.</span><span class="sxs-lookup"><span data-stu-id="2af8a-139">Specifies the number of times the process specified in **processPath** is allowed to crash per minute.</span></span> <span data-ttu-id="2af8a-140">Se questo limite viene superato, il modulo smetterà di avviare il processo per la parte restante del minuto.</span><span class="sxs-lookup"><span data-stu-id="2af8a-140">If this limit is exceeded, the module will stop launching the process for the remainder of the minute.</span></span></p><p><span data-ttu-id="2af8a-141">Il valore predefinito è 10.</span><span class="sxs-lookup"><span data-stu-id="2af8a-141">The default value is 10.</span></span></p> |
| <span data-ttu-id="2af8a-142">requestTimeout</span><span class="sxs-lookup"><span data-stu-id="2af8a-142">requestTimeout</span></span> | <p><span data-ttu-id="2af8a-143">Attributo timespan facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-143">Optional timespan attribute.</span></span></p><p><span data-ttu-id="2af8a-144">Specifica la durata per cui il modulo di base di ASP.NET attenderà una risposta dal processo in ascolto su % ASPNETCORE_PORT %.</span><span class="sxs-lookup"><span data-stu-id="2af8a-144">Specifies the duration for which the ASP.NET Core Module will wait for a response from the process listening on %ASPNETCORE_PORT%.</span></span></p><p><span data-ttu-id="2af8a-145">Il valore predefinito è "00:02:00".</span><span class="sxs-lookup"><span data-stu-id="2af8a-145">The default value is "00:02:00".</span></span></p><p><span data-ttu-id="2af8a-146">Il `requestTimeout` deve essere specificato in minuti interi solo in caso contrario il valore predefinito è 2 minuti.</span><span class="sxs-lookup"><span data-stu-id="2af8a-146">The `requestTimeout` must be specified in whole minutes only, otherwise it defaults to 2 minutes.</span></span></p> |
| <span data-ttu-id="2af8a-147">stdoutLogEnabled</span><span class="sxs-lookup"><span data-stu-id="2af8a-147">stdoutLogEnabled</span></span> | <p><span data-ttu-id="2af8a-148">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-148">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="2af8a-149">Se true, **stdout** e **stderr** per il processo specificato **processPath** verranno reindirizzati al file specificato **stdoutLogFile**.</span><span class="sxs-lookup"><span data-stu-id="2af8a-149">If true, **stdout** and **stderr** for the process specified in **processPath** will be redirected to the file specified in **stdoutLogFile**.</span></span></p><p><span data-ttu-id="2af8a-150">Il valore predefinito è false.</span><span class="sxs-lookup"><span data-stu-id="2af8a-150">The default value is false.</span></span></p> |
| <span data-ttu-id="2af8a-151">stdoutLogFile</span><span class="sxs-lookup"><span data-stu-id="2af8a-151">stdoutLogFile</span></span> | <p><span data-ttu-id="2af8a-152">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-152">Optional string attribute.</span></span></p><p><span data-ttu-id="2af8a-153">Specifica il percorso relativo o assoluto per il quale **stdout** e **stderr** dal processo specificato in **processPath** verranno registrati.</span><span class="sxs-lookup"><span data-stu-id="2af8a-153">Specifies the relative or absolute file path for which **stdout** and **stderr** from the process specified in **processPath** will be logged.</span></span> <span data-ttu-id="2af8a-154">I percorsi relativi sono relativi alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="2af8a-154">Relative paths are relative to the root of the site.</span></span> <span data-ttu-id="2af8a-155">Qualsiasi percorso inizia con '.' sarà relativo alla radice del sito e tutti gli altri percorsi verranno considerati come percorsi assoluti.</span><span class="sxs-lookup"><span data-stu-id="2af8a-155">Any path starting with '.' will be relative to the site root and all other paths will be treated as absolute paths.</span></span> <span data-ttu-id="2af8a-156">Tutte le cartelle nel percorso specificate devono essere presente affinché il modulo creare il file di log.</span><span class="sxs-lookup"><span data-stu-id="2af8a-156">Any folders provided in the path must exist in order for the module to create the log file.</span></span> <span data-ttu-id="2af8a-157">L'ID del processo, timestamp (*yyyyMdhms*) e l'estensione di file (*log*) con un carattere di sottolineatura delimitatori vengono aggiunti all'ultimo segmento del **stdoutLogFile** fornito.</span><span class="sxs-lookup"><span data-stu-id="2af8a-157">The process ID, timestamp (*yyyyMdhms*), and file extension (*.log*) with underscore delimiters are added to the last segment of the **stdoutLogFile** provided.</span></span></p><p><span data-ttu-id="2af8a-158">Il valore predefinito è `aspnetcore-stdout`.</span><span class="sxs-lookup"><span data-stu-id="2af8a-158">The default value is `aspnetcore-stdout`.</span></span></p> |
| <span data-ttu-id="2af8a-159">forwardWindowsAuthToken</span><span class="sxs-lookup"><span data-stu-id="2af8a-159">forwardWindowsAuthToken</span></span> | <span data-ttu-id="2af8a-160">true o false.</span><span class="sxs-lookup"><span data-stu-id="2af8a-160">true or false.</span></span></p><p><span data-ttu-id="2af8a-161">Se true, il token verrà inoltrato al processo figlio in ascolto su % ASPNETCORE_PORT % come un'intestazione 'MS-ASPNETCORE-WINAUTHTOKEN' per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="2af8a-161">If true, the token will be forwarded to the child process listening on %ASPNETCORE_PORT% as a header 'MS-ASPNETCORE-WINAUTHTOKEN' per request.</span></span> <span data-ttu-id="2af8a-162">È responsabilità del processo di chiamare CloseHandle questo token per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="2af8a-162">It is the responsibility of that process to call CloseHandle on this token per request.</span></span></p><p><span data-ttu-id="2af8a-163">Il valore predefinito è true.</span><span class="sxs-lookup"><span data-stu-id="2af8a-163">The default value is true.</span></span></p> |
| <span data-ttu-id="2af8a-164">disableStartUpErrorPage</span><span class="sxs-lookup"><span data-stu-id="2af8a-164">disableStartUpErrorPage</span></span> | <span data-ttu-id="2af8a-165">true o false.</span><span class="sxs-lookup"><span data-stu-id="2af8a-165">true or false.</span></span></p><p><span data-ttu-id="2af8a-166">Se true, il **502.5 - errore del processo di** pagina verrà eliminata e la tabella codici di 502 stato nel *Web. config* avrà la precedenza.</span><span class="sxs-lookup"><span data-stu-id="2af8a-166">If true, the **502.5 - Process Failure** page will be suppressed, and the 502 status code page configured in your *web.config* will take precedence.</span></span></p><p><span data-ttu-id="2af8a-167">Il valore predefinito è false.</span><span class="sxs-lookup"><span data-stu-id="2af8a-167">The default value is false.</span></span></p> |

### <a name="setting-environment-variables"></a><span data-ttu-id="2af8a-168">Impostazione delle variabili di ambiente</span><span class="sxs-lookup"><span data-stu-id="2af8a-168">Setting environment variables</span></span>

<span data-ttu-id="2af8a-169">Il modulo di base di ASP.NET consente di specificare le variabili di ambiente per il processo specificato nella `processPath` attributo specificandoli in uno o più `environmentVariable` gli elementi figlio di un `environmentVariables` elemento della raccolta nel `aspNetCore` elemento.</span><span class="sxs-lookup"><span data-stu-id="2af8a-169">The ASP.NET Core Module allows you specify environment variables for the process specified in the `processPath` attribute by specifying them in one or more `environmentVariable` child elements of an `environmentVariables` collection element under the `aspNetCore` element.</span></span> <span data-ttu-id="2af8a-170">Variabili di ambiente impostate in questa sezione hanno la precedenza su sistema le variabili di ambiente per il processo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-170">Environment variables set in this section take precedence over system environment variables for the process.</span></span>

<span data-ttu-id="2af8a-171">Nell'esempio seguente imposta due variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="2af8a-171">The example below sets two environment variables.</span></span> <span data-ttu-id="2af8a-172">`ASPNETCORE_ENVIRONMENT`verrà configurato l'ambiente dell'applicazione per `Development`.</span><span class="sxs-lookup"><span data-stu-id="2af8a-172">`ASPNETCORE_ENVIRONMENT` will configure the application's environment to `Development`.</span></span> <span data-ttu-id="2af8a-173">Uno sviluppatore può impostare temporaneamente questo valore *Web. config* file per forzare il [pagina eccezione developer](xref:fundamentals/error-handling) da caricare durante il debug di un'eccezione di app.</span><span class="sxs-lookup"><span data-stu-id="2af8a-173">A developer may temporarily set this value in the *web.config* file in order to force the [developer exception page](xref:fundamentals/error-handling) to load when debugging an app exception.</span></span> <span data-ttu-id="2af8a-174">`CONFIG_DIR`è un esempio di una variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto codice che verrà letto il valore all'avvio in modo da formare un percorso per caricare il file di configurazione dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2af8a-174">`CONFIG_DIR` is an example of a user-defined environment variable, where the developer has written code that will read the value on startup to form a path in order to load the app's configuration file.</span></span>

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile="\\?\%home%\LogFiles\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

## <a name="appofflinehtm"></a><span data-ttu-id="2af8a-175">App_offline.htm</span><span class="sxs-lookup"><span data-stu-id="2af8a-175">app_offline.htm</span></span>

<span data-ttu-id="2af8a-176">Se si inserisce un file con il nome *app_offline.htm* alla radice di una directory dell'applicazione web, il modulo di base di ASP.NET sarà tenta di arrestare normalmente l'app e arrestare l'elaborazione delle richieste in ingresso.</span><span class="sxs-lookup"><span data-stu-id="2af8a-176">If you place a file with the name *app_offline.htm* at the root of a web application directory, the ASP.NET Core Module will attempt to gracefully shutdown the app and stop processing incoming requests.</span></span> <span data-ttu-id="2af8a-177">Se l'app è ancora in esecuzione `shutdownTimeLimit` numero di secondi, il modulo di base di ASP.NET sarà terminare il processo in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2af8a-177">If the app is still running after `shutdownTimeLimit` number of seconds, the ASP.NET Core Module will kill the running process.</span></span>

<span data-ttu-id="2af8a-178">Mentre il *app_offline.htm* file è presente, il modulo di base di ASP.NET risponderà alle richieste restituendo il contenuto del *app_offline.htm* file.</span><span class="sxs-lookup"><span data-stu-id="2af8a-178">While the *app_offline.htm* file is present, the ASP.NET Core Module will respond to requests by sending back the contents of the *app_offline.htm* file.</span></span> <span data-ttu-id="2af8a-179">Una volta il *app_offline.htm* file viene rimosso, alla successiva richiesta di caricamento dell'applicazione, che quindi risponde alle richieste.</span><span class="sxs-lookup"><span data-stu-id="2af8a-179">Once the *app_offline.htm* file is removed, the next request loads the application, which then responds to requests.</span></span>

## <a name="start-up-error-page"></a><span data-ttu-id="2af8a-180">Pagina di errore di avvio</span><span class="sxs-lookup"><span data-stu-id="2af8a-180">Start-up error page</span></span>

<span data-ttu-id="2af8a-181">Se il modulo di base di ASP.NET non è possibile avviare il processo di back-end o l'avvio di processo di back-end ma ha esito negativo per l'ascolto sulla porta configurata, si visualizzeranno una pagina di codice di stato HTTP 502.5.</span><span class="sxs-lookup"><span data-stu-id="2af8a-181">If the ASP.NET Core Module fails to launch the backend process or the backend process starts but fails to listen on the configured port, you will see an HTTP 502.5 status code page.</span></span> <span data-ttu-id="2af8a-182">Per non visualizzare questa pagina e tornare alla tabella codici predefinita IIS 502 stato, utilizzare il `disableStartUpErrorPage` attributo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-182">To suppress this page and revert to the default IIS 502 status code page, use the `disableStartUpErrorPage` attribute.</span></span> <span data-ttu-id="2af8a-183">Per ulteriori informazioni sulla configurazione di messaggi di errore personalizzati, vedere [errori HTTP `<httpErrors>` ](https://docs.microsoft.com/iis/configuration/system.webServer/httpErrors/).</span><span class="sxs-lookup"><span data-stu-id="2af8a-183">For more information on configuring custom error messages, see [HTTP Errors `<httpErrors>`](https://docs.microsoft.com/iis/configuration/system.webServer/httpErrors/).</span></span>

![Pagina stato 502](aspnet-core-module/_static/ANCM-502_5.png)

## <a name="log-creation-and-redirection"></a><span data-ttu-id="2af8a-185">Reindirizzamento e la creazione di log</span><span class="sxs-lookup"><span data-stu-id="2af8a-185">Log creation and redirection</span></span>

<span data-ttu-id="2af8a-186">Il modulo di base di ASP.NET reindirizza `stdout` e `stderr` log su disco se si imposta la `stdoutLogEnabled` e `stdoutLogFile` gli attributi del `aspNetCore` elemento.</span><span class="sxs-lookup"><span data-stu-id="2af8a-186">The ASP.NET Core Module redirects `stdout` and `stderr` logs to disk if you set the `stdoutLogEnabled` and `stdoutLogFile` attributes of the `aspNetCore` element.</span></span> <span data-ttu-id="2af8a-187">Le cartelle di `stdoutLogFile` percorso deve essere presente affinché il modulo creare il file di log.</span><span class="sxs-lookup"><span data-stu-id="2af8a-187">Any folders in the `stdoutLogFile` path must exist in order for the module to create the log file.</span></span> <span data-ttu-id="2af8a-188">Un'estensione di timestamp e il file verrà aggiunto automaticamente quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="2af8a-188">A timestamp and file extension will be added automatically when the log file is created.</span></span> <span data-ttu-id="2af8a-189">Log non vengono ruotati, a meno che non si verifica il riciclo o al riavvio del processo.</span><span class="sxs-lookup"><span data-stu-id="2af8a-189">Logs are not rotated, unless process recycling/restart occurs.</span></span> <span data-ttu-id="2af8a-190">È responsabilità dell'host per limitare lo spazio su disco, che utilizzano i log.</span><span class="sxs-lookup"><span data-stu-id="2af8a-190">It is the responsibility of the hoster to limit the disk space the logs consume.</span></span> <span data-ttu-id="2af8a-191">Utilizzo di `stdout` log è consigliato solo per la risoluzione dei problemi di avvio dell'applicazione e non per scopi di registrazione generali dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2af8a-191">Using the `stdout` log is only recommended for troubleshooting application startup issues and not for general application logging purposes.</span></span>

<span data-ttu-id="2af8a-192">Il nome del file di log è costituito da aggiungendo l'ID processo (PID), timestamp (*yyyyMdhms*) e l'estensione di file (*log*) per l'ultimo segmento del `stdoutLogFile` percorso (in genere *stdout* ) delimitati da caratteri di sottolineatura.</span><span class="sxs-lookup"><span data-stu-id="2af8a-192">The log file name is composed by appending the process ID (PID), timestamp (*yyyyMdhms*), and file extension (*.log*) to the last segment of the `stdoutLogFile` path (typically *stdout*) delimited by underscores.</span></span> <span data-ttu-id="2af8a-193">Ad esempio se il `stdoutLogFile` percorso termina con *stdout*, un log per un'app con un PID di 10652 creato in 10/8/2017 12:05:02 ha il nome del file *stdout_10652_20178101252.log*.</span><span class="sxs-lookup"><span data-stu-id="2af8a-193">For example if the `stdoutLogFile` path ends with *stdout*, a log for an app with a PID of 10652 created on 8/10/2017 at 12:05:02 has the file name *stdout_10652_20178101252.log*.</span></span>

<span data-ttu-id="2af8a-194">Di seguito è riportato un esempio `aspNetCore` elemento che configura `stdout` registrazione.</span><span class="sxs-lookup"><span data-stu-id="2af8a-194">Here's a sample `aspNetCore` element that configures `stdout` logging.</span></span> <span data-ttu-id="2af8a-195">Il `stdoutLogFile` illustrato nell'esempio di percorso è appropriato per il servizio App di Azure.</span><span class="sxs-lookup"><span data-stu-id="2af8a-195">The `stdoutLogFile` path shown in the example is appropriate for the Azure App Service.</span></span> <span data-ttu-id="2af8a-196">Un percorso locale o un percorso di condivisione di rete è accettabile per l'accesso locale.</span><span class="sxs-lookup"><span data-stu-id="2af8a-196">A local path or network share path is acceptable for local logging.</span></span> <span data-ttu-id="2af8a-197">Verificare che l'identità dell'utente AppPool disponga dell'autorizzazione di scrittura per il percorso specificato.</span><span class="sxs-lookup"><span data-stu-id="2af8a-197">Confirm that the AppPool user identity has permission to write to the path provided.</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile="\\?\%home%\LogFiles\stdout">
</aspNetCore>
```
<span data-ttu-id="2af8a-198">Vedere [configurazione tramite Web. config](#configuration-via-webconfig) per un esempio del `aspNetCore` elemento il *Web. config* file.</span><span class="sxs-lookup"><span data-stu-id="2af8a-198">See [Configuration via web.config](#configuration-via-webconfig) for an example of the `aspNetCore` element in the *web.config* file.</span></span>

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a><span data-ttu-id="2af8a-199">Modulo Core ASP.NET con IIS condiviso</span><span class="sxs-lookup"><span data-stu-id="2af8a-199">ASP.NET Core Module with an IIS Shared Configuration</span></span>

<span data-ttu-id="2af8a-200">Il programma di installazione del modulo di base di ASP.NET viene eseguito con i privilegi del **sistema** account.</span><span class="sxs-lookup"><span data-stu-id="2af8a-200">The ASP.NET Core Module installer runs with the privileges of the **SYSTEM** account.</span></span> <span data-ttu-id="2af8a-201">Poiché l'account sistema locale non dispone dell'autorizzazione Modifica per il percorso della condivisione utilizzata per la configurazione condivisa di IIS, il programma di installazione verrà raggiunto un errore di accesso negato durante il tentativo di configurare le impostazioni del modulo in  *applicationHost. config* nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="2af8a-201">Because the local system account does not have modify permission for the share path which is used by the IIS Shared Configuration, the installer will hit an access denied error when attempting to configure the module settings in *applicationHost.config* on the share.</span></span>

<span data-ttu-id="2af8a-202">È la soluzione non supportata per disabilitare la configurazione condivisa di IIS, eseguire il programma di installazione e l'esportazione aggiornato *applicationHost. config* alla condivisione di file e abilitare nuovamente la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="2af8a-202">The unsupported workaround is to disable the IIS Shared Configuration, run the installer, export the updated *applicationHost.config* file to the share, and re-enable the IIS Shared Configuration.</span></span>

## <a name="module-schema-and-configuration-file-locations"></a><span data-ttu-id="2af8a-203">Percorsi dei file di modulo, schemi e configurazione</span><span class="sxs-lookup"><span data-stu-id="2af8a-203">Module, schema, and configuration file locations</span></span>

### <a name="module"></a><span data-ttu-id="2af8a-204">Modulo</span><span class="sxs-lookup"><span data-stu-id="2af8a-204">Module</span></span>

<span data-ttu-id="2af8a-205">**IIS (x86 o amd64):**</span><span class="sxs-lookup"><span data-stu-id="2af8a-205">**IIS (x86/amd64):**</span></span>

   * <span data-ttu-id="2af8a-206">%windir%\System32\inetsrv\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="2af8a-206">%windir%\System32\inetsrv\aspnetcore.dll</span></span>

   * <span data-ttu-id="2af8a-207">%windir%\SysWOW64\inetsrv\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="2af8a-207">%windir%\SysWOW64\inetsrv\aspnetcore.dll</span></span>

<span data-ttu-id="2af8a-208">**IIS Express (x86 o amd64):**</span><span class="sxs-lookup"><span data-stu-id="2af8a-208">**IIS Express (x86/amd64):**</span></span>

   * <span data-ttu-id="2af8a-209">%ProgramFiles%\IIS Express\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="2af8a-209">%ProgramFiles%\IIS Express\aspnetcore.dll</span></span>

   * <span data-ttu-id="2af8a-210">% ProgramFiles (x86) %\IIS Express\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="2af8a-210">%ProgramFiles(x86)%\IIS Express\aspnetcore.dll</span></span>

### <a name="schema"></a><span data-ttu-id="2af8a-211">Schema</span><span class="sxs-lookup"><span data-stu-id="2af8a-211">Schema</span></span>

<span data-ttu-id="2af8a-212">**IIS**</span><span class="sxs-lookup"><span data-stu-id="2af8a-212">**IIS**</span></span>

   * <span data-ttu-id="2af8a-213">%windir%\System32\inetsrv\config\schema\aspnetcore_schema.Xml</span><span class="sxs-lookup"><span data-stu-id="2af8a-213">%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml</span></span>

<span data-ttu-id="2af8a-214">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="2af8a-214">**IIS Express**</span></span>

   * <span data-ttu-id="2af8a-215">%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml</span><span class="sxs-lookup"><span data-stu-id="2af8a-215">%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml</span></span>

### <a name="configuration"></a><span data-ttu-id="2af8a-216">Configurazione</span><span class="sxs-lookup"><span data-stu-id="2af8a-216">Configuration</span></span>

<span data-ttu-id="2af8a-217">**IIS**</span><span class="sxs-lookup"><span data-stu-id="2af8a-217">**IIS**</span></span>

   * <span data-ttu-id="2af8a-218">%windir%\System32\inetsrv\config\applicationHost.config</span><span class="sxs-lookup"><span data-stu-id="2af8a-218">%windir%\System32\inetsrv\config\applicationHost.config</span></span>

<span data-ttu-id="2af8a-219">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="2af8a-219">**IIS Express**</span></span>

   * <span data-ttu-id="2af8a-220">.vs\config\applicationHost.config</span><span class="sxs-lookup"><span data-stu-id="2af8a-220">.vs\config\applicationHost.config</span></span>

<span data-ttu-id="2af8a-221">È possibile cercare *aspnetcore.dll* nel *applicationHost. config* file.</span><span class="sxs-lookup"><span data-stu-id="2af8a-221">You can search for *aspnetcore.dll* in the *applicationHost.config* file.</span></span> <span data-ttu-id="2af8a-222">Per IIS Express, il *applicationHost. config* file non esiste per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="2af8a-222">For IIS Express, the *applicationHost.config* file won't exist by default.</span></span> <span data-ttu-id="2af8a-223">Il file viene creato in *{radice dell'applicazione}\.vs\config* quando si avvia un progetto di applicazione web nella soluzione di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2af8a-223">The file is created at *{application root}\.vs\config* when you start any web application project in the Visual Studio solution.</span></span>