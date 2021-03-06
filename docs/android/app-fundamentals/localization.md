---
title: Android のローカライズ
description: このドキュメントでは、Android SDK のローカライズ機能と、Xamarin を使用してそれらにアクセスする方法について説明します。
ms.prod: xamarin
ms.assetid: D1277939-A1E8-468E-B136-820D816AF853
ms.technology: xamarin-android
author: davidortinau
ms.author: daortin
ms.date: 03/01/2018
ms.openlocfilehash: ae97297b81d33c4b9f814d4b3639984b05ce3d72
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73021649"
---
# <a name="android-localization"></a>Android のローカライズ

_このドキュメントでは、Android SDK のローカライズ機能と、Xamarin を使用してそれらにアクセスする方法について説明します。_

## <a name="android-platform-features"></a>Android プラットフォーム機能

このセクションでは、Android の主なローカライズ機能について説明します。 特定のコードと例については、[次のセクション](#basics)に進んでください。

### <a name="locale"></a>ロケール

ユーザーは、[**設定] > [言語 & 入力**] で言語を選択します。 この選択によって、表示される言語と地域の設定の両方が制御されます (例については、 日付と数値の書式設定の場合)。

現在のロケールは、現在のコンテキストの `Resources`を使用して照会できます。

```csharp
var lang = Resources.Configuration.Locale; // eg. "es_ES"
```

この値は、言語コードとロケールコードの両方がアンダースコアで区切られたロケール識別子になります。 詳細については、「StackOverflow を使用した Java ロケールと[Android でサポートされるロケール](https://stackoverflow.com/questions/7973023/what-is-the-list-of-supported-languages-locales-on-android)[の一覧](https://www.oracle.com/technetwork/java/javase/locales-137662.html)」を参照してください。

一般的な例は、次のとおりです。

- 英語 (米国) の `en_US`
- スペイン語 (スペイン) の `es_ES`
- 日本語 (日本) の `ja_JP`
- 中国語 (中国) の `zh_CN`
- 中国語 (台湾) の `zh_TW`
- ポルトガル語 (ポルトガル) の `pt_PT`
- ポルトガル語 (ブラジル) の `pt_BR`

### <a name="locale_changed"></a>LOCALE_CHANGED

Android では、ユーザーが言語の選択を変更すると `android.intent.action.LOCALE_CHANGED` が生成されます。

アクティビティは、次のようにアクティビティの `android:configChanges` 属性を設定することによって、これを処理することを選択できます。

```csharp
[Activity (Label = "@string/app_name", MainLauncher = true, Icon="@drawable/launcher",
    ConfigurationChanges = ConfigChanges.Locale | ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
```

<a name="basics" />

## <a name="internationalization-basics-in-android"></a>Android での国際化の基礎

Android のローカライズ戦略には、次の重要な部分があります。

- ローカライズされた文字列、画像、およびその他のリソースを格納するリソースフォルダー。

- `GetText` メソッド。コードでローカライズされた文字列を取得するために使用します。

- AXML ファイル内の `@string/id` して、ローカライズされた文字列をレイアウトに自動的に配置します。

### <a name="resource-folders"></a>リソースフォルダー

Android アプリケーションは、次のようなリソースフォルダーのほとんどのコンテンツを管理します。

- **layout** : axml レイアウトファイルが含まれます。
- 描画用-イメージとその他の描画**リソースを含み**ます。
- **値**-文字列が含まれます。
- **raw** -データファイルが含まれます。

ほとんどの開発者は、あらかじめ用意**されたディレクトリで**の**dpi**サフィックスを使用してイメージの複数のバージョンを提供することに慣れています。 Android では、デバイスごとに適切なバージョンを選択します。 言語とカルチャの識別子を使用してリソースディレクトリを suffixing することで、複数の言語翻訳を提供する場合にも同じメカニズムが使用されます。

![複数のカルチャ識別子のリソース/描画されたリソース/値フォルダーのスクリーンショット](localization-images/resources.png)

> [!NOTE]
> `es` のようなトップレベル言語を指定する場合は、2文字のみが必要です。ただし、完全なロケールを指定する場合、ディレクトリ名の形式では、2つの部分を区切るためにダッシュと小文字の**r**が必要です (例: **Pt-rbr**または**zh-tw-rbr**)。 これをコードで返された値と比較します。コードにはアンダースコア ( `pt_BR`) これらはどちらも、ダッシュのみを持つ .NET `CultureInfo` クラスで使用される値とは異なります ( `pt-BR`) Xamarin プラットフォーム間で作業する場合は、これらの違いに注意してください。

#### <a name="stringsxml-file-format"></a>文字列 .xml ファイル形式

ローカライズされた**値**のディレクトリ (例 **値-es**または**値-Pt-rbr**) には、そのロケールの翻訳されたテキストを含む、**文字列 .xml**という名前のファイルが含まれている必要があります。

変換可能な各文字列は、`name` 属性として指定されたリソース ID と値として変換された文字列を持つ XML 要素です。

```xml
<string name="app_name">TaskyL10n</string>
```

通常の XML 規則に従ってエスケープする必要があり、`name` は有効な Android リソース ID (スペースまたはダッシュなし) である必要があります。 例の既定の (英語) 文字列ファイルの例を次に示します。

**値/文字列 .xml**

```xml
<resources>
    <string name="app_name">TaskyL10n</string>
    <string name="taskadd">Add Task</string>
    <string name="taskname">Name</string>
    <string name="tasknotes">Notes</string>
    <string name="taskdone">Done</string>
    <string name="taskcancel">Cancel</string>
</resources>
```

スペイン語のディレクトリ**値-es**には、翻訳を含む同じ名前 (**文字列 .xml**) のファイルが含まれています。

**values-es/Strings .xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">TaskyLeon</string>
    <string name="taskadd">agregar tarea</string>
    <string name="taskname">Nombre</string>
    <string name="tasknotes">Notas</string>
    <string name="taskdone">Completo</string>
    <string name="taskcancel">Cancelar</string>
</resources>
```

![複数の値を持つフォルダーのスクリーンショット。それぞれに文字列 .xml ファイルが含まれています。](localization-images/values.png)

文字列ファイルを設定すると、変換された値をレイアウトとコードの両方で参照できます。

### <a name="axml-layout-files"></a>AXML レイアウトファイル

レイアウトファイルでローカライズされた文字列を参照するには、`@string/id` 構文を使用します。 このサンプルの XML スニペットは、ローカライズされたリソース Id で設定された `text` プロパティを示しています (他のいくつかの属性は省略されています)。

```xml
<TextView
    android:id="@+id/NameLabel"
    android:text="@string/taskname"
    ... />
<CheckBox
    android:id="@+id/chkDone"
    android:text="@string/taskdone"
    ... />
```

### <a name="gettext-method"></a>GetText メソッド

コードで翻訳された文字列を取得するには、`GetText` メソッドを使用して、リソース ID を渡します。

```csharp
var cancelText = Resources.GetText (Resource.String.taskcancel);
```

#### <a name="quantity-strings"></a>数量の文字列

Android 文字列リソースでは、次のように、翻訳者がさまざまな数量に対して異なる翻訳を提供できる*数量文字列*を作成することもできます。

- "1 つのタスクが残っています。"
- "残りのタスクは2つあります。"

("n 個のタスクが残っています")。

(**文字列 .xml** )

```xml
<plurals name="numberOfTasks">
   <!--
      As a developer, you should always supply "one" and "other"
      strings. Your translators will know which strings are actually
      needed for their language.
    -->
   <item quantity="one">There is %d task left.</item>
   <item quantity="other">There are %d tasks still to do.</item>
 </plurals>
```

完全な文字列を表示するには、`GetQuantityString` メソッドを使用して、リソース ID と表示される値を渡します (これは2回渡されます)。 2番目のパラメーターは、使用する *`quantity` 文字列*を決定するために Android によって使用されます。3番目のパラメーターは、実際に文字列に置き換えられる値です (両方とも必須です)。

```csharp
var translated = Resources.GetQuantityString (
                    Resource.Plurals.numberOfTasks, taskcount, taskcount);`
```

有効な `quantity` スイッチは次のとおりです。

- ゼロ
- one
- 回
- いくつか
- many
- その他

詳細については、Android の[ドキュメント](https://developer.android.com/guide/topics/resources/string-resource.html#Plurals)を参照してください。特定の言語で "特別な" 処理を必要としない場合、これらの `quantity` 文字列は無視されます (たとえば、英語では `one` と `other`のみが使用され、`zero` 文字列を指定しても効果はなく、使用されません)。

### <a name="images"></a>イメージ

ローカライズされたイメージは、文字列ファイルと同じ規則に従います。アプリケーションで参照されているすべてのイメージは、フォールバックされ**たディレクトリに**配置する必要があります。

ロケール固有のイメージは、描画可能なフォルダーに配置する必要があります。これには、描画可能なフォルダーや、描画可能な **-ja** (dpi 指定子も追加できます **) などが**あります。

このスクリーンショットでは、4つのイメージが、描画できるディレクトリに保存されていますが、1つの**フラグ .png**のみが、他**のディレクトリに**ローカライズされたコピーを持っています。

![複数の描画フォルダーのスクリーンショット (それぞれに1つ以上のローカライズされた .png ファイルが含まれています)](localization-images/drawable.png)

#### <a name="other-resource-types"></a>その他のリソースの種類

また、レイアウト、アニメーション、生ファイルなど、別の種類の言語固有のリソースを提供することもできます。 これは、1つまたは複数のターゲット言語に特定の画面レイアウトを提供できることを意味します。たとえば、非常に長いテキストラベルを使用できるドイツ語専用のレイアウトを作成できます。

アプリケーション設定 `android:supportsRtl="true"`に設定した場合、Android 4.2 では[右から左 (RTL) 言語](https://android-developers.blogspot.fr/2013/03/native-rtl-support-in-android-42.html)のサポートが導入されました。 リソース修飾子 `"ldrtl"` をディレクトリ名に含めて、RTL 表示用にデザインされたカスタムレイアウトを含めることができます。

リソースディレクトリの名前付けとフォールバックの詳細については、別の[リソースを提供](https://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)するための Android のドキュメントを参照してください。

### <a name="app-name"></a>アプリ名

アプリケーション名は、`MainLauncher` アクティビティのの `@string/id` を使用すると、簡単にローカライズできます。

```csharp
[Activity (Label = "@string/app_name", MainLauncher = true, Icon="@drawable/launcher",
    ConfigurationChanges =  ConfigChanges.Orientation | ConfigChanges.Locale)]
```

### <a name="right-to-left-rtl-languages"></a>右から左 (RTL) の言語

Android 4.2 以降では、RTL のレイアウトが完全にサポートされています。詳細については、「[ネイティブ Rtl サポートブログ](https://android-developers.blogspot.dk/2013/03/native-rtl-support-in-android-42.html)」を参照してください。

Android 4.2 (API レベル 17) 以降を使用している場合は、`left` および `right` ではなく、`start` と `end` でアラインメント値を指定できます (`android:paddingStart`など)。 `LayoutDirection`、`TextDirection`、`TextAlignment` などの新しい Api を使用して、RTL リーダーに適合する画面を構築することもできます。

次のスクリーンショットは、アラビア語のローカライズされた[ **tasky**サンプル](https://github.com/conceptdev/xamarin-samples/tree/master/TaskyL10n)を示しています。

[アラビア語の Tasky アプリの![スクリーンショット](localization-images/rtl-ar-sml.png)](localization-images/rtl-ar.png#lightbox) 

次のスクリーンショットは、ヘブライ語のローカライズされた[ **tasky**サンプル](https://github.com/conceptdev/xamarin-samples/tree/master/TaskyL10n)を示しています。

[ヘブライ語の Tasky アプリのスクリーンショットを![](localization-images/rtl-he-sml.png)](localization-images/rtl-he.png#lightbox)

RTL テキストは、LTR テキストと同じように、**文字列 .xml**ファイルを使用してローカライズされます。

## <a name="testing"></a>テスト中

既定のロケールを十分にテストしてください。 なんらかの理由で既定のリソースを読み込むことができない場合 (つまり、不足している)、アプリケーションがクラッシュします。

### <a name="emulator-testing"></a>エミュレーターのテスト

ADB シェルを使用してエミュレーターを特定のロケールに設定する方法については、 [Android Emulator セクションの](https://developer.android.com/guide/topics/resources/localization.html#testing)Google のテストに関するセクションを参照してください。

```shell
adb shell setprop persist.sys.locale fr-CA;stop;sleep 5;start
```

### <a name="device-testing"></a>デバイスのテスト

デバイスでテストするには、**設定**アプリの言語を変更します。

> [!TIP]
> メニュー項目のアイコンと場所をメモしておき、言語を元の設定に戻すことができるようにします。

## <a name="summary"></a>まとめ

この記事では、組み込みのリソース処理を使用した Android アプリケーションのローカライズの基本について説明します。 [このクロスプラットフォームガイド](~/cross-platform/app-fundamentals/localization.md)では、IOS、Android、クロスプラットフォーム (Xamarin. Forms) アプリの I18n と L10n の詳細について説明します。

## <a name="related-links"></a>関連リンク

- [Tasky (コードでローカライズ) (サンプル)](https://github.com/conceptdev/xamarin-samples/tree/master/TaskyL10n)
- [リソースを使用した Android のローカライズ](https://developer.android.com/guide/topics/resources/localization.html)
- [クロスプラットフォームのローカリゼーションの概要](~/cross-platform/app-fundamentals/localization.md)
- [Xamarin. フォームのローカリゼーション](~/xamarin-forms/app-fundamentals/localization/index.md)
- [iOS のローカライズ](~/ios/app-fundamentals/localization/index.md)
