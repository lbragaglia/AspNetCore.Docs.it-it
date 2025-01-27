---
title: Autenticazione e autorizzazione per ASP.NET Core Blazor
author: guardrex
description: Informazioni sugli scenari di autenticazione e autorizzazione per Blazor.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: security/blazor/index
ms.openlocfilehash: 85a6a32ea068e6cd00ebb71bdf7fe0bd06b77618
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391305"
---
# <a name="aspnet-core-blazor-authentication-and-authorization"></a>Autenticazione e autorizzazione per ASP.NET Core Blazor

Di [Steve Sanderson](https://github.com/SteveSandersonMS)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

ASP.NET Core supporta la configurazione e la gestione della sicurezza nelle app Blazor.

Gli scenari di sicurezza sono diversi tra le app Blazer server e blazer webassembly. Poiché le app del server Blazer vengono eseguite nel server, i controlli delle autorizzazioni sono in grado di determinare:

* Le opzioni dell'interfaccia utente presentate all'utente (ad esempio, le voci di menu disponibili per un utente).
* Le regole di accesso per le aree dell'app e i componenti.

Le app webassembly Blazer vengono eseguite sul client. L'autorizzazione viene usata *solo* per determinare quali opzioni dell'interfaccia utente visualizzare. Poiché i controlli lato client possono essere modificati o ignorati da un utente, un'app webassembly Blazer non può applicare regole di accesso alle autorizzazioni.

## <a name="authentication"></a>Authentication

Blazor usa i meccanismi di autenticazione di ASP.NET Core esistenti per stabilire l'identità dell'utente. Il meccanismo esatto dipende dal modo in cui l'app blazer è ospitata, il server blazer o il webassembly blazer.

### <a name="blazor-server-authentication"></a>Autenticazione server Blazer

Le app del server Blazer operano su una connessione in tempo reale creata con SignalR. L'[autenticazione nelle app basate su SignalR](xref:signalr/authn-and-authz) viene gestita quando viene stabilita la connessione. L'autenticazione può basarsi su un cookie o su altri bearer token.

Il modello di progetto server blazer può configurare automaticamente l'autenticazione quando viene creato il progetto.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Per creare un nuovo progetto server Blazer con un meccanismo di autenticazione, seguire le linee guida di Visual Studio disponibili nell'articolo <xref:blazor/get-started>.

Dopo aver scelto il modello **App server Blazor** nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core**, selezionare **Modifica** in **Autenticazione** .

Viene visualizzata una finestra di dialogo che offre lo stesso set di meccanismi di autenticazione disponibili per altri progetti ASP.NET Core:

* **Nessuna autenticazione**
* **Account utente individuali** &ndash; Gli account utente possono essere archiviati:
  * All'interno dell'app usando il sistema di [identità](xref:security/authentication/identity) di ASP.NET Core.
  * Con [Azure AD B2C](xref:security/authentication/azure-ad-b2c).
* **Account aziendali o dell'istituto di istruzione**
* **Autenticazione di Windows**

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Per creare un nuovo progetto server Blazer con un meccanismo di autenticazione, seguire le indicazioni Visual Studio Code nell'articolo <xref:blazor/get-started>:

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

I valori di autenticazione consentiti (`{AUTHENTICATION}`) sono indicati nella tabella seguente.

| Meccanismo di autenticazione                                                                 | Valore di`{AUTHENTICATION}` |
| ---------------------------------------------------------------------------------------- | :----------------------: |
| Nessuna autenticazione                                                                        | `None`                   |
| Utenti<br>individuali archiviati nell'app con ASP.NET Core Identity.                        | `Individual`             |
| Utenti<br>individuali archiviati in [Azure AD B2C](xref:security/authentication/azure-ad-b2c). | `IndividualB2C`          |
| Account aziendali o dell'istituto di istruzione<br>Autenticazione aziendale per un singolo tenant.            | `SingleOrg`              |
| Account aziendali o dell'istituto di istruzione<br>Autenticazione aziendale per più tenant.           | `MultiOrg`               |
| Autenticazione di Windows                                                                   | `Windows`                |

Il comando crea una cartella denominata con il valore fornito per il segnaposto `{APP NAME}` e il nome della cartella viene usato come nome dell'app. Per altre informazioni, vedere il comando [dotnet new](/dotnet/core/tools/dotnet-new) nella guida per .NET Core.

<!--

# [Visual Studio for Mac](#tab/visual-studio-mac)

1. Follow the Visual Studio for Mac guidance in the <xref:blazor/get-started> article.

1.

1.

-->

<!--
# [.NET Core CLI](#tab/netcore-cli/)

Follow the .NET Core CLI guidance in the <xref:blazor/get-started> article to create a new Blazor Server project with an authentication mechanism:

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

Permissible authentication values (`{AUTHENTICATION}`) are shown in the following table.

| Authentication mechanism                                                                 | `{AUTHENTICATION}` value |
| ---------------------------------------------------------------------------------------- | :----------------------: |
| No Authentication                                                                        | `None`                   |
| Individual<br>Users stored in the app with ASP.NET Core Identity.                        | `Individual`             |
| Individual<br>Users stored in [Azure AD B2C](xref:security/authentication/azure-ad-b2c). | `IndividualB2C`          |
| Work or School Accounts<br>Organizational authentication for a single tenant.            | `SingleOrg`              |
| Work or School Accounts<br>Organizational authentication for multiple tenants.           | `MultiOrg`               |
| Windows Authentication                                                                   | `Windows`                |

The command creates a folder named with the value provided for the `{APP NAME}` placeholder and uses the folder name as the app's name. For more information, see the [dotnet new](/dotnet/core/tools/dotnet-new) command in the .NET Core Guide.

-->

---

### <a name="blazor-webassembly-authentication"></a>Autenticazione webassembly Blazer

Nelle app webassembly blazer, i controlli di autenticazione possono essere ignorati perché tutto il codice lato client può essere modificato dagli utenti. Lo stesso vale per tutte le tecnologie per app sul lato client, tra cui i framework JavaScript SPA o le app native per qualsiasi sistema operativo.

Aggiungere un riferimento al pacchetto per [Microsoft. AspNetCore. Components. Authorization](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Authorization/) al file di progetto dell'app.

Le sezioni seguenti illustrano l'implementazione di un servizio `AuthenticationStateProvider` personalizzato per le app webassembly blazer.

## <a name="authenticationstateprovider-service"></a>Servizio AuthenticationStateProvider

Le app del server Blazer includono un servizio `AuthenticationStateProvider` incorporato che ottiene i dati sullo stato di autenticazione da `HttpContext.User` di ASP.NET Core. Questo è il modo in cui lo stato di autenticazione si integra con i meccanismi di autenticazione sul lato server ASP.NET Core esistenti.

`AuthenticationStateProvider` è il servizio sottostante usato dal componente `AuthorizeView` e dal componente `CascadingAuthenticationState` per ottenere lo stato di autenticazione.

In genere non si usa `AuthenticationStateProvider` direttamente. Usare gli approcci con il [componente AuthorizeView](#authorizeview-component) oppure [Task<AuthenticationState>](#expose-the-authentication-state-as-a-cascading-parameter) descritti più avanti in questo articolo. Lo svantaggio principale dell'uso diretto di `AuthenticationStateProvider` è che il componente non riceve alcuna notifica automaticamente se i dati relativi allo stato di autenticazione sottostanti cambiano.

Il servizio `AuthenticationStateProvider` può fornire i dati <xref:System.Security.Claims.ClaimsPrincipal> dell'utente corrente, come illustrato nell'esempio seguente:

```cshtml
@page "/"
@using Microsoft.AspNetCore.Components.Authorization
@inject AuthenticationStateProvider AuthenticationStateProvider

<button @onclick="@LogUsername">Write user info to console</button>

@code {
    private async Task LogUsername()
    {
        var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            Console.WriteLine($"{user.Identity.Name} is authenticated.");
        }
        else
        {
            Console.WriteLine("The user is NOT authenticated.");
        }
    }
}
```

Se `user.Identity.IsAuthenticated` è `true` e perché l'utente è un <xref:System.Security.Claims.ClaimsPrincipal>, è possibile enumerare le attestazioni e valutare l'appartenenza ai ruoli.

Per altre informazioni sull'inserimento delle dipendenze e sui servizi, vedere <xref:blazor/dependency-injection> e <xref:fundamentals/dependency-injection>.

## <a name="implement-a-custom-authenticationstateprovider"></a>Implementare un AuthenticationStateProvider personalizzato

Se si sta compilando un'app webassembly blazer o se la specifica dell'app richiede assolutamente un provider personalizzato, implementare un provider ed eseguire l'override di `GetAuthenticationStateAsync`:

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

namespace BlazorSample.Services
{
    public class CustomAuthStateProvider : AuthenticationStateProvider
    {
        public override Task<AuthenticationState> GetAuthenticationStateAsync()
        {
            var identity = new ClaimsIdentity(new[]
            {
                new Claim(ClaimTypes.Name, "mrfibuli"),
            }, "Fake authentication type");

            var user = new ClaimsPrincipal(identity);

            return Task.FromResult(new AuthenticationState(user));
        }
    }
}
```

Il servizio `CustomAuthStateProvider` è registrato in `Startup.ConfigureServices`:

```csharp
// using Microsoft.AspNetCore.Components.Authorization;
// using BlazorSample.Services;

services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
```

Usando `CustomAuthStateProvider` tutti gli utenti vengono autenticati con il nome utente `mrfibuli`.

## <a name="expose-the-authentication-state-as-a-cascading-parameter"></a>Esporre lo stato di autenticazione come un parametro a catena

Se i dati dello stato di autenticazione sono necessari per la logica procedurale, ad esempio quando si esegue un'azione attivata dall'utente, ottenere i dati dello stato di autenticazione definendo un parametro a catena di tipo `Task<AuthenticationState>`:

```cshtml
@page "/"

<button @onclick="@LogUsername">Log username</button>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async Task LogUsername()
    {
        var authState = await authenticationStateTask;
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            Console.WriteLine($"{user.Identity.Name} is authenticated.");
        }
        else
        {
            Console.WriteLine("The user is NOT authenticated.");
        }
    }
}
```

> [!NOTE]
> In un componente dell'app webassembly Blazer aggiungere lo spazio dei nomi `Microsoft.AspNetCore.Components.Authorization` (`@using Microsoft.AspNetCore.Components.Authorization`).

Se `user.Identity.IsAuthenticated` è `true`, è possibile enumerare le attestazioni e valutare l'appartenenza ai ruoli.

Configurare il parametro di propagazione `Task<AuthenticationState>` usando i componenti `AuthorizeRouteView` e `CascadingAuthenticationState`:

```cshtml
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <CascadingAuthenticationState>
            <LayoutView Layout="@typeof(MainLayout)">
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </CascadingAuthenticationState>
    </NotFound>
</Router>
```

## <a name="authorization"></a>Autorizzazione

Dopo l'autenticazione di un utente, vengono applicate le regole di *autorizzazione* per controllare le operazioni consentite per un utente.

L'accesso viene in genere concesso o negato in base alle condizioni seguenti:

* L'utente è autenticato (ha eseguito l'accesso).
* L'utente è incluso in un *ruolo*.
* L'utente ha un'*attestazione*.
* I *criteri* sono soddisfatti.

Questi concetti sono uguali a quelli validi per un'app ASP.NET Core MVC o Razor Pages. Per altre informazioni sulla sicurezza di ASP.NET Core, vedere gli articoli in [Sicurezza e identità per ASP.NET Core](xref:security/index).

## <a name="authorizeview-component"></a>Componente AuthorizeView

Il componente `AuthorizeView` visualizza in modo selettivo l'interfaccia utente a seconda del fatto che l'utente sia autorizzato a visualizzarla. Questo approccio è utile quando è necessario solo *visualizzare* dati per l'utente e non occorre usare l'identità dell'utente nella logica procedurale.

Il componente espone una variabile `context` di tipo `AuthenticationState`, che è possibile usare per accedere alle informazioni sull'utente connesso:

```cshtml
<AuthorizeView>
    <h1>Hello, @context.User.Identity.Name!</h1>
    <p>You can only see this content if you're authenticated.</p>
</AuthorizeView>
```

Se l'utente non è autenticato, è anche possibile fornire un contenuto diverso per la visualizzazione:

```cshtml
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authenticated.</p>
    </Authorized>
    <NotAuthorized>
        <h1>Authentication Failure!</h1>
        <p>You're not signed in.</p>
    </NotAuthorized>
</AuthorizeView>
```

Il contenuto dei tag `<Authorized>` e `<NotAuthorized>` può includere elementi arbitrari, ad esempio altri componenti interattivi.

Le condizioni di autorizzazione, ad esempio i ruoli o i criteri che consentono di controllare le opzioni dell'interfaccia utente o l'accesso, sono presentate nella sezione [Autorizzazione](#authorization).

Se non sono specificate condizioni di autorizzazione, `AuthorizeView` usa criteri predefiniti e considera:

* Gli utenti autenticati (che hanno eseguito l'accesso) come autorizzati.
* Gli utenti non autenticati (disconnessi) come non autorizzati.

### <a name="role-based-and-policy-based-authorization"></a>Autorizzazione basata sui ruoli e basata sui criteri

Il componente `AuthorizeView` supporta l'autorizzazione *basata sui ruoli* oppure *basata sui criteri*.

Per l'autorizzazione basata sui ruoli, usare il parametro `Roles`:

```cshtml
<AuthorizeView Roles="admin, superuser">
    <p>You can only see this if you're an admin or superuser.</p>
</AuthorizeView>
```

Per ulteriori informazioni, vedere <xref:security/authorization/roles>.

Per l'autorizzazione basata sui criteri, usare il parametro `Policy`:

```cshtml
<AuthorizeView Policy="content-editor">
    <p>You can only see this if you satisfy the "content-editor" policy.</p>
</AuthorizeView>
```

L'autorizzazione basata sulle attestazioni è un caso speciale di autorizzazione basata su criteri. Ad esempio, è possibile definire un criterio che richiede che gli utenti abbiano una determinata attestazione. Per ulteriori informazioni, vedere <xref:security/authorization/policies>.

Queste API possono essere usate nelle app del server blazer o del webassembly blazer.

Se non si specifica `Roles` o `Policy`, `AuthorizeView` usa i criteri predefiniti.

### <a name="content-displayed-during-asynchronous-authentication"></a>Contenuto visualizzato durante l'autenticazione asincrona

Blazor consente di determinare lo stato di autenticazione *in modo asincrono*. Lo scenario principale per questo approccio è nelle app webassembly di blazer che effettuano una richiesta a un endpoint esterno per l'autenticazione.

Mentre è in corso l'autenticazione `AuthorizeView` non visualizza alcun contenuto per impostazione predefinita. Per visualizzare contenuto durante l'autenticazione, usare l'elemento `<Authorizing>`:

```cshtml
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authenticated.</p>
    </Authorized>
    <Authorizing>
        <h1>Authentication in progress</h1>
        <p>You can only see this content while authentication is in progress.</p>
    </Authorizing>
</AuthorizeView>
```

Questo approccio non è in genere applicabile alle app del server blazer. Le app del server Blazer conoscono lo stato di autenticazione non appena viene stabilito lo stato. il contenuto `Authorizing` può essere fornito in un componente `AuthorizeView` dell'app del server blazer, ma il contenuto non viene mai visualizzato.

## <a name="authorize-attribute"></a>Attributo [Authorize]

È possibile usare l'attributo `[Authorize]` nei componenti Razor:

```cshtml
@page "/"
@attribute [Authorize]

You can only see this if you're signed in.
```

> [!NOTE]
> In un componente dell'app webassembly Blazer aggiungere lo spazio dei nomi `Microsoft.AspNetCore.Authorization` (`@using Microsoft.AspNetCore.Authorization`) agli esempi in questa sezione.

> [!IMPORTANT]
> Usare `[Authorize]` solo per i componenti `@page` raggiunti tramite il componente Router di Blazor. L'autorizzazione viene eseguita solo come un aspetto del routing e *non* per i componenti figlio di cui viene eseguito il rendering all'interno di una pagina. Per autorizzare la visualizzazione di parti specifiche all'interno di una pagina, usare invece `AuthorizeView`.

L'attributo `[Authorize]` supporta anche l'autorizzazione basata sui ruoli o basata sui criteri. Per l'autorizzazione basata sui ruoli, usare il parametro `Roles`:

```cshtml
@page "/"
@attribute [Authorize(Roles = "admin, superuser")]

<p>You can only see this if you're in the 'admin' or 'superuser' role.</p>
```

Per l'autorizzazione basata sui criteri, usare il parametro `Policy`:

```cshtml
@page "/"
@attribute [Authorize(Policy = "content-editor")]

<p>You can only see this if you satisfy the 'content-editor' policy.</p>
```

Se non si specifica `Roles` o `Policy`, `[Authorize]` usa i criteri predefiniti, che per impostazione predefinita considerano:

* Gli utenti autenticati (che hanno eseguito l'accesso) come autorizzati.
* Gli utenti non autenticati (disconnessi) come non autorizzati.

## <a name="customize-unauthorized-content-with-the-router-component"></a>Personalizzare il contenuto non autorizzato con il componente Router

Il componente `Router`, insieme al componente `AuthorizeRouteView`, consente all'app di specificare contenuto personalizzato se:

* Non viene trovato contenuto.
* L'utente non supera una condizione `[Authorize]` applicata al componente. L'attributo `[Authorize]` viene presentato nella sezione [Attributo [Authorize]](#authorize-attribute).
* L'autenticazione asincrona è in corso.

Nel modello di progetto server Blazer predefinito, il file *app. Razor* Mostra come impostare il contenuto personalizzato:

```cshtml
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
            <NotAuthorized>
                <h1>Sorry</h1>
                <p>You're not authorized to reach this page.</p>
                <p>You may need to log in as a different user.</p>
            </NotAuthorized>
            <Authorizing>
                <h1>Authentication in progress</h1>
                <p>Only visible while authentication is in progress.</p>
            </Authorizing>
        </AuthorizeRouteView>
    </Found>
    <NotFound>
        <CascadingAuthenticationState>
            <LayoutView Layout="@typeof(MainLayout)">
                <h1>Sorry</h1>
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </CascadingAuthenticationState>
    </NotFound>
</Router>
```

Il contenuto dei tag `<NotFound>`, `<NotAuthorized>` e `<Authorizing>` può includere elementi arbitrari, ad esempio altri componenti interattivi.

Se l'elemento `<NotAuthorized>` non è specificato, il `AuthorizeRouteView` utilizza il seguente messaggio di fallback:

```html
Not authorized.
```

## <a name="notification-about-authentication-state-changes"></a>Notifica per le modifiche dello stato di autenticazione

Se l'app determina che i dati dello stato di autenticazione sottostante sono stati modificati (ad esempio, perché l'utente si è disconnesso o un altro utente ha modificato i relativi ruoli), un `AuthenticationStateProvider` personalizzato può richiamare facoltativamente il metodo `NotifyAuthenticationStateChanged` sulla classe di base `AuthenticationStateProvider`. Viene così inviata notifica ai consumer dei dati di stato di autenticazione (ad esempio, `AuthorizeView`) di eseguire nuovamente il rendering usando i nuovi dati.

## <a name="procedural-logic"></a>Logica procedurale

Se l'app deve controllare le regole di autorizzazione come parte della logica procedurale, usare un parametro a catena di tipo `Task<AuthenticationState>` per ottenere il <xref:System.Security.Claims.ClaimsPrincipal> dell'utente. `Task<AuthenticationState>` può essere combinato con altri servizi, ad esempio `IAuthorizationService`, per valutare i criteri.

```cshtml
@inject IAuthorizationService AuthorizationService

<button @onclick="@DoSomething">Do something important</button>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async Task DoSomething()
    {
        var user = (await authenticationStateTask).User;

        if (user.Identity.IsAuthenticated)
        {
            // Perform an action only available to authenticated (signed-in) users.
        }

        if (user.IsInRole("admin"))
        {
            // Perform an action only available to users in the 'admin' role.
        }

        if ((await AuthorizationService.AuthorizeAsync(user, "content-editor"))
            .Succeeded)
        {
            // Perform an action only available to users satisfying the 
            // 'content-editor' policy.
        }
    }
}
```

> [!NOTE]
> In un componente dell'app webassembly Blazer aggiungere gli spazi dei nomi `Microsoft.AspNetCore.Authorization` e `Microsoft.AspNetCore.Components.Authorization`:
>
> ```cshtml
> @using Microsoft.AspNetCore.Authorization
> @using Microsoft.AspNetCore.Components.Authorization
> ```

## <a name="authorization-in-blazor-webassembly-apps"></a>Autorizzazione nelle app webassembly Blazer

Nelle app webassembly blazer, i controlli di autorizzazione possono essere ignorati perché tutto il codice lato client può essere modificato dagli utenti. Lo stesso vale per tutte le tecnologie per app sul lato client, tra cui i framework JavaScript SPA o le app native per qualsiasi sistema operativo.

**Eseguire sempre i controlli di autorizzazione nel server all'interno degli eventuali endpoint dell'API a cui accede l'app sul lato client.**

## <a name="troubleshoot-errors"></a>Risolvere gli errori

Errori comuni:

* **Per l'autorizzazione è necessario un parametro di propagazione di tipo Task @ no__t-1AuthenticationState >. Per fornire questo, provare a usare CascadingAuthenticationState.**

* **Valore `null` ricevuto per `authenticationStateTask`**

È probabile che il progetto non sia stato creato usando un modello di server Blazer con autenticazione abilitata. Eseguire il wrapping di un `<CascadingAuthenticationState>` in una parte dell'albero dell'interfaccia utente, ad esempio in *App.razor* come indicato di seguito:

```cshtml
<CascadingAuthenticationState>
    <Router AppAssembly="typeof(Startup).Assembly">
        ...
    </Router>
</CascadingAuthenticationState>
```

`CascadingAuthenticationState` fornisce il parametro a catena `Task<AuthenticationState>`, ricevuto a sua volta dal servizio di inserimento delle dipendenze `AuthenticationStateProvider` sottostante.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:security/index>
* <xref:security/blazor/server>
* <xref:security/authentication/windowsauth>
