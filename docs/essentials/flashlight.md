---
title: Xamarin.Essentials:Flashlight
description: このドキュメントでは、デバイスのカメラのフラッシュを懐中電灯にする機能をオンまたはオフにできる Xamarin.Essentials の Flashlight クラスについて説明します。
ms.assetid: 06A03553-D212-43A2-9E6E-C2D2D93EB136
author: jamesmontemagno
ms.author: jamont
ms.date: 11/04/2018
ms.openlocfilehash: b94ba73b334ac68b256ca840956f987a4ef670ce
ms.sourcegitcommit: b0ea451e18504e6267b896732dd26df64ddfa843
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/13/2020
ms.locfileid: "61075580"
---
# <a name="xamarinessentials-flashlight"></a>Xamarin.Essentials:Flashlight

**Flashlight** クラスを使用すると、デバイスのカメラのフラッシュを懐中電灯にする機能をオンまたはオフにできます。

## <a name="get-started"></a>作業開始

[!include[](~/essentials/includes/get-started.md)]

**Flashlight** の機能にアクセスするには、次のプラットフォーム固有の設定が必要です。

# <a name="android"></a>[Android](#tab/android)

Flashlight および Camera アクセス許可が必要であり、Android プロジェクトで構成する必要があります。 これは次の方法で追加できます。

**[プロパティ]** フォルダーにある **AssemblyInfo.cs** ファイルを開き、以下を追加します。

```csharp
[assembly: UsesPermission(Android.Manifest.Permission.Flashlight)]
[assembly: UsesPermission(Android.Manifest.Permission.Camera)]
```

または、Android マニフェストを追加します。

**[プロパティ]** フォルダーにある **AndroidManifest.xml** ファイルを開き、**manifest** ノードの内部に以下を追加します。

```xml
<uses-permission android:name="android.permission.FLASHLIGHT" />
<uses-permission android:name="android.permission.CAMERA" />
```

または、Android プロジェクトを右クリックし、プロジェクトのプロパティを開きます。 **[Android マニフェスト]** の下で **[必要なアクセス許可:]** 領域を探し、**FLASHLIGHT** および **CAMERA** アクセス許可をオンにします。 これにより、**AndroidManifest.xml** ファイルが自動的に更新されます。

これらのアクセス許可を追加すると、[Google Play では特定のハードウェアを持たないデバイスが自動的に除外されます](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions-features)。 Android プロジェクトで AssemblyInfo.cs ファイルに以下を追加することにより、これを回避できます。

```csharp
[assembly: UsesFeature("android.hardware.camera", Required = false)]
[assembly: UsesFeature("android.hardware.camera.autofocus", Required = false)]
```

# <a name="ios"></a>[iOS](#tab/ios)

追加の設定は必要ありません。

# <a name="uwp"></a>[UWP](#tab/uwp)

追加の設定は必要ありません。

-----

## <a name="using-flashlight"></a>Flashlight の使用

自分のクラスに Xamarin.Essentials への参照を追加します。

```csharp
using Xamarin.Essentials;
```

懐中電灯は、`TurnOnAsync` および `TurnOffAsync` メソッドを使用してオンまたはオフにできます。

```csharp
try
{
    // Turn On
    await Flashlight.TurnOnAsync();

    // Turn Off
    await Flashlight.TurnOffAsync();
}
catch (FeatureNotSupportedException fnsEx)
{
    // Handle not supported on device exception
}
catch (PermissionException pEx)
{
    // Handle permission exception
}
catch (Exception ex)
{
    // Unable to turn on/off flashlight
}
```

## <a name="platform-implementation-specifics"></a>プラットフォームの実装の詳細

### <a name="android"></a>[Android](#tab/android)

Flashlight クラスは、デバイスのオペレーティング システムに基づいて最適化されています。

#### <a name="api-level-23-and-higher"></a>API レベル 23 以上

新しい API レベルでは、デバイスのフラッシュ ユニットをオン/オフするために、[Torch モード](https://developer.android.com/reference/android/hardware/camera2/CameraManager.html#setTorchMode)が使用されます。

#### <a name="api-level-22-and-lower"></a>API レベル 22 以下

カメラ ユニットの `FlashMode` をオン/オフするために、カメラ サーフェス テクスチャが作成されます。 

### <a name="ios"></a>[iOS](#tab/ios)

デバイスの Torch モードと Flash モードをオン/オフするには、[AVCaptureDevice](xref:AVFoundation.AVCaptureDevice) を使用します。

### <a name="uwp"></a>[UWP](#tab/uwp)

オン/オフするデバイス背面の最初のランプを検出するには、[Lamp](https://docs.microsoft.com/uwp/api/windows.devices.lights.lamp) を使用します。

-----

## <a name="api"></a>API

- [Flashlight のソース コード](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/Flashlight)
- [Flashlight API のドキュメント](xref:Xamarin.Essentials.Flashlight)
