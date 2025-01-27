---
title: Implementazioni di server Web in ASP.NET Core
author: rick-anderson
description: Individuare i server Web Kestrel e HTTP.sys per ASP.NET Core. Informazioni su come scegliere un server e quando usare un server proxy inverso.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/10/2019
uid: fundamentals/servers/index
ms.openlocfilehash: 3bdc2bf776946b8fae8886a37ecd3ed5e3f860fe
ms.sourcegitcommit: 7d3c6565dda6241eb13f9a8e1e1fd89b1cfe4d18
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/11/2019
ms.locfileid: "72259832"
---
# <a name="web-server-implementations-in-aspnet-core"></a>Implementazioni di server Web in ASP.NET Core

Di [Tom Dykstra](https://github.com/tdykstra), [Steve Smith](https://ardalis.com/), [Stephen Halter](https://twitter.com/halter73) e [Chris Ross](https://github.com/Tratcher)

Un'app ASP.NET Core viene eseguita con un'implementazione del server HTTP in-process. L'implementazione del server attende le richieste HTTP e le rende visibili all'app come un set di [funzionalità di richiesta](xref:fundamentals/request-features) combinate in un <xref:Microsoft.AspNetCore.Http.HttpContext>.

## <a name="kestrel"></a>Kestrel

Kestrel è il server Web predefinito incluso nei modelli di progetto di ASP.NET Core.

Usare Kestrel:

* Da solo come server perimetrale che elabora le richieste direttamente da una rete, inclusa Internet.

  ![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

* Con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/). Il server proxy inverso riceve le richieste HTTP da Internet e le inoltra a Kestrel.

  ![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

È supportata la configurazione host @ no__t-0with o senza un server proxy inverso @ no__t-1is.

Per indicazioni per la configurazione di Kestrel e per informazioni su quando usare Kestrel in una configurazione con proxy inverso, vedere <xref:fundamentals/servers/kestrel>.

::: moniker range=">= aspnetcore-2.2"

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

ASP.NET Core include quanto segue:

* Il [server Kestrel](xref:fundamentals/servers/kestrel) è l'implementazione di server HTTP multipiattaforma predefinita.
* Il server HTTP IIS è un [server in-process](#hosting-models) per IIS.
* Il [server HTTP.sys](xref:fundamentals/servers/httpsys) è un server HTTP solo per Windows basato sul [driver del kernel HTTP.sys e l'API HTTP Server](/windows/desktop/Http/http-api-start-page).

Quando si usa [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) oppure [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), l'app viene eseguita:

* Nello stesso processo del processo di lavoro IIS ([modello di hosting in-process](#hosting-models)) con il server HTTP IIS. *In-process* è la configurazione consigliata.
* In un processo separato dal processo di lavoro IIS ([modello di hosting out-of-process](#hosting-models)) con il [server Kestrel](#kestrel).

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) è un modulo IIS nativo che gestisce le richieste IIS native tra IIS e il server HTTP IIS in-process o il server Kestrel. Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module>.

## <a name="hosting-models"></a>Modelli di hosting

Se si usa l'hosting in-process, un'app ASP.NET Core esegue lo stesso processo del processo di lavoro IIS. L'hosting in-process offre prestazioni migliori rispetto all'hosting out-of-process perché le richieste non vengono inserite in proxy sulla scheda di loopback, un'interfaccia di rete che restituisce il traffico di rete in uscita allo stesso computer. Per gestire il processo, IIS usa il [servizio Attivazione processo Windows (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Quando si usa l'hosting out-of-process, le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS e il modulo esegue la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo. Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Per altre informazioni e indicazioni per la configurazione, vedere gli argomenti seguenti:

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>

# <a name="macostabmacos"></a>[macOS](#tab/macos)

ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.

---

::: moniker-end

::: moniker range="< aspnetcore-2.2"

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

ASP.NET Core include quanto segue:

* Il [server kestrel](xref:fundamentals/servers/kestrel) è il server HTTP multipiattaforma predefinito.
* Il [server HTTP.sys](xref:fundamentals/servers/httpsys) è un server HTTP solo per Windows basato sul [driver del kernel HTTP.sys e l'API HTTP Server](/windows/desktop/Http/http-api-start-page).

Quando si usa [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) o [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), l'app viene eseguita in un processo separato dal processo di lavoro IIS (*out-of-process*) con il [server Kestrel](#kestrel).

Poiché le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS, il modulo esegue la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo. Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata out-of-process:

![Modulo ASP.NET Core](_static/ancm-outofprocess.png)

Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel. Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS). Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.

Il modulo specifica la porta tramite una variabile di ambiente all'avvio e il [middleware di integrazione IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configura il server per l'ascolto su `http://localhost:{port}`. Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate. Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.

Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel. La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.

Per indicazioni sulla configurazione di IIS e del modulo ASP.NET Core, vedere gli argomenti seguenti:

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>

# <a name="macostabmacos"></a>[macOS](#tab/macos)

ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.

---

::: moniker-end

### <a name="nginx-with-kestrel"></a>Nginx con Kestrel

Per informazioni su come usare Nginx in Linux come server proxy inverso per Kestrel, vedere <xref:host-and-deploy/linux-nginx>.

### <a name="apache-with-kestrel"></a>Apache con Kestrel

Per informazioni su come usare Apache in Linux come server proxy inverso per Kestrel, vedere <xref:host-and-deploy/linux-apache>.

## <a name="httpsys"></a>HTTP.sys

Se si eseguono app ASP.NET Core in Windows, HTTP.sys è un'alternativa a Kestrel. Kestrel è in genere consigliato per ottenere prestazioni ottimali. HTTP.sys è utilizzabile negli scenari in cui l'app viene esposta a Internet e le funzionalità necessarie sono supportate da HTTP.sys, ma non da Kestrel. Per altre informazioni, vedere <xref:fundamentals/servers/httpsys>.

![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

HTTP.sys può essere usato anche per le app che vengono esposte solo a una rete interna.

![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

Per indicazioni per la configurazione di HTTP.sys, vedere <xref:fundamentals/servers/httpsys>.

## <a name="aspnet-core-server-infrastructure"></a>Infrastruttura del server ASP.NET Core

L'oggetto <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> disponibile nel metodo `Startup.Configure` espone la proprietà <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ServerFeatures> del tipo <xref:Microsoft.AspNetCore.Http.Features.IFeatureCollection>. Kestrel e HTTP.sys espongono una singola funzionalità ciascuno, <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature>, ma implementazioni server diverse potrebbero esporre funzionalità aggiuntive.

È possibile usare `IServerAddressesFeature` per individuare la porta a cui è associata l'implementazione server in fase di esecuzione.

## <a name="custom-servers"></a>Server personalizzati

Se i server predefiniti non soddisfano i requisiti dell'app, è possibile creare un'implementazione server personalizzata. La [guida a Open Web Interface for .NET (OWIN)](xref:fundamentals/owin) spiega come scrivere un'implementazione [Nowin](https://github.com/Bobris/Nowin) di <xref:Microsoft.AspNetCore.Hosting.Server.IServer>. È necessario implementare solo le interfacce delle funzionalità usate dall'app, anche se è richiesto come minimo il supporto di <xref:Microsoft.AspNetCore.Http.Features.IHttpRequestFeature> e <xref:Microsoft.AspNetCore.Http.Features.IHttpResponseFeature>.

## <a name="server-startup"></a>Avvio del server

Il server viene avviato quando l'editor o l'ambiente di sviluppo integrato (IDE) avvia l'app:

* [Visual Studio](https://visualstudio.microsoft.com) &ndash; È possibile usare i profili di avvio per avviare l'app e il server con [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)/[Modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) o la console.
* [Visual Studio Code](https://code.visualstudio.com/) &ndash; L'app e il server vengono avviati da [Omnisharp](https://github.com/OmniSharp/omnisharp-vscode), che attiva il debugger CoreCLR.
* [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) &ndash; L'app e il server vengono avviati dal [debugger in modalità Mono Soft](https://www.mono-project.com/docs/advanced/runtime/docs/soft-debugger/).

All'avvio dell'app dal prompt dei comandi nella cartella del progetto, [dotnet run](/dotnet/core/tools/dotnet-run) avvia l'app e il server (solo Kestrel e HTTP.sys). La configurazione viene specificata dall'opzione `-c|--configuration`, impostata su `Debug` (valore predefinito) o su `Release`. Se sono presenti profili di avvio in un file *launchSettings.json*, usare l'opzione `--launch-profile <NAME>` per impostare il profilo di avvio (ad esempio, `Development` o `Production`). Per altre informazioni, vedere [dotnet run](/dotnet/core/tools/dotnet-run) e [Creazione di pacchetti di distribuzione di .NET Core](/dotnet/core/build/distribution-packaging).

## <a name="http2-support"></a>Supporto per HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è supportato con ASP.NET Core negli scenari di distribuzione seguenti:

::: moniker range=">= aspnetcore-2.2"

* [Kestrel](xref:fundamentals/servers/kestrel#http2-support)
  * Sistema operativo
    * Windows Server 2016/Windows 10 o versioni successive&dagger;
    * Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)
    * HTTP/2 verrà supportato in macOS in una versione futura.
  * Framework di destinazione: .NET Core 2.2 o versioni successive
* [HTTP.sys](xref:fundamentals/servers/httpsys#http2-support)
  * Windows Server 2016/Windows 10 o versioni successive
  * Framework di destinazione: non applicabile alle distribuzioni HTTP.sys.
* [IIS (in-process)](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
  * Framework di destinazione: .NET Core 2.2 o versioni successive
* [IIS (out-of-process)](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
  * Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso a Kestrel usa HTTP/1.1.
  * Framework di destinazione: non applicabile alle distribuzioni IIS out-of-process.

&dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1. Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato. Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* [HTTP.sys](xref:fundamentals/servers/httpsys#http2-support)
  * Windows Server 2016/Windows 10 o versioni successive
  * Framework di destinazione: non applicabile alle distribuzioni HTTP.sys.
* [IIS (out-of-process)](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
  * Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso a Kestrel usa HTTP/1.1.
  * Framework di destinazione: non applicabile alle distribuzioni IIS out-of-process.

::: moniker-end

Una connessione HTTP/2 deve usare [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3) e TLS 1.2 o versioni successive. Per altre informazioni, vedere gli argomenti relativi agli scenari di distribuzione server.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/servers/kestrel>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:fundamentals/servers/httpsys>
