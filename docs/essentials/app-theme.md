---
title: Xamarin.Essentials:アプリのテーマ
description: このドキュメントでは、Xamarin.Essentials の要求されたアプリ テーマに関する API について説明します。実行中のアプリに対して要求されているテーマ スタイルに関する情報が記載されています。
ms.assetid: F6F6D496-A8A9-4B9A-AF1A-370D937E5073
author: jamesmontemagno
ms.author: jamont
ms.date: 01/06/2020
ms.openlocfilehash: f322855f26d7a57acc06e97e0c97ab201c3fa586
ms.sourcegitcommit: a9280318bf7bb69e4e5744ee739e76a9cba36b28
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2020
ms.locfileid: "82047401"
---
# <a name="xamarinessentials-app-theme"></a>Xamarin.Essentials:アプリのテーマ

**RequestedTheme** API は [`AppInfo`](app-information.md) クラスの一部であり、実行中のアプリに対してシステムから要求されているテーマに関する情報が提供されます。

## <a name="get-started"></a>作業開始

[!include[](~/essentials/includes/get-started.md)]

## <a name="using-requestedtheme"></a>RequestedTheme の使用

自分のクラスに Xamarin.Essentials への参照を追加します。

```csharp
using Xamarin.Essentials;
```

## <a name="obtaining-theme-information"></a>テーマ情報の取得

要求されたアプリケーションのテーマは、次の API を使用して検出できます。

```csharp
AppTheme appTheme = AppInfo.RequestedTheme;

```

これにより、システムによって現在要求されているアプリケーションのテーマが提供されます。 その戻り値は、次のいずれかになります。

* 指定されていません。
* 淡色
* 濃色

オペレーティング システムから要求される特定のユーザー インターフェイス スタイルがない場合は、Unspecified が返されます。 この例としては、13.0 より前のバージョンの iOS を稼働しているデバイスなどがあります。


## <a name="platform-implementation-specifics"></a>プラットフォームの実装の詳細

# <a name="android"></a>[Android](#tab/android)

Android では、要求するテーマの種類を指定するために、ユーザーからの構成モードが使用されます。 これは、Android のバージョンに基づき、ユーザーが変更したり、バッテリ節約モードが有効になっているときに変更されたりする場合があります。

詳細については、公式の[ダーク テーマに関する Android ドキュメント](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme)を参照してください。


# <a name="ios"></a>[iOS](#tab/ios)

13.0 より前のバージョンの iOS では、常に Unspecified が返されます 


# <a name="uwp"></a>[UWP](#tab/uwp)

UWP アプリケーションでは、**RequestedTheme** で UWP App.xaml の設定が尊重されます。 特定のテーマに設定されている場合、Xamarin. Essentials は常にこの設定を返します。 OS の動的テーマを使用するには、アプリケーションからこのノードを削除します。その後、アプリが実行されると、Windows の設定でユーザーが設定したテーマが返されます ( **[設定] > [パーソナル化] > [色] > [Choose your default app mode]\(既定のアプリ モードを選択する\)** )。

詳細については、[UWP の要求されたテーマに関するドキュメント](https://docs.microsoft.com/uwp/api/windows.ui.xaml.application.requestedtheme)を参照してください。

--------------

## <a name="api"></a>API

- [AppInfo のソース コード](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/AppInfo)
- [AppInfo API のドキュメント](xref:Xamarin.Essentials.AppInfo)
