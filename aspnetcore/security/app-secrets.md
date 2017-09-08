---
title: "安全儲存體的過程中的程式開發 ASP.NET Core 應用程式密碼"
author: rick-anderson
description: "示範如何安全地將密碼儲存在開發期間"
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 7/14/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/app-secrets
ms.openlocfilehash: 99a1129549d6b9802315c7e5accfa22907994a41
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2017
---
# <a name="safe-storage-of-app-secrets-during-development-in-aspnet-core"></a><span data-ttu-id="e77a4-104">安全儲存體的 ASP.NET Core 在開發期間的應用程式密碼</span><span class="sxs-lookup"><span data-stu-id="e77a4-104">Safe storage of app secrets during development in ASP.NET Core</span></span>

<a name=security-app-secrets></a>

<span data-ttu-id="e77a4-105">由[Rick Anderson](https://twitter.com/RickAndMSFT)，[奧 Roth](https://github.com/danroth27)，和[Scott Addie](https://scottaddie.com)</span><span class="sxs-lookup"><span data-stu-id="e77a4-105">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Daniel Roth](https://github.com/danroth27), and [Scott Addie](https://scottaddie.com)</span></span> 

<span data-ttu-id="e77a4-106">本文件示範如何使用密碼管理員工具在開發程式碼保持機密資料。</span><span class="sxs-lookup"><span data-stu-id="e77a4-106">This document shows how you can use the Secret Manager tool in development to keep secrets out of your code.</span></span> <span data-ttu-id="e77a4-107">您應該永遠不會儲存密碼或其他機密資料來源的程式碼，而您不應該在開發和測試模式中使用實際執行的密碼最重要的一點。</span><span class="sxs-lookup"><span data-stu-id="e77a4-107">The most important point is you should never store passwords or other sensitive data in source code, and you shouldn't use production secrets in development and test mode.</span></span> <span data-ttu-id="e77a4-108">您可以改為使用[組態](../fundamentals/configuration.md)系統環境變數中讀取這些值，或使用密碼管理員儲存的值從工具。</span><span class="sxs-lookup"><span data-stu-id="e77a4-108">You can instead use the [configuration](../fundamentals/configuration.md) system to read these values from environment variables or from values stored using the Secret Manager tool.</span></span> <span data-ttu-id="e77a4-109">密碼管理員工具可協助防止機密資料被簽入原始檔控制。</span><span class="sxs-lookup"><span data-stu-id="e77a4-109">The Secret Manager tool helps prevent sensitive data from being checked into source control.</span></span> <span data-ttu-id="e77a4-110">[組態](../fundamentals/configuration.md)系統可以讀取儲存在本文中所描述的密碼管理員工具與密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-110">The [configuration](../fundamentals/configuration.md) system can read secrets stored with the Secret Manager tool described in this article.</span></span>

<span data-ttu-id="e77a4-111">密碼管理員工具只能用於開發。</span><span class="sxs-lookup"><span data-stu-id="e77a4-111">The Secret Manager tool is used only in development.</span></span> <span data-ttu-id="e77a4-112">您可以保護 Azure 測試與實際的機密資料以[Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/)組態提供者。</span><span class="sxs-lookup"><span data-stu-id="e77a4-112">You can safeguard Azure test and production secrets with the [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) configuration provider.</span></span> <span data-ttu-id="e77a4-113">請參閱[Azure 金鑰保存庫的組態提供者](https://docs.microsoft.com/aspnet/core/security/key-vault-configuration)如需詳細資訊。</span><span class="sxs-lookup"><span data-stu-id="e77a4-113">See [Azure Key Vault configuration provider](https://docs.microsoft.com/aspnet/core/security/key-vault-configuration) for more information.</span></span>

## <a name="environment-variables"></a><span data-ttu-id="e77a4-114">環境變數</span><span class="sxs-lookup"><span data-stu-id="e77a4-114">Environment variables</span></span>

<span data-ttu-id="e77a4-115">若要避免在程式碼，或在本機的組態檔中儲存應用程式密碼，您可以將密碼儲存在環境變數中。</span><span class="sxs-lookup"><span data-stu-id="e77a4-115">To avoid storing app secrets in code or in local configuration files, you store secrets in environment variables.</span></span> <span data-ttu-id="e77a4-116">您可以設定[組態](../fundamentals/configuration.md)架構，以讀取環境變數中的值，藉由呼叫`AddEnvironmentVariables`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-116">You can setup the [configuration](../fundamentals/configuration.md) framework to read values from environment variables by calling `AddEnvironmentVariables`.</span></span> <span data-ttu-id="e77a4-117">然後，您可以使用環境變數覆寫所有先前指定的設定來源的組態值。</span><span class="sxs-lookup"><span data-stu-id="e77a4-117">You can then use environment variables to override configuration values for all previously specified configuration sources.</span></span>

<span data-ttu-id="e77a4-118">例如，如果您建立新的 ASP.NET Core web 應用程式與個別使用者帳戶時，它會將新增的預設連接字串*appsettings.json*專案具有索引鍵中的檔案`DefaultConnection`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-118">For example, if you create a new ASP.NET Core web app with individual user accounts, it will add a default connection string to the *appsettings.json* file in the project with the key `DefaultConnection`.</span></span> <span data-ttu-id="e77a4-119">預設的連接字串是安裝程式使用 LocalDB，但在使用者模式下執行，而不需要密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-119">The default connection string is setup to use LocalDB, which runs in user mode and doesn't require a password.</span></span> <span data-ttu-id="e77a4-120">當您部署到測試或實際執行伺服器應用程式時，您可以覆寫`DefaultConnection`機碼值與包含測試或實際執行資料庫的連接字串 （潛在機密的認證） 的環境變數設定伺服器。</span><span class="sxs-lookup"><span data-stu-id="e77a4-120">When you deploy your application to a test or production server, you can override the `DefaultConnection` key value with an environment variable setting that contains the connection string (potentially with sensitive credentials) for a test or production database server.</span></span>

>[!WARNING]
> <span data-ttu-id="e77a4-121">環境變數通常會以純文字儲存，不會加密。</span><span class="sxs-lookup"><span data-stu-id="e77a4-121">Environment variables are generally stored in plain text and are not encrypted.</span></span> <span data-ttu-id="e77a4-122">如果電腦或處理序遭到入侵，然後可以在不受信任的合作對象來存取環境變數。</span><span class="sxs-lookup"><span data-stu-id="e77a4-122">If the machine or process is compromised, then environment variables can be accessed by untrusted parties.</span></span> <span data-ttu-id="e77a4-123">可能仍然需要其他措施以避免洩露使用者密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-123">Additional measures to prevent disclosure of user secrets may still be required.</span></span>

## <a name="secret-manager"></a><span data-ttu-id="e77a4-124">密碼管理員</span><span class="sxs-lookup"><span data-stu-id="e77a4-124">Secret Manager</span></span>

<span data-ttu-id="e77a4-125">密碼管理員工具會儲存為您專案的樹狀目錄外部的開發工作的敏感性資料。</span><span class="sxs-lookup"><span data-stu-id="e77a4-125">The Secret Manager tool stores sensitive data for development work outside of your project tree.</span></span> <span data-ttu-id="e77a4-126">密碼管理員工具是一種專案的工具，可以用來儲存的秘密資訊[.NET Core](https://microsoft.com/net/core)在開發期間的專案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-126">The Secret Manager tool is a project tool that can be used to store secrets for a [.NET Core](https://microsoft.com/net/core) project during development.</span></span> <span data-ttu-id="e77a4-127">使用密碼管理員 工具中，您可以將應用程式密碼與特定的專案產生關聯，並共用跨多個專案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-127">With the Secret Manager tool, you can associate app secrets with a specific project and share them across multiple projects.</span></span>

>[!WARNING]
> <span data-ttu-id="e77a4-128">密碼管理員工具不會加密預存機密資料，並不會被視為受信任存放區。</span><span class="sxs-lookup"><span data-stu-id="e77a4-128">The Secret Manager tool does not encrypt the stored secrets and should not be treated as a trusted store.</span></span> <span data-ttu-id="e77a4-129">它是僅限開發用途。</span><span class="sxs-lookup"><span data-stu-id="e77a4-129">It is for development purposes only.</span></span> <span data-ttu-id="e77a4-130">索引鍵和值會儲存在使用者設定檔的目錄中的 JSON 組態檔。</span><span class="sxs-lookup"><span data-stu-id="e77a4-130">The keys and values are stored in a JSON configuration file in the user profile directory.</span></span>

### <a name="visual-studio-2017-installing-the-secret-manager-tool"></a><span data-ttu-id="e77a4-131">Visual Studio 2017： 安裝 「 密碼管理員工具</span><span class="sxs-lookup"><span data-stu-id="e77a4-131">Visual Studio 2017: Installing the Secret Manager tool</span></span>

<span data-ttu-id="e77a4-132">以滑鼠右鍵按一下方案總管 中的專案，然後選取**編輯\<project_name\>.csproj**從內容功能表。</span><span class="sxs-lookup"><span data-stu-id="e77a4-132">Right-click the project in Solution Explorer, and select **Edit \<project_name\>.csproj** from the context menu.</span></span> <span data-ttu-id="e77a4-133">將反白顯示的行加入*.csproj*檔案，並將儲存到還原相關聯的 NuGet 套件：</span><span class="sxs-lookup"><span data-stu-id="e77a4-133">Add the highlighted line to the *.csproj* file, and save to restore the associated NuGet package:</span></span>

<span data-ttu-id="e77a4-134">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?highlight=21)]</span><span class="sxs-lookup"><span data-stu-id="e77a4-134">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?highlight=21)]</span></span>

<span data-ttu-id="e77a4-135">以滑鼠右鍵按一下方案總管 中的專案，然後選取**管理使用者密碼**從內容功能表。</span><span class="sxs-lookup"><span data-stu-id="e77a4-135">Right-click the project in Solution Explorer, and select **Manage User Secrets** from the context menu.</span></span> <span data-ttu-id="e77a4-136">此筆勢加入新`UserSecretsId`節點內`PropertyGroup`的*.csproj*檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-136">This gesture adds a new `UserSecretsId` node within a `PropertyGroup` of the *.csproj* file.</span></span> <span data-ttu-id="e77a4-137">它也會開啟`secrets.json`文字編輯器中的檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-137">It also opens a `secrets.json` file in the text editor.</span></span>

<span data-ttu-id="e77a4-138">將下列內容加入 `secrets.json`：</span><span class="sxs-lookup"><span data-stu-id="e77a4-138">Add the following to `secrets.json`:</span></span>

```json
{
    "MySecret": "ValueOfMySecret"
}
```

### <a name="visual-studio-2015-installing-the-secret-manager-tool"></a><span data-ttu-id="e77a4-139">Visual Studio 2015： 安裝 「 密碼管理員工具</span><span class="sxs-lookup"><span data-stu-id="e77a4-139">Visual Studio 2015: Installing the Secret Manager tool</span></span>

<span data-ttu-id="e77a4-140">開啟專案的`project.json`檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-140">Open the project's `project.json` file.</span></span> <span data-ttu-id="e77a4-141">將參考加入`Microsoft.Extensions.SecretManager.Tools`內`tools`屬性，並儲存至還原相關聯的 NuGet 套件：</span><span class="sxs-lookup"><span data-stu-id="e77a4-141">Add a reference to `Microsoft.Extensions.SecretManager.Tools` within the `tools` property, and save to restore the associated NuGet package:</span></span>

```json
"tools": {
    "Microsoft.Extensions.SecretManager.Tools": "1.0.0-preview2-final",
    "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final"
},
```

<span data-ttu-id="e77a4-142">以滑鼠右鍵按一下方案總管 中的專案，然後選取**管理使用者密碼**從內容功能表。</span><span class="sxs-lookup"><span data-stu-id="e77a4-142">Right-click the project in Solution Explorer, and select **Manage User Secrets** from the context menu.</span></span> <span data-ttu-id="e77a4-143">此筆勢加入新`userSecretsId`屬性`project.json`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-143">This gesture adds a new `userSecretsId` property to `project.json`.</span></span> <span data-ttu-id="e77a4-144">它也會開啟`secrets.json`文字編輯器中的檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-144">It also opens a `secrets.json` file in the text editor.</span></span>

<span data-ttu-id="e77a4-145">將下列內容加入 `secrets.json`：</span><span class="sxs-lookup"><span data-stu-id="e77a4-145">Add the following to `secrets.json`:</span></span>

```json
{
    "MySecret": "ValueOfMySecret"
}
```

### <a name="visual-studio-code-or-command-line-installing-the-secret-manager-tool"></a><span data-ttu-id="e77a4-146">Visual Studio 程式碼或命令列： 安裝 「 密碼管理員工具</span><span class="sxs-lookup"><span data-stu-id="e77a4-146">Visual Studio Code or Command Line: Installing the Secret Manager tool</span></span>

<span data-ttu-id="e77a4-147">新增`Microsoft.Extensions.SecretManager.Tools`至*.csproj*檔，然後執行`dotnet restore`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-147">Add `Microsoft.Extensions.SecretManager.Tools` to the *.csproj* file and run `dotnet restore`.</span></span>

<span data-ttu-id="e77a4-148">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?highlight=21)]</span><span class="sxs-lookup"><span data-stu-id="e77a4-148">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?highlight=21)]</span></span>

<span data-ttu-id="e77a4-149">測試密碼管理員工具執行下列命令：</span><span class="sxs-lookup"><span data-stu-id="e77a4-149">Test the Secret Manager tool by running the following command:</span></span>

```console
dotnet user-secrets -h
```

<span data-ttu-id="e77a4-150">密碼管理員工具會顯示使用方式、 選項和命令的說明。</span><span class="sxs-lookup"><span data-stu-id="e77a4-150">The Secret Manager tool will display usage, options and command help.</span></span>

> [!NOTE]
> <span data-ttu-id="e77a4-151">您必須在相同的目錄*.csproj*檔案來執行工具中定義*.csproj*檔案的`DotNetCliToolReference`節點。</span><span class="sxs-lookup"><span data-stu-id="e77a4-151">You must be in the same directory as the *.csproj* file to run tools defined in the *.csproj* file's `DotNetCliToolReference` nodes.</span></span>

<span data-ttu-id="e77a4-152">密碼管理員工具會依據儲存在您的使用者設定檔中的專案特定的組態設定。</span><span class="sxs-lookup"><span data-stu-id="e77a4-152">The Secret Manager tool operates on project-specific configuration settings that are stored in your user profile.</span></span> <span data-ttu-id="e77a4-153">若要使用使用者密碼，必須指定專案`UserSecretsId`值在其*.csproj*檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-153">To use user secrets, the project must specify a `UserSecretsId` value in its *.csproj* file.</span></span> <span data-ttu-id="e77a4-154">值`UserSecretsId`是任意的但通常獨有的專案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-154">The value of `UserSecretsId` is arbitrary, but is generally unique to the project.</span></span> <span data-ttu-id="e77a4-155">開發人員通常會產生的 GUID `UserSecretsId`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-155">Developers typically generate a GUID for the `UserSecretsId`.</span></span>

<span data-ttu-id="e77a4-156">新增`UserSecretsId`中的專案*.csproj*檔案：</span><span class="sxs-lookup"><span data-stu-id="e77a4-156">Add a `UserSecretsId` for your project in the *.csproj* file:</span></span>

<span data-ttu-id="e77a4-157">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?range=7-9&highlight=2)]</span><span class="sxs-lookup"><span data-stu-id="e77a4-157">[!code-xml[Main](app-secrets/sample/UserSecrets/UserSecrets.csproj?range=7-9&highlight=2)]</span></span>

<span data-ttu-id="e77a4-158">您可以使用密碼管理員工具來設定密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-158">Use the Secret Manager tool to set a secret.</span></span> <span data-ttu-id="e77a4-159">例如，在命令視窗從專案目錄中，輸入下列內容：</span><span class="sxs-lookup"><span data-stu-id="e77a4-159">For example, in a command window from the project directory, enter the following:</span></span>

```console
dotnet user-secrets set MySecret ValueOfMySecret
```

<span data-ttu-id="e77a4-160">您可以執行密碼管理員工具，從其他目錄，但您必須使用`--project`傳遞的路徑中的選項*.csproj*檔案：</span><span class="sxs-lookup"><span data-stu-id="e77a4-160">You can run the Secret Manager tool from other directories, but you must use the `--project` option to pass in the path to the *.csproj* file:</span></span>
 
```console
dotnet user-secrets set MySecret ValueOfMySecret --project c:\work\WebApp1\src\webapp1
```

<span data-ttu-id="e77a4-161">您也可以使用密碼管理員工具來列出、 移除，並清除應用程式密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-161">You can also use the Secret Manager tool to list, remove and clear app secrets.</span></span>

## <a name="accessing-user-secrets-via-configuration"></a><span data-ttu-id="e77a4-162">透過設定存取的使用者密碼</span><span class="sxs-lookup"><span data-stu-id="e77a4-162">Accessing user secrets via configuration</span></span>

<span data-ttu-id="e77a4-163">透過組態系統存取的密碼管理員密碼。</span><span class="sxs-lookup"><span data-stu-id="e77a4-163">You access Secret Manager secrets through the configuration system.</span></span> <span data-ttu-id="e77a4-164">新增`Microsoft.Extensions.Configuration.UserSecrets`封裝及執行`dotnet restore`。</span><span class="sxs-lookup"><span data-stu-id="e77a4-164">Add the `Microsoft.Extensions.Configuration.UserSecrets` package and run `dotnet restore`.</span></span>

<span data-ttu-id="e77a4-165">將使用者密碼設定來源加入`Startup`方法：</span><span class="sxs-lookup"><span data-stu-id="e77a4-165">Add the user secrets configuration source to the `Startup` method:</span></span>

<span data-ttu-id="e77a4-166">[!code-csharp[Main](app-secrets/sample/UserSecrets/Startup.cs?highlight=16-19)]</span><span class="sxs-lookup"><span data-stu-id="e77a4-166">[!code-csharp[Main](app-secrets/sample/UserSecrets/Startup.cs?highlight=16-19)]</span></span>

<span data-ttu-id="e77a4-167">您可以存取透過設定應用程式開發介面的使用者密碼：</span><span class="sxs-lookup"><span data-stu-id="e77a4-167">You can access user secrets via the configuration API:</span></span>

<span data-ttu-id="e77a4-168">[!code-csharp[Main](app-secrets/sample/UserSecrets/Startup.cs?highlight=26-29)]</span><span class="sxs-lookup"><span data-stu-id="e77a4-168">[!code-csharp[Main](app-secrets/sample/UserSecrets/Startup.cs?highlight=26-29)]</span></span>

## <a name="how-the-secret-manager-tool-works"></a><span data-ttu-id="e77a4-169">密碼管理員工具的運作方式</span><span class="sxs-lookup"><span data-stu-id="e77a4-169">How the Secret Manager tool works</span></span>

<span data-ttu-id="e77a4-170">密碼管理員工具抽象化實作詳細資料，例如值儲存的位置和方式。</span><span class="sxs-lookup"><span data-stu-id="e77a4-170">The Secret Manager tool abstracts away the implementation details, such as where and how the values are stored.</span></span> <span data-ttu-id="e77a4-171">您可以使用此工具，而不需要知道這些實作細節。</span><span class="sxs-lookup"><span data-stu-id="e77a4-171">You can use the tool without knowing these implementation details.</span></span> <span data-ttu-id="e77a4-172">在目前版本中，值會儲存在[JSON](http://json.org/)使用者設定檔的目錄中的組態檔：</span><span class="sxs-lookup"><span data-stu-id="e77a4-172">In the current version, the values are stored in a [JSON](http://json.org/) configuration file in the user profile directory:</span></span>

* <span data-ttu-id="e77a4-173">Windows:`%APPDATA%\microsoft\UserSecrets\<userSecretsId>\secrets.json`</span><span class="sxs-lookup"><span data-stu-id="e77a4-173">Windows: `%APPDATA%\microsoft\UserSecrets\<userSecretsId>\secrets.json`</span></span>

* <span data-ttu-id="e77a4-174">Linux:`~/.microsoft/usersecrets/<userSecretsId>/secrets.json`</span><span class="sxs-lookup"><span data-stu-id="e77a4-174">Linux: `~/.microsoft/usersecrets/<userSecretsId>/secrets.json`</span></span>

* <span data-ttu-id="e77a4-175">Mac:`~/.microsoft/usersecrets/<userSecretsId>/secrets.json`</span><span class="sxs-lookup"><span data-stu-id="e77a4-175">Mac: `~/.microsoft/usersecrets/<userSecretsId>/secrets.json`</span></span>

<span data-ttu-id="e77a4-176">值`userSecretsId`中指定的值是來自*.csproj*檔案。</span><span class="sxs-lookup"><span data-stu-id="e77a4-176">The value of `userSecretsId` comes from the value specified in *.csproj* file.</span></span>

<span data-ttu-id="e77a4-177">您不應該撰寫程式碼所依賴這些實作細節，可能會變更位置或使用密碼管理員 工具中，儲存的資料格式。</span><span class="sxs-lookup"><span data-stu-id="e77a4-177">You should not write code that depends on the location or format of the data saved with the Secret Manager tool, as these implementation details might change.</span></span> <span data-ttu-id="e77a4-178">例如，密碼值是目前*不*加密，但是也可能是吧。</span><span class="sxs-lookup"><span data-stu-id="e77a4-178">For example, the secret values are currently *not* encrypted today, but could be someday.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="e77a4-179">其他資源</span><span class="sxs-lookup"><span data-stu-id="e77a4-179">Additional Resources</span></span>

* [<span data-ttu-id="e77a4-180">組態</span><span class="sxs-lookup"><span data-stu-id="e77a4-180">Configuration</span></span>](../fundamentals/configuration.md)