---
title: Effettuare richieste HTTP usando IHttpClientFactory in ASP.NET Core
author: stevejgordon
description: Informazioni sull'uso dell'interfaccia IHttpClientFactory per gestire le istanze di HttpClient logiche in ASP.NET Core.
ms.author: scaddie
ms.custom: mvc
ms.date: 10/27/2019
uid: fundamentals/http-requests
ms.openlocfilehash: a963833acfa12889c8ae3dac443962682e1cb931
ms.sourcegitcommit: 032113208bb55ecfb2faeb6d3e9ea44eea827950
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/31/2019
ms.locfileid: "73190589"
---
# <a name="make-http-requests-using-ihttpclientfactory-in-aspnet-core"></a>Effettuare richieste HTTP usando IHttpClientFactory in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak), [Steve Gordon](https://github.com/stevejgordon), [Rick Anderson](https://twitter.com/RickAndMSFT)e [Kirk Larkin](https://github.com/serpent5)

È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app. `IHttpClientFactory` offre i seguenti vantaggi:

* Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche. Ad esempio, un client denominato *GitHub* potrebbe essere registrato e configurato per accedere a [GitHub](https://github.com/). Un client predefinito può essere registrato per l'accesso generale.
* Codifica il concetto di middleware in uscita tramite la delega dei gestori in `HttpClient`. Fornisce le estensioni per il middleware basato su Polly per sfruttare i vantaggi delegare i gestori in `HttpClient`.
* Gestisce il pool e la durata delle istanze di `HttpClientMessageHandler` sottostanti. La gestione automatica evita i problemi comuni di DNS (Domain Name System) che si verificano quando si gestiscono manualmente `HttpClient` durate.
* Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample)).

Il codice di esempio in questa versione dell'argomento USA <xref:System.Text.Json> per deserializzare il contenuto JSON restituito nelle risposte HTTP. Per esempi che usano `Json.NET` e `ReadAsAsync<T>`, usare il selettore di versione per selezionare una versione 2. x di questo argomento.

## <a name="consumption-patterns"></a>Modelli di consumo

`IHttpClientFactory` può essere usato in un'app in diversi modi:

* [Utilizzo di base](#basic-usage)
* [Client denominati](#named-clients)
* [Client tipizzati](#typed-clients)
* [Client generati](#generated-clients)

L'approccio migliore dipende dai requisiti dell'app.

### <a name="basic-usage"></a>Utilizzo di base

`IHttpClientFactory` può essere registrato chiamando `AddHttpClient`:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

È possibile richiedere un `IHttpClientFactory` usando l' [inserimento di dipendenze](xref:fundamentals/dependency-injection). Il codice seguente usa `IHttpClientFactory` per creare un'istanza di `HttpClient`:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

L'uso di `IHttpClientFactory` come nell'esempio precedente è un modo efficace per effettuare il refactoring di un'app esistente. Non ha alcun effetto sul modo in cui viene usato `HttpClient`. Nelle posizioni in cui vengono create `HttpClient` istanze in un'app esistente, sostituire tali occorrenze con le chiamate a <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.

### <a name="named-clients"></a>Client denominati

I client denominati sono una scelta ottimale nei casi seguenti:

* L'app richiede molti usi distinti di `HttpClient`.
* Molti `HttpClient`s hanno una configurazione diversa.

La configurazione di un `HttpClient` denominato può essere specificata durante la registrazione in `Startup.ConfigureServices`:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

Nel codice precedente il client viene configurato con:

* Indirizzo di base `https://api.github.com/`.
* Due intestazioni necessarie per lavorare con l'API GitHub.

#### <a name="createclient"></a>CreateClient

Ogni volta che viene chiamato <xref:System.Net.Http.IHttpClientFactory.CreateClient*>:

* Viene creata una nuova istanza di `HttpClient`.
* Viene chiamata l'azione di configurazione.

Per creare un client denominato, passare il nome in `CreateClient`:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

Nel codice precedente non è necessario che la richiesta specifichi un nome host. Il codice può passare solo il percorso, perché viene usato l'indirizzo di base configurato per il client.

### <a name="typed-clients"></a>Client tipizzati

Client tipizzati:

* Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.
* Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.
* La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione. Ad esempio, è possibile usare un singolo client tipizzato:
  * Per un singolo endpoint back-end.
  * Per incapsulare tutta la logica che riguarda l'endpoint.
* Usare DI e può essere inserito laddove necessario nell'app.

Un client tipizzato accetta il parametro `HttpClient` nel proprio costruttore:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

Nel codice precedente:

* La configurazione viene spostata nel client tipizzato.
* L'oggetto `HttpClient` viene esposto come una proprietà pubblica.

È possibile creare metodi specifici dell'API che espongono `HttpClient` funzionalità. Ad esempio, il metodo `GetAspNetDocsIssues` incapsula il codice per recuperare i problemi aperti.

Il codice seguente chiama <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> in `Startup.ConfigureServices` per registrare una classe client tipizzata:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze. Il client tipizzato può essere inserito e usato direttamente:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

La configurazione di un client tipizzato può essere specificata durante la registrazione in `Startup.ConfigureServices`, anziché nel costruttore del client tipizzato:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

Il `HttpClient` può essere incapsulato all'interno di un client tipizzato. Anziché esporla come proprietà, definire un metodo che chiama l'istanza di `HttpClient` internamente:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

Nel codice precedente, il `HttpClient` viene archiviato in un campo privato. L'accesso alla `HttpClient` è dal metodo public `GetRepos`.

### <a name="generated-clients"></a>Client generati

`IHttpClientFactory` può essere usato in combinazione con librerie di terze parti, ad esempio il [refitting](https://github.com/paulcbetts/refit). Refit è una libreria REST per .NET. Converte le API REST in interfacce live. Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.

Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddControllers();
}
```

L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a>Middleware per richieste in uscita

`HttpClient` ha il concetto di delegare i gestori che possono essere collegati insieme per le richieste HTTP in uscita. `IHttpClientFactory`:

* Semplifica la definizione dei gestori da applicare per ogni client denominato.
* Supporta la registrazione e il concatenamento di più gestori per compilare una pipeline middleware per le richieste in uscita. Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita. Questo modello:

  * È simile alla pipeline del middleware in ingresso in ASP.NET Core.
  * Fornisce un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, ad esempio:

    * memorizzazione nella cache
    * gestione errori
    * serializzazione
    * registrazione

Per creare un gestore di delega:

* Derivare da <xref:System.Net.Http.DelegatingHandler>.
* Eseguire l'override di <xref:System.Net.Http.DelegatingHandler.SendAsync*>. Eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

Il codice precedente controlla se l'intestazione `X-API-KEY` è presente nella richiesta. Se `X-API-KEY` è mancante, viene restituito <xref:System.Net.HttpStatusCode.BadRequest>.

È possibile aggiungere più di un gestore alla configurazione per un `HttpClient` con <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*?displayProperty=fullName>:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup2.cs?name=snippet1)]

Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze. L'interfaccia `IHttpClientFactory` crea un ambito di inserimento delle dipendenze separato per ogni gestore. I gestori possono dipendere da servizi di qualsiasi ambito. I servizi da cui dipendono i gestori vengono eliminati al momento dell'eliminazione del gestore.

Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.

È possibile registrare più gestori nell'ordine di esecuzione. Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:

* Passare i dati nel gestore usando [HttpRequestMessage. Properties](xref:System.Net.Http.HttpRequestMessage.Properties).
* Usare <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> per accedere alla richiesta corrente.
* Creare un oggetto di archiviazione <xref:System.Threading.AsyncLocal`1> personalizzato per passare i dati.

## <a name="use-polly-based-handlers"></a>Usare gestori basati su Polly

`IHttpClientFactory` si integra con la libreria di terze parti [Polly](https://github.com/App-vNext/Polly). Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET. Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.

Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione. Le estensioni di Polly supportano l'aggiunta di gestori basati su Polly ai client. Polly richiede il pacchetto NuGet [Microsoft. Extensions. http. Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) .

### <a name="handle-transient-faults"></a>Gestire gli errori temporanei

Gli errori si verificano in genere quando le chiamate HTTP esterne sono temporanee. <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> consente di definire un criterio per gestire gli errori temporanei. I criteri configurati con `AddTransientHttpErrorPolicy` gestiscono le risposte seguenti:

* <xref:System.Net.Http.HttpRequestException>
* 5xx HTTP
* HTTP 408

`AddTransientHttpErrorPolicy` fornisce l'accesso a un oggetto `PolicyBuilder` configurato per gestire gli errori che rappresentano un possibile errore temporaneo:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup3.cs?name=snippet1)]

Nel codice precedente viene definito un criterio `WaitAndRetryAsync`. Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.

### <a name="dynamically-select-policies"></a>Selezionare i criteri in modo dinamico

I metodi di estensione vengono forniti per aggiungere gestori basati su Polly, ad esempio <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandler*>. L'overload di `AddPolicyHandler` seguente controlla la richiesta per decidere quali criteri applicare:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi. Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.

### <a name="add-multiple-polly-handlers"></a>Aggiungere più gestori Polly

È comune annidare i criteri Polly:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

Nell'esempio precedente:

* Vengono aggiunti due gestori.
* Il primo gestore USA <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> per aggiungere un criterio di ripetizione dei tentativi. Le richieste non riuscite vengono ritentate fino a tre volte.
* La seconda chiamata di `AddTransientHttpErrorPolicy` aggiunge un criterio di interruttore. Altre richieste esterne vengono bloccate per 30 secondi se 5 tentativi non riusciti si verificano in sequenza. I criteri dell'interruttore di circuito sono con stato. Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.

### <a name="add-policies-from-the-polly-registry"></a>Aggiungere criteri dal registro Polly

Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`.

Nel codice seguente:

* Vengono aggiunti i criteri "regular" e "Long".
* <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandlerFromRegistry*> aggiunge i criteri "regular" e "Long" dal registro di sistema.

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup4.cs?name=snippet1)]

Per ulteriori informazioni sulle integrazioni `IHttpClientFactory` e Polly, vedere il [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).

## <a name="httpclient-and-lifetime-management"></a>Gestione di HttpClient e durata

Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`. Viene creato un <xref:System.Net.Http.HttpMessageHandler> per ogni client denominato. La factory gestisce le durate delle istanze di `HttpMessageHandler`.

`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse. Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.

Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti. La creazione di più gestori del necessario può causare ritardi di connessione. Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito, che possono impedire al gestore di reagire alle modifiche DNS (Domain Name System).

La durata del gestore predefinito è di due minuti. È possibile eseguire l'override del valore predefinito in base al client denominato:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup5.cs?name=snippet1)]

le istanze di `HttpClient` possono in genere essere considerate come oggetti .NET che **non** richiedono l'eliminazione. L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>. `IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`.

Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`. Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.

## <a name="logging"></a>Registrazione

I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste. Abilitare il livello di informazioni appropriato nella configurazione di registrazione per visualizzare i messaggi di log predefiniti. La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.

La categoria di log usata per ogni client include il nome del client. Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria "System .NET. http. HttpClient. **MyNamedClient**. LogicalHandler". I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste. Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata. Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.

La registrazione avviene anche all'interno della pipeline del gestore delle richieste. Nell'esempio *MyNamedClient* i messaggi vengono registrati con la categoria di log "System .NET. http. HttpClient. **MyNamedClient**. ClientHandler". Per la richiesta, questo errore si verifica dopo l'esecuzione di tutti gli altri gestori e immediatamente prima dell'invio della richiesta. Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.

L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline. Questo può includere modifiche alle intestazioni delle richieste o al codice di stato della risposta.

L'inclusione del nome del client nella categoria log consente il filtraggio dei log per specifici client denominati.

## <a name="configure-the-httpmessagehandler"></a>Configurare HttpMessageHandler

Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.

Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`. È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato. Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup6.cs?name=snippet1)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>Usare IHttpClientFactory in un'app console

In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

Nell'esempio seguente:

* <xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).
* `MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`. `HttpClient` viene usato per recuperare una pagina Web.
* `Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.

[!code-csharp[](http-requests/samples/3.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [Implementazione dello schema Circuit Breaker](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Di [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak) e [Steve Gordon](https://github.com/stevejgordon)

È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app. I vantaggi offerti sono i seguenti:

* Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche. Ad esempio, è possibile registrare e configurare un client *github* per accedere a [GitHub](https://github.com/). È possibile registrare un client predefinito per altri scopi.
* Codifica il concetto di middleware in uscita tramite la delega di gestori in `HttpClient` e offre estensioni per il middleware basato su Polly per sfruttarne i vantaggi.
* Gestisce il pooling e la durata delle istanze di `HttpClientMessageHandler` sottostanti per evitare problemi DNS comuni che si verificano quando le durate di `HttpClient` vengono gestite manualmente.
* Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="consumption-patterns"></a>Modelli di consumo

`IHttpClientFactory` può essere usato in un'app in diversi modi:

* [Utilizzo di base](#basic-usage)
* [Client denominati](#named-clients)
* [Client tipizzati](#typed-clients)
* [Client generati](#generated-clients)

Nessuno di questi modi può essere considerato superiore a un altro. L'approccio migliore dipende dai vincoli dell'app.

### <a name="basic-usage"></a>Utilizzo di base

È possibile registrare `IHttpClientFactory` chiamando il metodo di estensione `AddHttpClient` in `IServiceCollection`, all'interno del metodo `Startup.ConfigureServices`.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

Dopo la registrazione, il codice può accettare un'interfaccia `IHttpClientFactory` ovunque sia possibile inserire servizi con l'[inserimento di dipendenze](xref:fundamentals/dependency-injection). `IHttpClientFactory` può essere usato per creare un'istanza di `HttpClient`:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

Questo uso di `IHttpClientFactory` è un modo efficace per effettuare il refactoring di un'app esistente. Non influisce in alcun modo sulle modalità d'uso di `HttpClient`. Nelle posizioni in cui vengono attualmente create istanze di `HttpClient`, sostituire le occorrenze con una chiamata a <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.

### <a name="named-clients"></a>Client denominati

Se un'app richiede molti usi distinti di `HttpClient`, ognuno con una configurazione diversa, un'opzione è l'uso di **client denominati**. La configurazione di un `HttpClient` denominato può essere specificata durante la registrazione in `Startup.ConfigureServices`.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

Nel codice precedente viene chiamato `AddHttpClient`, in cui viene specificato il nome *github*. Al client viene applicata una configurazione predefinita, ovvero l'indirizzo di base e due intestazioni necessari per l'uso dell'API GitHub.

Ogni volta che `CreateClient` viene chiamato, verrà creata una nuova istanza di `HttpClient` e verrà chiamata l'azione di configurazione.

Per usare un client denominato, è possibile passare un parametro di stringa a `CreateClient`. Specificare il nome del client da creare:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

Nel codice precedente non è necessario che la richiesta specifichi un nome host. Poiché viene usato l'indirizzo di base configurato per il client, è possibile passare solo il percorso.

### <a name="typed-clients"></a>Client tipizzati

Client tipizzati:

* Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.
* Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.
* La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione. Per un singolo endpoint back-end è ad esempio possibile usare un unico client tipizzato che incapsuli tutta la logica relativa all'endpoint.
* Usano l'inserimento di dipendenze e possono essere inseriti dove necessario nell'app.

Un client tipizzato accetta il parametro `HttpClient` nel proprio costruttore:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

Nel codice precedente la configurazione viene spostata nel client tipizzato. L'oggetto `HttpClient` viene esposto come una proprietà pubblica. È possibile definire metodi di API specifiche che espongono la funzionalità `HttpClient`. Il metodo `GetAspNetDocsIssues` incapsula il codice necessario per eseguire una query e analizzare gli ultimi problemi aperti in un repository GitHub.

Per registrare un client tipizzato è possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> generico all'interno di `Startup.ConfigureServices`, specificando la classe del client tipizzato:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze. Il client tipizzato può essere inserito e usato direttamente:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

Se si preferisce, è possibile specificare la configurazione di un client tipizzato durante la registrazione in `Startup.ConfigureServices`, anziché nel relativo costruttore:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

È possibile incapsulare interamente `HttpClient` all'interno di un client tipizzato. Anziché esporlo come una proprietà, è possibile specificare metodi pubblici che chiamano l'istanza di `HttpClient` internamente.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

Nel codice precedente `HttpClient` viene archiviato come un campo privato. L'accesso per effettuare chiamate esterne passa attraverso il metodo `GetRepos`.

### <a name="generated-clients"></a>Client generati

È possibile usare `IHttpClientFactory` in combinazione con altre librerie di terze parti, ad esempio [Refit](https://github.com/paulcbetts/refit). Refit è una libreria REST per .NET. Converte le API REST in interfacce live. Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.

Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("https://localhost:5001");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a>Middleware per richieste in uscita

`HttpClient` include già il concetto di delega di gestori concatenati per le richieste HTTP in uscita. `IHttpClientFactory` semplifica la definizione dei gestori da applicare per ogni client denominato. Supporta la registrazione e il concatenamento di più gestori per creare una pipeline di middleware per le richieste in uscita. Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita. Questo modello è simile alla pipeline di middleware in ingresso in ASP.NET Core. Il modello offre un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, tra cui memorizzazione nella cache, gestione degli errori, serializzazione e registrazione.

Per creare un gestore, definire una classe che deriva da <xref:System.Net.Http.DelegatingHandler>. Eseguire l'override del metodo `SendAsync` per eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

Il codice precedente definisce un gestore di base. Verifica se un'intestazione `X-API-KEY` è stata inclusa nella richiesta. Se l'intestazione non è presente, può evitare la chiamata HTTP e restituire una risposta appropriata.

Durante la registrazione è possibile aggiungere alla configurazione uno o più gestori per un'istanza di `HttpClient`. Questa attività viene eseguita tramite metodi di estensione in <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze. L'interfaccia `IHttpClientFactory` crea un ambito di inserimento delle dipendenze separato per ogni gestore. I gestori possono dipendere da servizi di qualsiasi ambito. I servizi da cui dipendono i gestori vengono eliminati al momento dell'eliminazione del gestore.

Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.

È possibile registrare più gestori nell'ordine di esecuzione. Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:

* Passare i dati al gestore usando `HttpRequestMessage.Properties`.
* Usare `IHttpContextAccessor` per accedere alla richiesta corrente.
* Creare un oggetto di archiviazione `AsyncLocal` personalizzato per passare i dati.

## <a name="use-polly-based-handlers"></a>Usare gestori basati su Polly

`IHttpClientFactory` si integra con una libreria di terze parti piuttosto diffusa denominata [Polly](https://github.com/App-vNext/Polly). Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET. Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.

Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione. Le estensioni Polly:

* Supportano l'aggiunta di gestori basati su Polly ai client.
* Possono essere usate dopo l'installazione del pacchetto NuGet [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/). Il pacchetto non è incluso nel framework condiviso ASP.NET Core.

### <a name="handle-transient-faults"></a>Gestire gli errori temporanei

La maggior parte degli errori comuni si verifica quando le chiamate HTTP esterne sono temporanee. Per definire un criterio in grado di gestire gli errori temporanei è disponibile un pratico metodo di estensione denominato `AddTransientHttpErrorPolicy`. I criteri configurati con questo metodo di estensione gestiscono `HttpRequestException`, risposte HTTP 5xx e risposte HTTP 408.

L'estensione `AddTransientHttpErrorPolicy` può essere usata all'interno di `Startup.ConfigureServices`. L'estensione consente l'accesso a un oggetto `PolicyBuilder` configurato per gestire gli errori che rappresentano un possibile errore temporaneo:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

Nel codice precedente viene definito un criterio `WaitAndRetryAsync`. Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.

### <a name="dynamically-select-policies"></a>Selezionare i criteri in modo dinamico

Per aggiungere gestori basati su Polly è possibile usare altri metodi di estensione. Una di queste estensioni è `AddPolicyHandler`, che include più overload. Un overload consente l'ispezione della richiesta al momento della definizione del criterio da applicare:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi. Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.

### <a name="add-multiple-polly-handlers"></a>Aggiungere più gestori Polly

In molti casi i criteri Polly vengono annidati per offrire funzionalità avanzate:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

Nell'esempio precedente vengono aggiunti due gestori. Il primo usa l'estensione `AddTransientHttpErrorPolicy` per aggiungere criteri di ripetizione. Le richieste non riuscite vengono ritentate fino a tre volte. La seconda chiamata a `AddTransientHttpErrorPolicy` aggiunge criteri dell'interruttore di circuito. Ulteriori richieste esterne vengono bloccate per 30 secondi nel caso si verifichino cinque tentativi non riusciti consecutivi. I criteri dell'interruttore di circuito sono con stato. Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.

### <a name="add-policies-from-the-polly-registry"></a>Aggiungere criteri dal registro Polly

Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`. Per aggiungere un gestore usando un criterio del registro è disponibile un metodo di estensione:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

Nel codice precedente vengono registrati due criteri quando si aggiunge `PolicyRegistry` a `ServiceCollection`. Per usare un criterio dal registro viene usato il metodo `AddPolicyHandlerFromRegistry` passando il nome del criterio da applicare.

Altre informazioni su `IHttpClientFactory` e le integrazioni con Polly sono disponibili nel [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).

## <a name="httpclient-and-lifetime-management"></a>Gestione di HttpClient e durata

Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`. È presente un <xref:System.Net.Http.HttpMessageHandler> per ogni client denominato. La factory gestisce le durate delle istanze di `HttpMessageHandler`.

`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse. Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.

Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti. La creazione di più gestori del necessario può causare ritardi di connessione. Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito. Ciò può impedire al gestore di reagire alle modifiche DNS.

La durata del gestore predefinito è di due minuti. Il valore predefinito può essere sottoposto a override per ogni client denominato. Per eseguire l'override, chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> nell'elemento `IHttpClientBuilder` restituito al momento della creazione del client:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

L'eliminazione del client non è obbligatoria. L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>. `IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`. Le istanze di `HttpClient` possono essere considerate a livello generale come oggetti .NET che non richiedono l'eliminazione.

Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`. Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.

## <a name="logging"></a>Registrazione

I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste. Per visualizzare i messaggi di log predefiniti, abilitare il livello di informazioni appropriato nella configurazione di registrazione. La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.

La categoria di log usata per ogni client include il nome del client. Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`. I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste. Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata. Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.

La registrazione avviene anche all'interno della pipeline del gestore delle richieste. Nell'esempio *MyNamedClient*, i messaggi vengono registrati nella categoria di log `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`. Per la richiesta, ciò avviene dopo l'esecuzione di tutti gli altri gestori e immediatamente prima che la richiesta sia inviata in rete. Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.

L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline. Le modifiche possono ad esempio riguardare le intestazioni delle richieste o il codice di stato della risposta.

L'inclusione del nome del client nella categoria di log consente di filtrare i log in base a client denominati specifici, se necessario.

## <a name="configure-the-httpmessagehandler"></a>Configurare HttpMessageHandler

Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.

Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`. È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato. Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>Usare IHttpClientFactory in un'app console

In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

Nell'esempio seguente:

* <xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).
* `MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`. `HttpClient` viene usato per recuperare una pagina Web.
* `Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [Implementazione dello schema Circuit Breaker](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end

::: moniker range="<= aspnetcore-2.1"

Di [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak) e [Steve Gordon](https://github.com/stevejgordon)

È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app. I vantaggi offerti sono i seguenti:

* Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche. Ad esempio, è possibile registrare e configurare un client *github* per accedere a [GitHub](https://github.com/). È possibile registrare un client predefinito per altri scopi.
* Codifica il concetto di middleware in uscita tramite la delega di gestori in `HttpClient` e offre estensioni per il middleware basato su Polly per sfruttarne i vantaggi.
* Gestisce il pooling e la durata delle istanze di `HttpClientMessageHandler` sottostanti per evitare problemi DNS comuni che si verificano quando le durate di `HttpClient` vengono gestite manualmente.
* Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.

[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisites

I progetti destinati a .NET Framework richiedono l'installazione del pacchetto NuGet [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/). I progetti destinati a .NET Core che fanno riferimento al [metapacchetto Microsoft.AspNetCore.All](xref:fundamentals/metapackage-app) sono già inclusi nel pacchetto `Microsoft.Extensions.Http`.

## <a name="consumption-patterns"></a>Modelli di consumo

`IHttpClientFactory` può essere usato in un'app in diversi modi:

* [Utilizzo di base](#basic-usage)
* [Client denominati](#named-clients)
* [Client tipizzati](#typed-clients)
* [Client generati](#generated-clients)

Nessuno di questi modi può essere considerato superiore a un altro. L'approccio migliore dipende dai vincoli dell'app.

### <a name="basic-usage"></a>Utilizzo di base

È possibile registrare `IHttpClientFactory` chiamando il metodo di estensione `AddHttpClient` in `IServiceCollection`, all'interno del metodo `Startup.ConfigureServices`.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

Dopo la registrazione, il codice può accettare un'interfaccia `IHttpClientFactory` ovunque sia possibile inserire servizi con l'[inserimento di dipendenze](xref:fundamentals/dependency-injection). `IHttpClientFactory` può essere usato per creare un'istanza di `HttpClient`:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

Questo uso di `IHttpClientFactory` è un modo efficace per effettuare il refactoring di un'app esistente. Non influisce in alcun modo sulle modalità d'uso di `HttpClient`. Nelle posizioni in cui vengono attualmente create istanze di `HttpClient`, sostituire le occorrenze con una chiamata a <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.

### <a name="named-clients"></a>Client denominati

Se un'app richiede molti usi distinti di `HttpClient`, ognuno con una configurazione diversa, un'opzione è l'uso di **client denominati**. La configurazione di un `HttpClient` denominato può essere specificata durante la registrazione in `Startup.ConfigureServices`.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

Nel codice precedente viene chiamato `AddHttpClient`, in cui viene specificato il nome *github*. Al client viene applicata una configurazione predefinita, ovvero l'indirizzo di base e due intestazioni necessari per l'uso dell'API GitHub.

Ogni volta che `CreateClient` viene chiamato, verrà creata una nuova istanza di `HttpClient` e verrà chiamata l'azione di configurazione.

Per usare un client denominato, è possibile passare un parametro di stringa a `CreateClient`. Specificare il nome del client da creare:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

Nel codice precedente non è necessario che la richiesta specifichi un nome host. Poiché viene usato l'indirizzo di base configurato per il client, è possibile passare solo il percorso.

### <a name="typed-clients"></a>Client tipizzati

Client tipizzati:

* Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.
* Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.
* La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione. Per un singolo endpoint back-end è ad esempio possibile usare un unico client tipizzato che incapsuli tutta la logica relativa all'endpoint.
* Usano l'inserimento di dipendenze e possono essere inseriti dove necessario nell'app.

Un client tipizzato accetta il parametro `HttpClient` nel proprio costruttore:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

Nel codice precedente la configurazione viene spostata nel client tipizzato. L'oggetto `HttpClient` viene esposto come una proprietà pubblica. È possibile definire metodi di API specifiche che espongono la funzionalità `HttpClient`. Il metodo `GetAspNetDocsIssues` incapsula il codice necessario per eseguire una query e analizzare gli ultimi problemi aperti in un repository GitHub.

Per registrare un client tipizzato è possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> generico all'interno di `Startup.ConfigureServices`, specificando la classe del client tipizzato:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze. Il client tipizzato può essere inserito e usato direttamente:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

Se si preferisce, è possibile specificare la configurazione di un client tipizzato durante la registrazione in `Startup.ConfigureServices`, anziché nel relativo costruttore:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

È possibile incapsulare interamente `HttpClient` all'interno di un client tipizzato. Anziché esporlo come una proprietà, è possibile specificare metodi pubblici che chiamano l'istanza di `HttpClient` internamente.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

Nel codice precedente `HttpClient` viene archiviato come un campo privato. L'accesso per effettuare chiamate esterne passa attraverso il metodo `GetRepos`.

### <a name="generated-clients"></a>Client generati

È possibile usare `IHttpClientFactory` in combinazione con altre librerie di terze parti, ad esempio [Refit](https://github.com/paulcbetts/refit). Refit è una libreria REST per .NET. Converte le API REST in interfacce live. Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.

Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a>Middleware per richieste in uscita

`HttpClient` include già il concetto di delega di gestori concatenati per le richieste HTTP in uscita. `IHttpClientFactory` semplifica la definizione dei gestori da applicare per ogni client denominato. Supporta la registrazione e il concatenamento di più gestori per creare una pipeline di middleware per le richieste in uscita. Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita. Questo modello è simile alla pipeline di middleware in ingresso in ASP.NET Core. Il modello offre un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, tra cui memorizzazione nella cache, gestione degli errori, serializzazione e registrazione.

Per creare un gestore, definire una classe che deriva da <xref:System.Net.Http.DelegatingHandler>. Eseguire l'override del metodo `SendAsync` per eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

Il codice precedente definisce un gestore di base. Verifica se un'intestazione `X-API-KEY` è stata inclusa nella richiesta. Se l'intestazione non è presente, può evitare la chiamata HTTP e restituire una risposta appropriata.

Durante la registrazione è possibile aggiungere alla configurazione uno o più gestori per un'istanza di `HttpClient`. Questa attività viene eseguita tramite metodi di estensione in <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze. Il gestore **deve** essere registrato nell'inserimento di dipendenze come servizio temporaneo, senza definizione di ambito. Se il gestore viene registrato come un servizio con ambito e i servizi da cui dipende il gestore sono eliminabili:

* I servizi del gestore potrebbero essere eliminati prima che il gestore esca dall'ambito.
* I servizi del gestore eliminati causano un errore del gestore.

Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.

È possibile registrare più gestori nell'ordine di esecuzione. Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:

* Passare i dati al gestore usando `HttpRequestMessage.Properties`.
* Usare `IHttpContextAccessor` per accedere alla richiesta corrente.
* Creare un oggetto di archiviazione `AsyncLocal` personalizzato per passare i dati.

## <a name="use-polly-based-handlers"></a>Usare gestori basati su Polly

`IHttpClientFactory` si integra con una libreria di terze parti piuttosto diffusa denominata [Polly](https://github.com/App-vNext/Polly). Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET. Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.

Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione. Le estensioni Polly:

* Supportano l'aggiunta di gestori basati su Polly ai client.
* Possono essere usate dopo l'installazione del pacchetto NuGet [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/). Il pacchetto non è incluso nel framework condiviso ASP.NET Core.

### <a name="handle-transient-faults"></a>Gestire gli errori temporanei

La maggior parte degli errori comuni si verifica quando le chiamate HTTP esterne sono temporanee. Per definire un criterio in grado di gestire gli errori temporanei è disponibile un pratico metodo di estensione denominato `AddTransientHttpErrorPolicy`. I criteri configurati con questo metodo di estensione gestiscono `HttpRequestException`, risposte HTTP 5xx e risposte HTTP 408.

L'estensione `AddTransientHttpErrorPolicy` può essere usata all'interno di `Startup.ConfigureServices`. L'estensione consente l'accesso a un oggetto `PolicyBuilder` configurato per gestire gli errori che rappresentano un possibile errore temporaneo:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

Nel codice precedente viene definito un criterio `WaitAndRetryAsync`. Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.

### <a name="dynamically-select-policies"></a>Selezionare i criteri in modo dinamico

Per aggiungere gestori basati su Polly è possibile usare altri metodi di estensione. Una di queste estensioni è `AddPolicyHandler`, che include più overload. Un overload consente l'ispezione della richiesta al momento della definizione del criterio da applicare:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi. Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.

### <a name="add-multiple-polly-handlers"></a>Aggiungere più gestori Polly

In molti casi i criteri Polly vengono annidati per offrire funzionalità avanzate:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

Nell'esempio precedente vengono aggiunti due gestori. Il primo usa l'estensione `AddTransientHttpErrorPolicy` per aggiungere criteri di ripetizione. Le richieste non riuscite vengono ritentate fino a tre volte. La seconda chiamata a `AddTransientHttpErrorPolicy` aggiunge criteri dell'interruttore di circuito. Ulteriori richieste esterne vengono bloccate per 30 secondi nel caso si verifichino cinque tentativi non riusciti consecutivi. I criteri dell'interruttore di circuito sono con stato. Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.

### <a name="add-policies-from-the-polly-registry"></a>Aggiungere criteri dal registro Polly

Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`. Per aggiungere un gestore usando un criterio del registro è disponibile un metodo di estensione:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

Nel codice precedente vengono registrati due criteri quando si aggiunge `PolicyRegistry` a `ServiceCollection`. Per usare un criterio dal registro viene usato il metodo `AddPolicyHandlerFromRegistry` passando il nome del criterio da applicare.

Altre informazioni su `IHttpClientFactory` e le integrazioni con Polly sono disponibili nel [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).

## <a name="httpclient-and-lifetime-management"></a>Gestione di HttpClient e durata

Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`. È presente un <xref:System.Net.Http.HttpMessageHandler> per ogni client denominato. La factory gestisce le durate delle istanze di `HttpMessageHandler`.

`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse. Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.

Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti. La creazione di più gestori del necessario può causare ritardi di connessione. Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito. Ciò può impedire al gestore di reagire alle modifiche DNS.

La durata del gestore predefinito è di due minuti. Il valore predefinito può essere sottoposto a override per ogni client denominato. Per eseguire l'override, chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> nell'elemento `IHttpClientBuilder` restituito al momento della creazione del client:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

L'eliminazione del client non è obbligatoria. L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>. `IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`. Le istanze di `HttpClient` possono essere considerate a livello generale come oggetti .NET che non richiedono l'eliminazione.

Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`. Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.

## <a name="logging"></a>Registrazione

I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste. Per visualizzare i messaggi di log predefiniti, abilitare il livello di informazioni appropriato nella configurazione di registrazione. La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.

La categoria di log usata per ogni client include il nome del client. Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`. I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste. Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata. Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.

La registrazione avviene anche all'interno della pipeline del gestore delle richieste. Nell'esempio *MyNamedClient*, i messaggi vengono registrati nella categoria di log `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`. Per la richiesta, ciò avviene dopo l'esecuzione di tutti gli altri gestori e immediatamente prima che la richiesta sia inviata in rete. Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.

L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline. Le modifiche possono ad esempio riguardare le intestazioni delle richieste o il codice di stato della risposta.

L'inclusione del nome del client nella categoria di log consente di filtrare i log in base a client denominati specifici, se necessario.

## <a name="configure-the-httpmessagehandler"></a>Configurare HttpMessageHandler

Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.

Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`. È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato. Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a>Usare IHttpClientFactory in un'app console

In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:

* [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http)

Nell'esempio seguente:

* <xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).
* `MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`. `HttpClient` viene usato per recuperare una pagina Web.
* `Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [Implementazione dello schema Circuit Breaker](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end
