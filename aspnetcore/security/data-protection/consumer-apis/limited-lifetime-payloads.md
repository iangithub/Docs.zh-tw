---
title: "限制受保護的裝載的存留期"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 000175e2-10fc-43dd-bfc2-51e004b97b44
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/limited-lifetime-payloads
ms.openlocfilehash: 4ff13803b328c1e9dd2934c38c88b43f5798de03
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2017
---
# <a name="limiting-the-lifetime-of-protected-payloads"></a>限制受保護的裝載的存留期

沒有應用程式開發人員要建立一段時間之後過期的受保護的內容的案例。 例如，受保護的內容可能代表一小時只能為有效的密碼重設語彙基元。 很可能建立自己裝載格式，其中包含內嵌的到期日，開發人員和進階開發人員可能會想要這樣做，但對於大部分的開發人員管理這些到期日的成長繁瑣。

若要簡化此我們的開發人員對象，封裝 Microsoft.AspNetCore.DataProtection.Extensions 包含公用程式 Api 建立在一段時間後自動過期的裝載。 這些 Api 的停止回應開 ITimeLimitedDataProtector 型別。

## <a name="api-usage"></a>API 使用方式

ITimeLimitedDataProtector 介面是保護和設定時間限制 / 自我裝載設定為已過期的核心介面。 若要建立 ITimeLimitedDataProtector 的執行個體，您必須先執行個體的一般[IDataProtector](overview.md)特定目的所建構。 一旦可以使用 IDataProtector 執行個體，呼叫 IDataProtector.ToTimeLimitedDataProtector 擴充方法來取得回保護裝置具有內建到期功能。

ITimeLimitedDataProtector 會公開下列 API 介面和擴充方法：

* CreateProtector （字串用途）： ITimeLimitedDataProtector 此 API 是類似於現有 IDataProtectionProvider.CreateProtector，它可以用來建立[用途鏈結](purpose-strings.md)從根時間限制保護裝置。

* 保護 （位元組 [純文字、 DateTimeOffset 到期）： byte]

* 保護 （位元組 [純文字、 TimeSpan 存留時間）： byte]

* 保護 （位元組 [純文字）： byte]

* 保護 （字串純文字、 DateTimeOffset 到期）： 字串

* 保護 （字串純文字、 TimeSpan 存留時間）： 字串

* 保護 （字串純文字）： 字串

除了核心保護方法需要純文字，還有新的多載可讓您指定的內容到期日。 可以指定到期日，為的絕對日期 （透過 DateTimeOffset) 或相對的時間 （從目前的系統時間，透過 TimeSpan）。 如果呼叫時，才會到期的多載，承載會假設永遠不會為過期。

* 取消保護 (位元組 [] protectedData，DateTimeOffset 到期時): byte]

* 取消保護 ([] protectedData 位元組): byte]

* 取消保護 (DateTimeOffset 到期時，字串 protectedData): 字串

* 取消保護 (字串 protectedData): 字串

Unprotect 方法會傳回原始未受保護的資料。 如果裝載尚未尚未過期，絕對期限會傳回做為選擇性的 out 參數，以及原始未受保護的資料。 如果裝載已過期，取消保護方法的所有多載會擲回 CryptographicException。

>[!WARNING]
> 不建議您先使用這些 Api 來保護裝載需要長期或無限期的持續性。 「 可我負擔的月份是永久無法復原受保護的內容嗎？ 」 可以做為最佳經驗法則;如果答案是沒有然後開發人員應該考慮替代的 Api。

使用下列的範例[非 DI 程式碼路徑](../configuration/non-di-scenarios.md)具現化的資料保護系統。 若要執行此範例，請確認您有第一次加入 Microsoft.AspNetCore.DataProtection.Extensions 封裝的參考。

[!code-none[Main](limited-lifetime-payloads/samples/limitedlifetimepayloads.cs)]
