---
title: Xamarin.Essentials:アクセス許可
description: このドキュメントでは、Xamarin.Essentials の Permissions クラスについて説明します。これにより、実行時のアクセス許可を確認および要求することができます。
ms.assetid: 34062D84-3E55-4AF7-A688-8551068B1E57
author: jamesmontemagno
ms.author: jamont
ms.date: 01/06/2020
ms.openlocfilehash: 21f2079ace4adae6fd84d89426e5d66692af2a0a
ms.sourcegitcommit: b0ea451e18504e6267b896732dd26df64ddfa843
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/13/2020
ms.locfileid: "78289807"
---
# <a name="xamarinessentials-permissions"></a>Xamarin.Essentials:アクセス許可

**Permissions** クラスでは、実行時のアクセス許可を確認および要求する機能が提供されます。

## <a name="get-started"></a>作業開始

[!include[](~/essentials/includes/get-started.md)]

## <a name="using-permissions"></a>Permissions の使用

自分のクラスに Xamarin.Essentials への参照を追加します。

```csharp
using Xamarin.Essentials;
```

## <a name="checking-permissions"></a>アクセス許可の確認

アクセス許可の現在の状態を確認するには、状態を取得する特定のアクセス許可を指定して `CheckStatusAsync` メソッドを使用します。

```csharp
var status = await Permissions.CheckStatusAsync<Permissions.LocationWhenInUse>();
```

必要なアクセス許可が宣言されていない場合は、`PermissionException` がスローされます。

## <a name="requesting-permissions"></a>権限の要求

ユーザーからのアクセス許可を要求するには、要求する特定のアクセス許可を指定して `RequestAsync` メソッドを使用します。 ユーザーが以前にアクセス許可を付与していて、まだ取り消していない場合は、このメソッドによって直ちに `Granted` が返され、ダイアログは表示されません。 

```csharp
var status = await Permissions.RequestAsync<Permissions.LocationWhenInUse>();
```

必要なアクセス許可が宣言されていない場合は、`PermissionException` がスローされます。 

一部のプラットフォームでは、アクセス許可要求をアクティブにできるのは 1 回だけであることに注意してください。 アクセス許可が `Denied` 状態であるかどうかを確認し、手動で有効にするようにユーザーに依頼するには、開発者がさらにプロンプトを処理する必要があります。

## <a name="permission-status"></a>アクセス許可の状態

`CheckStatusAsync` または `RequestAsync` を使用すると、`PermissionStatus` が返されます。これを使って次の手順を決定できます。

* Unknown - アクセス許可は不明な状態です
* Denied - ユーザーがアクセス許可要求を拒否しました
* Disabled - この機能はデバイス上で無効になっています
* Granted - ユーザーがアクセス許可を付与したか、自動的に付与されます
* Restricted - 制限された状態です

## <a name="available-permissions"></a>利用可能なアクセス許可

Xamarin.Essentials では可能な限り多くのアクセス許可を抽象化しようとしますが、各オペレーティング システムにはさまざまな実行時アクセス許可のセットが備わっています。 また、一部のアクセス許可に対しては 1 つの API を提供できるという違いもあります。 現在利用可能なアクセス許可についてのガイドを次に示します。

アイコンによるガイド:

* ![完全サポート](~/media/shared/yes.png "完全サポート") - サポートされています
* ![サポートされていません](~/media/shared/no.png "サポートされていないか、不要です") - サポートされていないか、不要です

| アクセス許可 | Android | iOS | UWP | watchOS | tvOS | Tizen |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: 
| CalendarRead   | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| CalendarWrite | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Camera | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされています](~/media/shared/yes.png "Tizen はサポートされています") |
| ContactsRead | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| ContactsWrite | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Flashlight | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされていません](~/media/shared/no.png "iOS はサポートされていません") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされています](~/media/shared/yes.png "Tizen はサポートされています") |
| LocationWhenInUse | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされています](~/media/shared/yes.png "tvOS はサポートされています")  | ![Tizen はサポートされています](~/media/shared/yes.png "Tizen はサポートされています") |
| LocationAlways | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされています](~/media/shared/yes.png "Tizen はサポートされています") |
| Media | ![Android はサポートされていません](~/media/shared/no.png "Android はサポートされていません") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Microphone | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされています](~/media/shared/yes.png "Tizen はサポートされています") |
| Phone | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Photos | ![Android はサポートされていません](~/media/shared/no.png "Android はサポートされていません") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされています](~/media/shared/yes.png "tvOS はサポートされています") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Reminders | ![Android はサポートされていません](~/media/shared/no.png "Android はサポートされていません") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Sensors | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされています](~/media/shared/yes.png "UWP はサポートされています") | ![watchOS はサポートされています](~/media/shared/yes.png "watchOS はサポートされています") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Sms | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| Speech | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされています](~/media/shared/yes.png "iOS はサポートされています") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| StorageRead | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされていません](~/media/shared/no.png "iOS はサポートされていません") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |
| StorageWrite | ![Android はサポートされています](~/media/shared/yes.png "Android はサポートされています") | ![iOS はサポートされていません](~/media/shared/no.png "iOS はサポートされていません") | ![UWP はサポートされていません](~/media/shared/no.png "UWP はサポートされていません") | ![watchOS はサポートされていません](~/media/shared/no.png "watchOS はサポートされていません") | ![tvOS はサポートされていません](~/media/shared/no.png "tvOS はサポートされていません") | ![Tizen はサポートされていません](~/media/shared/no.png "Tizen はサポートされていません") |

アクセス許可に ![サポートされていません](~/media/shared/no.png "サポート外")マークが付いている場合は、確認または要求されたときに常に `Granted` が返されます。

## <a name="general-usage"></a>一般的な使用法
アクセス許可を処理するための一般的な使用パターンを次に示します。

```csharp
public async Task<PermissionStatus> CheckAndRequestPermissionAsync<TPermission>()
{
    var status = await Permissions.CheckStatusAsync<Permissions.LocationWhenInUse>();
    if (status != PermissionStatus.Granted)
    {
        status = await Permissions.RequestAsync<Permissions.LocationWhenInUse>();
    }

    // Additionally could prompt the user to turn on in settings

    return status;
}
```

各種類のアクセス許可では、そのインスタンスを作成して、メソッドを直接呼び出すことができます。

```csharp
public async Task GetLocationAsync()
{
    var status = await CheckAndRequestPermissionAsync(new Permissions.LocationWhenInUse());
    if (status != PermissionStatus.Granted)
    {
        // Notify user permission was denied
        return;
    }

    var location = await Geolocation.GetLocationAsync();
}

public async Task<PermissionStatus> CheckAndRequestPermissionAsync<T>(T permission)
            where T : BasePermission
{
    var status = await permission.CheckStatusAsync();
    if (status != PermissionStatus.Granted)
    {
        status = await permission.RequestAsync();
    }

    return status;
}
```

## <a name="extending-permissions"></a>アクセス許可の拡張

Permissions API は、Xamarin.Essentials に含まれていない追加の検証やアクセス許可が必要なアプリケーションに対して、柔軟かつ拡張可能であるように作成されています。 `BasePermission` から継承した新しいクラスを作成し、必要な抽象メソッドを実装します。 Then 

```csharp
public class MyPermission : BasePermission
{
    // This method checks if current status of the permission
    public override Task<PermissionStatus> CheckStatusAsync()
    {
        throw new System.NotImplementedException();
    }

    // This method is optional and a PermissionException is often thrown if a permission is not declared
    public override void EnsureDeclared()
    {
        throw new System.NotImplementedException();
    }

    // Requests the user to accept or deny a permission
    public override Task<PermissionStatus> RequestAsync()
    {
        throw new System.NotImplementedException();
    }
}
```

特定のプラットフォームでアクセス許可を実装する場合、`BasePlatformPermission` クラスから継承することができます。 これにより、宣言を自動的に確認するための、追加のプラットフォーム ヘルパー メソッドが提供されます。

## <a name="platform-implementation-specifics"></a>プラットフォームの実装の詳細

# <a name="android"></a>[Android](#tab/android)

アクセス許可には、Android マニフェスト ファイルに一致する属性が設定されている必要があります。

詳細については、「[Xamarin.Android のアクセス許可](https://docs.microsoft.com/xamarin/android/app-fundamentals/permissions)」のドキュメントをご覧ください。

# <a name="ios"></a>[iOS](#tab/ios)

アクセス許可は、`Info.plist` ファイルの文字列に一致する必要があります。 一度アクセス許可を要求して拒否されると、もう一度アクセス許可を要求してもポップアップが表示されなくなります。 iOS のアプリケーション設定画面で設定を手動で調整するようにユーザーに要求する必要があります。

詳細については、「[iOS のセキュリティとプライバシーの機能](https://docs.microsoft.com/xamarin/ios/app-fundamentals/security-privacy)」のドキュメントをご覧ください。

# <a name="uwp"></a>[UWP](#tab/uwp)

アクセス許可には、パッケージ マニフェストで宣言された機能と一致する機能が備わっている必要があります。

詳細については、[アプリ機能の宣言](https://docs.microsoft.com/windows/uwp/packaging/app-capability-declarations)に関するドキュメントをご覧ください。

--------------

## <a name="api"></a>API

- [アクセス許可のソース コード](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/Permissions)
- [Permissions API のドキュメント](xref:Xamarin.Essentials.Permissions)

