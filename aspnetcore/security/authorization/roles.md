---
title: Autorizzazione basata sui ruoli in ASP.NET Core
author: rick-anderson
description: Informazioni su come limitare l'accesso ASP.NET Core controller e le azioni passando i ruoli all'attributo di autorizzazione.
ms.author: riande
ms.date: 10/14/2016
uid: security/authorization/roles
ms.openlocfilehash: 28aa3df6aa661d0b762df78fe611cd827af43f75
ms.sourcegitcommit: 6628cd23793b66e4ce88788db641a5bbf470c3c1
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 11/06/2019
ms.locfileid: "73660043"
---
# <a name="role-based-authorization-in-aspnet-core"></a>Autorizzazione basata sui ruoli in ASP.NET Core

<a name="security-authorization-role-based"></a>

Quando viene creata un'identità, può appartenere a uno o più ruoli. Ad esempio, Tracy può appartenere ai ruoli amministratore e utente mentre Scott può appartenere solo al ruolo utente. La modalità di creazione e gestione di questi ruoli dipende dall'archivio di backup del processo di autorizzazione. I ruoli vengono esposti allo sviluppatore tramite il metodo [IsInRole](/dotnet/api/system.security.principal.genericprincipal.isinrole) nella classe [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal) .

## <a name="adding-role-checks"></a>Aggiunta di controlli di ruolo

I controlli delle autorizzazioni in base al ruolo sono dichiarativi&mdash;lo sviluppatore li incorpora nel codice, a fronte di un controller o di un'azione all'interno di un controller, specificando i ruoli di cui l'utente corrente deve essere membro per accedere alla risorsa richiesta.

Il codice seguente, ad esempio, limita l'accesso a qualsiasi azione sul `AdministrationController` agli utenti che sono membri del ruolo `Administrator`:

```csharp
[Authorize(Roles = "Administrator")]
public class AdministrationController : Controller
{
}
```

È possibile specificare più ruoli come un elenco delimitato da virgole:

```csharp
[Authorize(Roles = "HRManager,Finance")]
public class SalaryController : Controller
{
}
```

Questo controller sarebbe accessibile solo agli utenti che sono membri del ruolo `HRManager` o del ruolo `Finance`.

Se si applicano più attributi, un utente che esegue l'accesso deve essere un membro di tutti i ruoli specificati. per l'esempio seguente è necessario che un utente sia membro del ruolo `PowerUser` e `ControlPanelUser`.

```csharp
[Authorize(Roles = "PowerUser")]
[Authorize(Roles = "ControlPanelUser")]
public class ControlPanelController : Controller
{
}
```

È possibile limitare ulteriormente l'accesso applicando attributi di autorizzazione del ruolo aggiuntivi a livello di azione:

```csharp
[Authorize(Roles = "Administrator, PowerUser")]
public class ControlPanelController : Controller
{
    public ActionResult SetTime()
    {
    }

    [Authorize(Roles = "Administrator")]
    public ActionResult ShutDown()
    {
    }
}
```

Nel frammento di codice precedente i membri del ruolo `Administrator` o del ruolo `PowerUser` possono accedere al controller e all'azione `SetTime`, ma solo i membri del ruolo `Administrator` possono accedere all'azione `ShutDown`.

È anche possibile bloccare un controller, ma consentire l'accesso anonimo e non autenticato a singole azioni.

```csharp
[Authorize]
public class ControlPanelController : Controller
{
    public ActionResult SetTime()
    {
    }

    [AllowAnonymous]
    public ActionResult Login()
    {
    }
}
```

::: moniker range=">= aspnetcore-2.0"

Per Razor Pages, il `AuthorizeAttribute` può essere applicato da uno dei seguenti:

* Utilizzando una [convenzione](xref:razor-pages/razor-pages-conventions#page-model-action-conventions)o
* Applicazione dell'`AuthorizeAttribute` all'istanza di `PageModel`:

```csharp
[Authorize(Policy = "RequireAdministratorRole")]
public class UpdateModel : PageModel
{
    public ActionResult OnPost()
    {
    }
}
```

> [!IMPORTANT]
> Gli attributi di filtro, inclusi `AuthorizeAttribute`, possono essere applicati solo a PageModel e non possono essere applicati a metodi specifici del gestore di pagine.
::: moniker-end

<a name="security-authorization-role-policy"></a>

## <a name="policy-based-role-checks"></a>Controlli dei ruoli basati su criteri

I requisiti del ruolo possono essere espressi anche usando la nuova sintassi dei criteri, in cui uno sviluppatore registra un criterio all'avvio come parte della configurazione del servizio di autorizzazione. Questa situazione si verifica in genere in `ConfigureServices()` nel file *Startup.cs* .

::: moniker range=">= aspnetcore-3.0"
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddRazorPages();

    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireAdministratorRole",
             policy => policy.RequireRole("Administrator"));
    });
}
```
::: moniker-end

::: moniker range="< aspnetcore-3.0"
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireAdministratorRole",
             policy => policy.RequireRole("Administrator"));
    });
}
```
::: moniker-end

I criteri vengono applicati utilizzando la proprietà `Policy` sull'attributo `AuthorizeAttribute`:

```csharp
[Authorize(Policy = "RequireAdministratorRole")]
public IActionResult Shutdown()
{
    return View();
}
```

Se si desidera specificare più ruoli consentiti in un requisito, è possibile specificarli come parametri per il metodo `RequireRole`:

```csharp
options.AddPolicy("ElevatedRights", policy =>
                  policy.RequireRole("Administrator", "PowerUser", "BackupAdministrator"));
```

Questo esempio autorizza gli utenti che appartengono ai ruoli `Administrator`, `PowerUser` o `BackupAdministrator`.

### <a name="add-role-services-to-identity"></a>Aggiungere servizi ruolo all'identità

Aggiungere [Aggiungi ruoli](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) per aggiungere servizi ruolo:

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](roles/samples/3_0/Startup.cs?name=snippet&highlight=7)]
::: moniker-end

::: moniker range="< aspnetcore-3.0"
[!code-csharp[](roles/samples/2_2/Startup.cs?name=snippet&highlight=7)]
::: moniker-end

