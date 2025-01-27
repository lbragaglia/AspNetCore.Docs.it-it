---
title: Autenticazione dei provider Facebook, Google ed esterni in ASP.NET Core
author: rick-anderson
description: Questa esercitazione illustra come compilare un'app ASP.NET Core usando OAuth 2,0 con provider di autenticazione esterni.
ms.author: riande
ms.custom: mvc
ms.date: 10/21/2019
uid: security/authentication/social/index
ms.openlocfilehash: 627ca483d60514d85e38c0e346ff5aef64ad9fee
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034299"
---
# <a name="facebook-google-and-external-provider-authentication-in-aspnet-core"></a>Autenticazione dei provider Facebook, Google ed esterni in ASP.NET Core

Da [Valeriy Novytskyy](https://github.com/01binary) e [Rick Anderson](https://twitter.com/RickAndMSFT)

Questa esercitazione illustra come compilare un'app ASP.NET Core 3,0 che consente agli utenti di accedere usando OAuth 2,0 con le credenziali dei provider di autenticazione esterni.

I provider [Facebook](xref:security/authentication/facebook-logins), [Twitter](xref:security/authentication/twitter-logins), [Google](xref:security/authentication/google-logins)e [Microsoft](xref:security/authentication/microsoft-logins) sono descritti nelle sezioni seguenti e usano il progetto iniziale creato in questo articolo. Altri provider sono disponibili nei pacchetti di terze parti, ad esempio [AspNet.Security.OAuth.Providers](https://github.com/aspnet-contrib/AspNet.Security.OAuth.Providers) e [AspNet.Security.OpenId.Providers](https://github.com/aspnet-contrib/AspNet.Security.OpenId.Providers).

Il fatto di consentire agli utenti l'accesso con le proprie credenziali esistenti:

* È pratico per gli utenti.
* Trasferisce a terzi molti aspetti complicati del processo di accesso.

Per degli esempi di come gli account di accesso ai social possano risultare utili per la conversione del traffico e del cliente, vedere dei case study da [Facebook](https://www.facebook.com/unsupportedbrowser) e [Twitter](https://dev.twitter.com/resources/case-studies).

## <a name="create-a-new-aspnet-core-project"></a>Creare un nuovo progetto ASP.NET Core

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Creare un nuovo progetto.
* Selezionare **Applicazione Web ASP.NET Core** e **Avanti**.
* Specificare un **Nome progetto** e confermare o modificare il **Percorso**. Scegliere **Crea**.
* Selezionare **ASP.NET Core 3,0** nell'elenco a discesa, quindi selezionare **applicazione Web**.
* In **Autenticazione** selezionare **Modifica** e impostare l'autenticazione su **Account utente individuali**. Scegliere **OK**.
* Nella finestra **Crea una nuova applicazione Web ASP.NET Core** selezionare **Crea**.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code/Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

* Aprire il terminale.  Per Visual Studio Code è possibile aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).

* Cambiare directory (`cd`) e passare alla cartella che conterrà il progetto.

* Per Windows, eseguire il comando seguente:

  ```dotnetcli
  dotnet new webapp -o WebApp1 -au Individual -uld
  ```

  Per macOS e Linux, eseguire il comando seguente:

  ```dotnetcli
  dotnet new webapp -o WebApp1 -au Individual
  ```

  * Il comando `dotnet new` crea un nuovo progetto Razor Pages nella cartella *WebApp1*.
  * `-au Individual` crea il codice per l'autenticazione individuale.
  * `-uld` usa il database locale, una versione leggera di SQL Server Express per Windows. Omettere `-uld` per usare SQLite.
  * Il comando `code` apre la cartella *WebApp1* in una nuova istanza di Visual Studio Code.

---

## <a name="apply-migrations"></a>Applicare le migrazioni

* Eseguire l'app e selezionare il collegamento **Registra**.
* Immettere l'indirizzo di posta elettronica e la password per il nuovo account, quindi selezionare **Registra**.
* Seguire le istruzioni per applicare le migrazioni.

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

## <a name="use-secretmanager-to-store-tokens-assigned-by-login-providers"></a>Usare SecretManager per archiviare i token assegnati dai provider di accesso

I provider di accesso ai social assegnano i token **ID applicazione** e **Segreto dell'applicazione** durante il processo di registrazione. La denominazione esatta dei token varia a seconda del provider. Questi token rappresentano le credenziali usate dall'app per accedere alle API. I token costituiscono i "segreti" che è possibile collegare alla configurazione dell'app usando [Secret Manager](xref:security/app-secrets#secret-manager). Secret Manager è un'alternativa che offre maggior sicurezza rispetto alla memorizzazione dei token in un file di configurazione, ad esempio *appsettings.json*.

> [!IMPORTANT]
> Secret Manager è progettato esclusivamente per lo sviluppo. È possibile memorizzare e proteggere i segreti relativi al test e alla produzione di Azure usando il [provider di configurazione di Azure Key Vault](xref:security/key-vault-configuration).

Seguire la procedura descritta nell'argomento [Archiviazione sicura di segreti dell'app durante lo sviluppo di ASP.NET Core](xref:security/app-secrets) per archiviare i token assegnati dai singoli provider di accesso elencati di seguito.

## <a name="setup-login-providers-required-by-your-application"></a>Impostare i provider di accesso necessari per l'applicazione

Usare gli argomenti seguenti per configurare l'applicazione per usare i rispettivi provider:

* Istruzioni di [Facebook](xref:security/authentication/facebook-logins)
* Istruzioni di [Twitter](xref:security/authentication/twitter-logins)
* Istruzioni di [Google](xref:security/authentication/google-logins)
* Istruzioni di [Microsoft](xref:security/authentication/microsoft-logins)
* Istruzioni di [altri provider](xref:security/authentication/otherlogins)

[!INCLUDE[](includes/chain-auth-providers.md)]

## <a name="optionally-set-password"></a>Impostare facoltativamente la password

Quando ci si registra con un provider di accesso esterno, non si dispone di una password registrata con l'app. Questo evita la creazione e la memorizzazione di una password per il sito, ma crea anche una dipendenza dal provider di accesso esterno. Se il provider di accesso esterno non è disponibile, non si potrà accedere al sito web.

Per creare una password e accedere usando la posta elettronica impostata durante il processo di accesso con provider esterni:

* Selezionare il collegamento **Salve &lt;alias di posta elettronica&gt;** nell'angolo superiore destro per passare alla vista **Gestione**.

![Vista Gestione dell'applicazione Web](index/_static/pass1a.png)

* Selezionare **Crea**.

![Impostare la pagina della password](index/_static/pass2a.png)

* Impostare una password valida in modo da usarla per accedere con la posta elettronica.

## <a name="next-steps"></a>Passaggi successivi

* Questo articolo ha introdotto l'autenticazione esterna e ha illustrato i prerequisiti necessari per aggiungere gli account di accesso esterni all'app di ASP.NET Core.

* Pagine di riferimento specifico del provider per configurare gli account di accesso per i provider richiesti dall'app.

* Può essere opportuno salvare in modo permanente dati aggiuntivi sull'utente e il relativo accesso e i token di aggiornamento. Per ulteriori informazioni, vedere <xref:security/authentication/social/additional-claims>.
