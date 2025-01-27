---
title: Middleware di memorizzazione nella cache delle risposte in ASP.NET Core
author: guardrex
description: Informazioni su come configurare e usare il middleware di memorizzazione nella cache delle risposte in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/04/2019
uid: performance/caching/middleware
ms.openlocfilehash: a8e656e1d59114e2e953323e98e0a2399efca98a
ms.sourcegitcommit: 09f4a5ded39cc8204576fe801d760bd8b611f3aa
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 11/05/2019
ms.locfileid: "73611461"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>Middleware di memorizzazione nella cache delle risposte in ASP.NET Core

Di [Luke Latham](https://github.com/guardrex) e [John Luo](https://github.com/JunTaoLuo)

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/middleware/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

Questo articolo illustra come configurare il middleware di memorizzazione nella cache delle risposte in un'app ASP.NET Core. Il middleware determina quando le risposte sono memorizzabili nella cache, archivia le risposte e fornisce risposte dalla cache. Per un'introduzione alla memorizzazione nella cache HTTP e all'attributo [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) , vedere [caching delle risposte](xref:performance/caching/response).

## <a name="configuration"></a>Configurazione

::: moniker range=">= aspnetcore-3.0"

Il middleware di memorizzazione nella cache delle risposte è disponibile in modo implicito per le app ASP.NET Core tramite il Framework condiviso.

In `Startup.ConfigureServices`aggiungere il middleware di caching della risposta alla raccolta di servizi:

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurare l'app per l'uso del middleware con il metodo di estensione <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*>, che aggiunge il middleware alla pipeline di elaborazione delle richieste in `Startup.Configure`:

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=16)]

L'app di esempio aggiunge intestazioni per controllare la memorizzazione nella cache nelle richieste successive:

* Cache [-Control](https://tools.ietf.org/html/rfc7234#section-5.2) &ndash; memorizza nella cache le risposte memorizzabili nella cache per un massimo di 10 secondi.
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4) &ndash; configura il middleware in modo da fornire una risposta memorizzata nella cache solo se l'intestazione [`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4) delle richieste successive corrisponde a quella della richiesta originale.

[!code-csharp[](middleware/samples_snippets/3.x/AddHeaders.cs)]

Il middleware di memorizzazione nella cache delle risposte memorizza nella cache solo le risposte del server che generano un codice di stato 200 (OK). Qualsiasi altra risposta, incluse le [pagine di errore](xref:fundamentals/error-handling), viene ignorata dal middleware.

> [!WARNING]
> Le risposte contenenti contenuto per i client autenticati devono essere contrassegnate come non memorizzabili nella cache per impedire al middleware di archiviare e servire tali risposte. Per informazioni dettagliate sul modo in cui il middleware determina se una risposta è memorizzabile nella cache, vedere [condizioni per la memorizzazione nella cache](#conditions-for-caching) .

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Usare il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o aggiungere un riferimento al pacchetto [Microsoft. AspNetCore. ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) .

In `Startup.ConfigureServices`aggiungere il middleware di caching della risposta alla raccolta di servizi:

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurare l'app per l'uso del middleware con il metodo di estensione <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*>, che aggiunge il middleware alla pipeline di elaborazione delle richieste in `Startup.Configure`:

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=14)]

L'app di esempio aggiunge intestazioni per controllare la memorizzazione nella cache nelle richieste successive:

* Cache [-Control](https://tools.ietf.org/html/rfc7234#section-5.2) &ndash; memorizza nella cache le risposte memorizzabili nella cache per un massimo di 10 secondi.
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4) &ndash; configura il middleware in modo da fornire una risposta memorizzata nella cache solo se l'intestazione [`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4) delle richieste successive corrisponde a quella della richiesta originale.

[!code-csharp[](middleware/samples_snippets/2.x/AddHeaders.cs)]

Il middleware di memorizzazione nella cache delle risposte memorizza nella cache solo le risposte del server che generano un codice di stato 200 (OK). Qualsiasi altra risposta, incluse le [pagine di errore](xref:fundamentals/error-handling), viene ignorata dal middleware.

> [!WARNING]
> Le risposte contenenti contenuto per i client autenticati devono essere contrassegnate come non memorizzabili nella cache per impedire al middleware di archiviare e servire tali risposte. Per informazioni dettagliate sul modo in cui il middleware determina se una risposta è memorizzabile nella cache, vedere [condizioni per la memorizzazione nella cache](#conditions-for-caching) .

::: moniker-end

## <a name="options"></a>Opzioni

Le opzioni di memorizzazione nella cache delle risposte sono illustrate nella tabella seguente.

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | Dimensioni maggiori memorizzabili nella cache per il corpo della risposta in byte. Il valore predefinito è `64 * 1024 * 1024` (64 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | Limite delle dimensioni per il middleware della cache di risposta in byte. Il valore predefinito è `100 * 1024 * 1024` (100 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | Determina se le risposte vengono memorizzate nella cache nei percorsi che fanno distinzione Il valore predefinito è `false`. |

Nell'esempio seguente il middleware viene configurato per:

* Risposte della cache con dimensioni del corpo inferiori o uguali a 1.024 byte.
* Archiviare le risposte in base ai percorsi con distinzione tra maiuscole e minuscole. `/page1` e `/Page1`, ad esempio, vengono archiviati separatamente.

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

Quando si usano i controller dell'API Web o MVC o i modelli di pagina Razor Pages, l'attributo [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) specifica i parametri necessari per impostare le intestazioni appropriate per la memorizzazione nella cache delle risposte. L'unico parametro dell'attributo `[ResponseCache]` che richiede rigorosamente il middleware è <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys>, che non corrisponde a un'intestazione HTTP effettiva. Per ulteriori informazioni, vedere <xref:performance/caching/response#responsecache-attribute>.

Quando non si usa l'attributo `[ResponseCache]`, la memorizzazione nella cache delle risposte può variare con `VaryByQueryKeys`. Usare il <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> direttamente da [HttpContext. Features](xref:Microsoft.AspNetCore.Http.HttpContext.Features):

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

L'utilizzo di un singolo valore uguale a `*` in `VaryByQueryKeys` varia la cache in base a tutti i parametri di query della richiesta.

## <a name="http-headers-used-by-response-caching-middleware"></a>Intestazioni HTTP usate dal middleware di memorizzazione nella cache delle risposte

Nella tabella seguente vengono fornite informazioni sulle intestazioni HTTP che influiscono sulla memorizzazione nella cache delle risposte.

| Header | Dettagli |
| ------ | ------- |
| `Authorization` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. |
| `Cache-Control` | Il middleware considera solo le risposte di memorizzazione nella cache contrassegnate con la direttiva `public` cache. Controllare la memorizzazione nella cache con i parametri seguenti:<ul><li>validità massima</li><li>massimo-non aggiornato&#8224;</li><li>min-Fresh</li><li>must-revalidate</li><li>No-cache</li><li>Nessun archivio</li><li>solo-if-Cached</li><li>private</li><li>public</li><li>s-maxage</li><li>proxy-revalidate&#8225;</li></ul>&#8224;Se non viene specificato alcun limite per `max-stale`, il middleware non esegue alcuna azione.<br>&#8225;`proxy-revalidate` ha lo stesso effetto di `must-revalidate`.<br><br>Per altre informazioni, vedere [RFC 7231: direttive Cache-Control della richiesta](https://tools.ietf.org/html/rfc7234#section-5.2.1). |
| `Pragma` | Una `Pragma: no-cache` intestazione nella richiesta produce lo stesso effetto di `Cache-Control: no-cache`. Questa intestazione viene sottoposta a override dalle direttive rilevanti nell'intestazione `Cache-Control`, se presente. Considerato per compatibilità con le versioni precedenti di HTTP/1.0. |
| `Set-Cookie` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. Qualsiasi middleware nella pipeline di elaborazione delle richieste che imposta uno o più cookie impedisce al middleware di caching della risposta di memorizzare nella cache la risposta, ad esempio il [provider TempData basato su cookie](xref:fundamentals/app-state#tempdata).  |
| `Vary` | L'intestazione `Vary` viene utilizzata per variare la risposta memorizzata nella cache da un'altra intestazione. Ad esempio, memorizzare nella cache le risposte per codifica includendo l'intestazione `Vary: Accept-Encoding`, che memorizza nella cache le risposte per le richieste con intestazioni `Accept-Encoding: gzip` e `Accept-Encoding: text/plain` separatamente. Una risposta con un valore di intestazione di `*` non viene mai archiviata. |
| `Expires` | Una risposta ritenuta obsoleta da questa intestazione non viene archiviata o recuperata a meno che non venga sottoposta a override da altre intestazioni `Cache-Control`. |
| `If-None-Match` | La risposta completa viene servita dalla cache se il valore non è `*` e il `ETag` della risposta non corrisponde a nessuno dei valori forniti. In caso contrario, viene servita una risposta 304 (non modificata). |
| `If-Modified-Since` | Se l'intestazione `If-None-Match` non è presente, viene fornita una risposta completa dalla cache se la data di risposta memorizzata nella cache è più recente del valore specificato. In caso contrario, viene servita una risposta *304 non modificata* . |
| `Date` | Quando viene servito dalla cache, l'intestazione `Date` viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Content-Length` | Quando viene servito dalla cache, l'intestazione `Content-Length` viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Age` | L'intestazione `Age` inviata nella risposta originale viene ignorata. Il middleware calcola un nuovo valore quando serve una risposta memorizzata nella cache. |

## <a name="caching-respects-request-cache-control-directives"></a>Caching rispetta le direttive di controllo della cache delle richieste

Il middleware rispetta le regole della [specifica HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234#section-5.2). Le regole richiedono una cache per rispettare un'intestazione `Cache-Control` valida inviata dal client. In base alla specifica, un client può effettuare richieste con un valore di intestazione `no-cache` e forzare il server a generare una nuova risposta per ogni richiesta. Attualmente, non esiste alcun controllo dello sviluppatore su questo comportamento di Caching quando si usa il middleware perché il middleware rispetta la specifica di Caching ufficiale.

Per un maggiore controllo sul comportamento della memorizzazione nella cache, esplorare altre funzionalità di memorizzazione nella cache di ASP.NET Core. Vedere gli argomenti seguenti:

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>Troubleshooting

Se il comportamento di memorizzazione nella cache non è quello previsto, verificare che le risposte siano memorizzabili nella cache e in grado di essere servite dalla cache. Esaminare le intestazioni in ingresso della richiesta e le intestazioni in uscita della risposta. Abilitare la [registrazione](xref:fundamentals/logging/index) per facilitare il debug.

Durante il test e la risoluzione dei problemi relativi al comportamento di Caching, un browser può impostare intestazioni di richiesta che influiscono sulla memorizzazione nella cache in modi indesiderati Ad esempio, un browser può impostare l'intestazione `Cache-Control` su `no-cache` o `max-age=0` durante l'aggiornamento di una pagina. Gli strumenti seguenti possono impostare in modo esplicito le intestazioni delle richieste e sono preferiti per il test della memorizzazione nella cache:

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>Condizioni per la memorizzazione nella cache

* La richiesta deve avere come risultato una risposta del server con un codice di stato 200 (OK).
* Il metodo di richiesta deve essere GET o HEAD.
* In `Startup.Configure`il middleware di memorizzazione nella cache delle risposte deve essere inserito prima del middleware che richiede la memorizzazione nella cache. Per ulteriori informazioni, vedere <xref:fundamentals/middleware/index>.
* L'intestazione `Authorization` non deve essere presente.
* `Cache-Control` parametri di intestazione devono essere validi e la risposta deve essere contrassegnata `public` e non contrassegnata come `private`.
* Non è necessario che l'intestazione del `Pragma: no-cache` sia presente se l'intestazione del `Cache-Control` non è presente, perché l'intestazione del `Cache-Control` sostituisce l'intestazione del `Pragma` quando è presente.
* L'intestazione `Set-Cookie` non deve essere presente.
* `Vary` parametri di intestazione devono essere validi e diversi da `*`.
* Il valore dell'intestazione `Content-Length` (se impostato) deve corrispondere alla dimensione del corpo della risposta.
* Il <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature> non viene utilizzato.
* La risposta non deve essere obsoleta come specificato dall'intestazione `Expires` e dalle direttive `max-age` e `s-maxage` cache.
* Il buffer delle risposte deve avere esito positivo. Le dimensioni della risposta devono essere inferiori a quelle configurate o predefinite <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit>. Le dimensioni del corpo della risposta devono essere inferiori a quelle configurate o predefinite <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize>.
* La risposta deve essere memorizzata nella cache in base alle specifiche [RFC 7234](https://tools.ietf.org/html/rfc7234) . Ad esempio, la direttiva `no-store` non deve esistere nei campi di intestazione della richiesta o della risposta. Per informazioni dettagliate, vedere la *sezione 3: archiviazione delle risposte nelle cache* di [RFC 7234](https://tools.ietf.org/html/rfc7234) .

> [!NOTE]
> Il sistema antifalsificazione per la generazione di token protetti per evitare attacchi di richiesta intersito falsa (CSRF) imposta le intestazioni `Cache-Control` e `Pragma` su `no-cache` in modo che le risposte non vengano memorizzate nella cache. Per informazioni su come disabilitare i token antifalsificazione per gli elementi del form HTML, vedere <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration>.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
