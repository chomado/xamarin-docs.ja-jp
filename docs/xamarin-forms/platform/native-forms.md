---
title: Xamarin Native プロジェクトで Xamarin.Forms
description: この記事では、Xamarin のネイティブ プロジェクトに直接追加される ContentPage から派生したページを使用する方法とそれらの間を移動する方法について説明します。
ms.prod: xamarin
ms.assetid: f343fc21-dfb1-4364-a332-9da6705d36bc
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 08/19/2019
ms.openlocfilehash: 83b445b8379ad5181ab1dce142cca06625be79ad
ms.sourcegitcommit: d0e6436edbf7c52d760027d5e0ccaba2531d9fef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75487335"
---
# <a name="xamarinforms-in-xamarin-native-projects"></a>Xamarin Native プロジェクトで Xamarin.Forms

[![サンプルのダウンロード](~/media/shared/download.png)サンプルのダウンロード](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/native2forms)

通常、Xamarin.Forms アプリケーションから派生した 1 つまたは複数のページが含まれます。 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)、これらのページは、.NET Standard ライブラリ プロジェクトまたは共有プロジェクトのすべてのプラットフォームによって共有されます。 ただし、ネイティブのフォームでは、 `ContentPage`-Xamarin.iOS、Xamarin.Android、および UWP のネイティブ アプリケーションに直接追加するページを派生します。 使用するネイティブ プロジェクトを持つと比較して`ContentPage`-派生ページから、.NET Standard ライブラリ プロジェクトまたは共有プロジェクト、ネイティブ プロジェクトに直接ページの追加の利点は、ネイティブ ビューでページを拡張することができます。 XAML でのネイティブ ビューを付けることができますし、`x:Name`分離コードから参照されているとします。 ネイティブ ビューの詳細については、次を参照してください。[ネイティブ ビュー](~/xamarin-forms/platform/native-views/index.md)します。

Xamarin.Forms を使用するためのプロセス[ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ネイティブ プロジェクト内の派生のページを次に示します。

1. ネイティブ プロジェクトに Xamarin.Forms NuGet パッケージを追加します。
1. 追加、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ページ、およびネイティブ プロジェクトへのすべての依存関係を派生します。
1. `Forms.Init` メソッドを呼び出します。
1. インスタンスを構築、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ページを派生し、次の拡張メソッドのいずれかを使用して、適切なネイティブ型に変換: `CreateViewController` ios、 `CreateSupportFragment` for Android、または`CreateFrameworkElement`のUWP です。
1. ネイティブな型表現に移動し、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ネイティブ ナビゲーション API を使用してページを派生します。

Xamarin.Forms を呼び出すことによって初期化する必要があります、`Forms.Init`メソッドのネイティブ プロジェクトを作成する前に、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ページを派生します。 アプリケーション フローの最も便利なタイミングによって決まります主にこれを実行するタイミングを選択する – アプリケーションの起動時または直前に実行できます、 `ContentPage`-派生ページを構築します。 この記事と付属のサンプル アプリケーションで、`Forms.Init`メソッドは、アプリケーションの起動時に呼び出されます。

> [!NOTE]
> **NativeForms**サンプル アプリケーション ソリューションには、Xamarin.Forms プロジェクトが含まれていません。 代わりに、Xamarin.iOS プロジェクトを Xamarin.Android プロジェクトの場合、および UWP プロジェクトで構成されます。 各プロジェクトは、使用するネイティブのフォームを使用するネイティブ プロジェクト[ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-ページを派生します。 ただし、ネイティブのプロジェクトで利用できませんでした理由理由はありません`ContentPage`-.NET Standard ライブラリ プロジェクトまたは共有プロジェクトからページを派生します。

ネイティブ数字形式を使用して、Xamarin.Forms などの機能[ `DependencyService` ](xref:Xamarin.Forms.DependencyService)、 [ `MessagingCenter` ](xref:Xamarin.Forms.MessagingCenter)、およびデータ バインディング エンジンで、すべて引き続き使用します。 ただし、ページ ナビゲーションは、ネイティブ ナビゲーション API を使用して実行する必要があります。

## <a name="ios"></a>iOS

Ios では、`FinishedLaunching`で上書き、`AppDelegate`クラスは、アプリケーションを実行する場所では通常、スタートアップ タスクに関連します。 アプリケーションがついに発売されましたが、通常、メイン ウィンドウを構成し、コント ローラーを表示するオーバーライド後に呼び出されます。 次のコード例は、`AppDelegate`サンプル アプリケーション内のクラス。

```csharp
[Register("AppDelegate")]
public class AppDelegate : UIApplicationDelegate
{
    public static AppDelegate Instance;
    UIWindow _window;
    AppNavigationController _navigation;

    public static string FolderPath { get; private set; }

    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        Forms.Init();

        Instance = this;
        _window = new UIWindow(UIScreen.MainScreen.Bounds);

        UINavigationBar.Appearance.SetTitleTextAttributes(new UITextAttributes
        {
            TextColor = UIColor.Black
        });

        FolderPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData));
        UIViewController mainPage = new NotesPage().CreateViewController();
        mainPage.Title = "Notes";

        _navigation = new AppNavigationController(mainPage);
        _window.RootViewController = _navigation;
        _window.MakeKeyAndVisible();

        return true;
    }
    // ...
}
```

`FinishedLaunching`メソッドは、次のタスクを実行します。

- Xamarin.Forms は呼び出すことによって初期化、`Forms.Init`メソッド。
- 参照、`AppDelegate`でクラスが格納されている、 `static` `Instance`フィールド。 これは他のクラスで定義されているメソッドを呼び出すためのメカニズムを提供する、`AppDelegate`クラス。
- `UIWindow`、これは、ネイティブの iOS アプリケーションでのビューのメイン コンテナーを作成します。
- `FolderPath` プロパティは、ノートデータが格納されるデバイス上のパスに初期化されます。
- `NotesPage`クラスは、これは、Xamarin.Forms [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページの XAML で定義されている、作成され、変換、`UIViewController`を使用して、`CreateViewController`拡張メソッド。
- `Title`のプロパティ、`UIViewController`に表示される設定、`UINavigationBar`します。
- A`AppNavigationController`階層型ナビゲーションを管理するために作成されます。 これは、`UINavigationController`から派生するカスタムナビゲーションコントローラークラスです。 `AppNavigationController` オブジェクトはビューコントローラーのスタックを管理し、コンストラクターに渡された `UIViewController` は `AppNavigationController` が読み込まれるときに最初に表示されます。
- `AppNavigationController` オブジェクトは `UIWindow`の最上位の `UIViewController` として設定され、`UIWindow` はアプリケーションのキーウィンドウとして設定され、表示されるようになります。

1 回、`FinishedLaunching`メソッドの実行が、UI は、Xamarin.Forms で定義されている`NotesPage`次のスクリーン ショットに示すようにクラスが表示されます。

[![XAML で定義された UI を使用する Xamarin iOS アプリケーションのスクリーンショット](native-forms-images/ios-notespage.png "XAML UI を使用した Xamarin iOS アプリ")](native-forms-images/ios-notespage-large.png#lightbox "XAML UI を使用した Xamarin iOS アプリ")

たとえば、 **+** [`Button`](xref:Xamarin.Forms.Button)をタップして UI と対話すると、`NotesPage` コードビハインドで次のイベントハンドラーが実行されます。

```csharp
void OnNoteAddedClicked(object sender, EventArgs e)
{
    AppDelegate.Instance.NavigateToNoteEntryPage(new Note());
}
```

`static` `AppDelegate.Instance`フィールドを使用できます、`AppDelegate.NavigateToNoteEntryPage`の次のコード例に示すメソッドが呼び出されます。

```csharp
public void NavigateToNoteEntryPage(Note note)
{
    var noteEntryPage = new NoteEntryPage
    {
        BindingContext = note
    }.CreateViewController();
    noteEntryPage.Title = "Note Entry";
    _navigation.PushViewController(noteEntryPage, true);
}
```

`NavigateToNoteEntryPage`メソッドは、Xamarin.Forms を変換[ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生にページ、`UIViewController`で、`CreateViewController`拡張メソッド、およびセット、`Title`のプロパティ、`UIViewController`です。 `UIViewController`しプッシュ`AppNavigationController`によって、`PushViewController`メソッド。 そのため、UI を Xamarin.Forms で定義されている`NoteEntryPage`次のスクリーン ショットに示すようにクラスが表示されます。

[![XAML で定義された UI を使用する Xamarin iOS アプリケーションのスクリーンショット](native-forms-images/ios-noteentrypage.png "XAML UI を使用した Xamarin iOS アプリ")](native-forms-images/ios-noteentrypage-large.png#lightbox "XAML UI を使用した Xamarin iOS アプリ")

`NoteEntryPage` が表示されると、戻るナビゲーションによって `AppNavigationController`から `NoteEntryPage` クラスの `UIViewController` がポップされ、`UIViewController` クラスの `NotesPage` にユーザーが返されます。 ただし、iOS ネイティブナビゲーションスタックから `UIViewController` をポップしても、`UIViewController` およびアタッチされた `Page` オブジェクトは自動的に破棄されません。 したがって、`AppNavigationController` クラスは `PopViewController` メソッドをオーバーライドして、後方ナビゲーション時にビューコントローラーを破棄します。

```csharp
public class AppNavigationController : UINavigationController
{
    //...
    public override UIViewController PopViewController(bool animated)
    {
        UIViewController topView = TopViewController;
        if (topView != null)
        {
            // Dispose of ViewController on back navigation.
            topView.Dispose();
        }
        return base.PopViewController(animated);
    }
}
```

`PopViewController` オーバーライドは、iOS ネイティブナビゲーションスタックからポップされた `UIViewController` オブジェクトの `Dispose` メソッドを呼び出します。 この操作を行わないと、`UIViewController` とアタッチされた `Page` オブジェクトが孤立します。

> [!IMPORTANT]
> 孤立したオブジェクトはガベージコレクションできないため、メモリリークが発生します。

## <a name="android"></a>Android

Android では、`OnCreate`で上書き、`MainActivity`クラスは、アプリケーションを実行する場所では通常、スタートアップ タスクに関連します。 次のコード例は、`MainActivity`サンプル アプリケーション内のクラス。

```csharp
public class MainActivity : AppCompatActivity
{
    public static string FolderPath { get; private set; }

    public static MainActivity Instance;

    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        Forms.Init(this, bundle);
        Instance = this;

        SetContentView(Resource.Layout.Main);
        var toolbar = FindViewById<Toolbar>(Resource.Id.toolbar);
        SetSupportActionBar(toolbar);
        SupportActionBar.Title = "Notes";

        FolderPath = Path.Combine(System.Environment.GetFolderPath(System.Environment.SpecialFolder.LocalApplicationData));
        Android.Support.V4.App.Fragment mainPage = new NotesPage().CreateSupportFragment(this);
        SupportFragmentManager
            .BeginTransaction()
            .Replace(Resource.Id.fragment_frame_layout, mainPage)
            .Commit();
        ...
    }
    ...
}
```

`OnCreate`メソッドは、次のタスクを実行します。

- Xamarin.Forms は呼び出すことによって初期化、`Forms.Init`メソッド。
- 参照、`MainActivity`でクラスが格納されている、 `static` `Instance`フィールド。 これは他のクラスで定義されているメソッドを呼び出すためのメカニズムを提供する、`MainActivity`クラス。
- `Activity`レイアウト リソースからコンテンツを設定します。 レイアウトは、サンプル アプリケーションで、`LinearLayout`を格納している、 `Toolbar`、および`FrameLayout`フラグメントのコンテナーとして機能します。
- `Toolbar`が取得され、設定の操作バーとして、 `Activity`、操作バーのタイトルを設定します。
- `FolderPath` プロパティは、ノートデータが格納されるデバイス上のパスに初期化されます。
- `NotesPage`クラスは、これは、Xamarin.Forms [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページの XAML で定義されている、作成され、変換、`Fragment`を使用して、`CreateSupportFragment`拡張メソッド。
- `SupportFragmentManager`クラスを作成し、置換するトランザクションのコミット、`FrameLayout`インスタンス、`Fragment`の`NotesPage`クラス。

フラグメントの詳細については、次を参照してください。[フラグメント](~/android/platform/fragments/index.md)します。

1 回、`OnCreate`メソッドの実行が、UI は、Xamarin.Forms で定義されている`NotesPage`次のスクリーン ショットに示すようにクラスが表示されます。

[![XAML で定義された UI を使用する Xamarin Android アプリケーションのスクリーンショット](native-forms-images/android-notespage.png "XAML UI を使用した Xamarin Android アプリ")](native-forms-images/android-notespage-large.png#lightbox "XAML UI を使用した Xamarin Android アプリ")

たとえば、 **+** [`Button`](xref:Xamarin.Forms.Button)をタップして UI と対話すると、`NotesPage` コードビハインドで次のイベントハンドラーが実行されます。

```csharp
void OnNoteAddedClicked(object sender, EventArgs e)
{
    MainActivity.Instance.NavigateToNoteEntryPage(new Note());
}
```

`static` `MainActivity.Instance`フィールドを使用できます、`MainActivity.NavigateToNoteEntryPage`の次のコード例に示すメソッドが呼び出されます。

```csharp
public void NavigateToNoteEntryPage(Note note)
{
    Android.Support.V4.App.Fragment noteEntryPage = new NoteEntryPage
    {
        BindingContext = note
    }.CreateSupportFragment(this);
    SupportFragmentManager
        .BeginTransaction()
        .AddToBackStack(null)
        .Replace(Resource.Id.fragment_frame_layout, noteEntryPage)
        .Commit();
}
```

`NavigateToNoteEntryPage`メソッドは、Xamarin.Forms を変換します。 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生にページ、`Fragment`で、`CreateSupportFragment`拡張メソッドを追加し、`Fragment`フラグメント バック スタック。 そのため、UI を Xamarin.Forms で定義されている`NoteEntryPage`次のスクリーン ショットに示すように表示されます。

[![XAML で定義された UI を使用する Xamarin Android アプリケーションのスクリーンショット](native-forms-images/android-noteentrypage.png "XAML UI を使用した Xamarin Android アプリ")](native-forms-images/android-noteentrypage-large.png#lightbox "XAML UI を使用した Xamarin Android アプリ")

ときに、`NoteEntryPage`背面をタップして、表示される矢印が表示されます、`Fragment`の`NoteEntryPage`フラグメントのバック スタックからユーザーを返す、`Fragment`の`NotesPage`クラス。

### <a name="enable-back-navigation-support"></a>バックナビゲーションサポートを有効にする

`SupportFragmentManager`クラスには、`BackStackChanged`フラグメント戻るスタックのコンテンツが変更されるたびに発生するイベントです。 `OnCreate`メソッドで、`MainActivity`クラスには、このイベントの匿名のイベント ハンドラーが含まれています。

```csharp
SupportFragmentManager.BackStackChanged += (sender, e) =>
{
    bool hasBack = SupportFragmentManager.BackStackEntryCount > 0;
    SupportActionBar.SetHomeButtonEnabled(hasBack);
    SupportActionBar.SetDisplayHomeAsUpEnabled(hasBack);
    SupportActionBar.Title = hasBack ? "Note Entry" : "Notes";
};
```

このイベント ハンドラーは、1 つまたは複数があること、操作バーで [戻る] ボタンを表示`Fragment`フラグメント上のインスタンスがスタックをバックアップします。 [戻る] ボタンをタップする応答の処理、`OnOptionsItemSelected`をオーバーライドします。

```csharp
public override bool OnOptionsItemSelected(Android.Views.IMenuItem item)
{
    if (item.ItemId == global::Android.Resource.Id.Home && SupportFragmentManager.BackStackEntryCount > 0)
    {
        SupportFragmentManager.PopBackStack();
        return true;
    }
    return base.OnOptionsItemSelected(item);
}
```

`OnOptionsItemSelected`オプション メニュー内の項目が選択されるたびに、オーバーライドが呼び出されます。 この実装は、[戻る] ボタンが選択されており、1 つまたは複数があることに、フラグメントのバック スタックから現在のフラグメントをポップ`Fragment`フラグメント上のインスタンスがスタックをバックアップします。

### <a name="multiple-activities"></a>複数のアクティビティ

アプリケーションが複数のアクティビティで構成される場合[ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページは、各アクティビティに埋め込まれることができます。 このシナリオで、`Forms.Init`メソッドでのみ呼び出す必要があります、`OnCreate`最初のオーバーライド`Activity`、Xamarin.Forms を埋め込みますを`ContentPage`します。 ただし、これに、次の影響です。

- 値`Xamarin.Forms.Color.Accent`から取得されます、`Activity`という、`Forms.Init`メソッド。
- 値`Xamarin.Forms.Application.Current`に関連付けられた、`Activity`という、`Forms.Init`メソッド。

### <a name="choose-a-file"></a>ファイルの選択

埋め込み時、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生を使用するページ、 [ `WebView` ](xref:Xamarin.Forms.WebView)する必要がある HTML"Choose File"をサポートするボタン、`Activity`をオーバーライドする必要があります、 `OnActivityResult`方法:

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    ActivityResultCallbackRegistry.InvokeCallback(requestCode, resultCode, data);
}
```

## <a name="uwp"></a>UWP

UWP、ネイティブの`App`クラスは、アプリケーションを実行する場所では通常、スタートアップ タスクに関連します。 Xamarin.Forms は、通常は、Xamarin.Forms UWP アプリケーションで初期化、`OnLaunched`ネイティブ オーバーライド`App`を渡すためのクラス、`LaunchActivatedEventArgs`への引数、`Forms.Init`メソッド。 この理由は、Xamarin.Forms を使用するネイティブの UWP アプリケーション[ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページが最も簡単に呼び出すことができます、`Forms.Init`からメソッド、`App.OnLaunched`メソッド。

既定では、ネイティブ`App`クラスの起動、`MainPage`アプリケーションの最初のページとしてクラス。 次のコード例は、`MainPage`サンプル アプリケーション内のクラス。

```csharp
public sealed partial class MainPage : Page
{
    NotesPage notesPage;
    NoteEntryPage noteEntryPage;
    
    public static MainPage Instance;
    public static string FolderPath { get; private set; }

    public MainPage()
    {
        this.InitializeComponent();
        this.NavigationCacheMode = NavigationCacheMode.Enabled;
        Instance = this;
        FolderPath = Path.Combine(System.Environment.GetFolderPath(System.Environment.SpecialFolder.LocalApplicationData));
        notesPage = new Notes.UWP.Views.NotesPage();
        this.Content = notesPage.CreateFrameworkElement();
        // ...        
    } 
    // ...
}
```

`MainPage`コンス トラクターは、次のタスクを実行します。

- ページのキャッシュが有効になっているように、新しい`MainPage`のページに戻るユーザーが移動したときに構成されていません。
- 参照、`MainPage`でクラスが格納されている、 `static` `Instance`フィールド。 これは他のクラスで定義されているメソッドを呼び出すためのメカニズムを提供する、`MainPage`クラス。
- `FolderPath` プロパティは、ノートデータが格納されるデバイス上のパスに初期化されます。
- `NotesPage`クラスは、これは、Xamarin.Forms [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページの XAML で定義されているが構築されに変換、`FrameworkElement`を使用して、`CreateFrameworkElement`拡張メソッドとのコンテンツとして設定します`MainPage`クラスです。

1 回、`MainPage`コンス トラクターが実行される、Xamarin.Forms で定義されている UI`NotesPage`次のスクリーン ショットに示すようにクラスが表示されます。

[![Xamarin の XAML で定義された UI を使用する UWP アプリケーションのスクリーンショット](native-forms-images/uwp-notespage.png "Xamarin. Forms XAML UI を使用した UWP アプリ")](native-forms-images/uwp-notespage-large.png#lightbox "Xamarin. Forms XAML UI を使用した UWP アプリ")

たとえば、 **+** [`Button`](xref:Xamarin.Forms.Button)をタップして UI と対話すると、`NotesPage` コードビハインドで次のイベントハンドラーが実行されます。

```csharp
void OnNoteAddedClicked(object sender, EventArgs e)
{
    MainPage.Instance.NavigateToNoteEntryPage(new Note());
}
```

`static` `MainPage.Instance`フィールドを使用できます、`MainPage.NavigateToNoteEntryPage`の次のコード例に示すメソッドが呼び出されます。

```csharp
public void NavigateToNoteEntryPage(Note note)
{
    noteEntryPage = new Notes.UWP.Views.NoteEntryPage
    {
        BindingContext = note
    };
    this.Frame.Navigate(noteEntryPage);
}
```

UWP でのナビゲーションは、通常の実行、`Frame.Navigate`を受け取るメソッドを`Page`引数。 Xamarin.Forms の定義、`Frame.Navigate`拡張メソッドを受け取る、 [ `ContentPage` ](xref:Xamarin.Forms.ContentPage)-派生ページ インスタンス。 したがって、ときに、`NavigateToNoteEntryPage`メソッドが実行される、Xamarin.Forms で定義されている UI`NoteEntryPage`次のスクリーン ショットに示すように表示されます。

[![Xamarin の XAML で定義された UI を使用する UWP アプリケーションのスクリーンショット](native-forms-images/uwp-noteentrypage.png "Xamarin. Forms XAML UI を使用した UWP アプリ")](native-forms-images/uwp-noteentrypage-large.png#lightbox "Xamarin. Forms XAML UI を使用した UWP アプリ")

ときに、`NoteEntryPage`背面をタップして、表示される矢印が表示されます、`FrameworkElement`の`NoteEntryPage`アプリ内のバック スタックからユーザーを返す、`FrameworkElement`の`NotesPage`クラス。

### <a name="enable-page-resizing-support"></a>ページサイズ変更のサポートを有効にする

UWP アプリケーションウィンドウのサイズを変更する場合は、Xamarin のコンテンツのサイズも変更する必要があります。 これを実現するには、`MainPage` コンストラクターに `Loaded` イベントのイベントハンドラーを登録します。

```csharp
public MainPage()
{
    // ...
    this.Loaded += OnMainPageLoaded;
    // ...
}
```

`Loaded` イベントは、ページがレイアウトされ、レンダリングされ、操作の準備が整ったときに発生し、`OnMainPageLoaded` メソッドを応答として実行します。

```csharp
void OnMainPageLoaded(object sender, RoutedEventArgs e)
{
    this.Frame.SizeChanged += (o, args) =>
    {
        if (noteEntryPage != null)
            noteEntryPage.Layout(new Xamarin.Forms.Rectangle(0, 0, args.NewSize.Width, args.NewSize.Height));
        else
            notesPage.Layout(new Xamarin.Forms.Rectangle(0, 0, args.NewSize.Width, args.NewSize.Height));
    };
}
```

`OnMainPageLoaded` メソッドは、`Frame.SizeChanged` イベントの匿名イベントハンドラーを登録します。これは、`Frame`で `ActualHeight` または `ActualWidth` プロパティが変更されたときに発生します。 応答として、アクティブページの Xamarin のコンテンツは `Layout` メソッドを呼び出すことによってサイズが変更されます。

### <a name="enable-back-navigation-support"></a>バックナビゲーションサポートを有効にする

UWP では、アプリケーションが別のデバイス フォーム ファクターのすべてのハードウェアとソフトウェア戻るボタン、ナビゲーションを有効にする必要があります。 これは、`BackRequested` イベントのイベントハンドラーを登録することによって実現できます。これは、`MainPage` コンストラクターで実行できます。

```csharp
public MainPage()
{
    // ...
    SystemNavigationManager.GetForCurrentView().BackRequested += OnBackRequested;
}
```

アプリケーションを起動すると、`GetForCurrentView`メソッドの取得、`SystemNavigationManager`オブジェクトは、現在のビューに関連付けられているし、イベント ハンドラーを登録します、`BackRequested`イベント。 アプリケーションは、前面のアプリケーションは、応答を呼び出す場合のみこのイベントを受け取る、`OnBackRequested`イベント ハンドラー。

```csharp
void OnBackRequested(object sender, BackRequestedEventArgs e)
{
    Frame rootFrame = Window.Current.Content as Frame;
    if (rootFrame.CanGoBack)
    {
        e.Handled = true;
        rootFrame.GoBack();
        noteEntryPage = null;
    }
}
```

`OnBackRequested`イベント ハンドラーの呼び出し、`GoBack`セット、アプリケーションのルート フレームのメソッド、`BackRequestedEventArgs.Handled`プロパティを`true`イベントを処理済みとしてマークします。 イベントを処理済みとしてマークしないと、イベントは無視されます。

アプリケーションでは、タイトルバーに [戻る] ボタンを表示するかどうかを選択します。 これを実現するには、`AppViewBackButtonVisibility` プロパティを `App` クラスの `AppViewBackButtonVisibility` 列挙値のいずれかに設定します。

```csharp
void OnNavigated(object sender, NavigationEventArgs e)
{
    SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility =
        ((Frame)sender).CanGoBack ? AppViewBackButtonVisibility.Visible : AppViewBackButtonVisibility.Collapsed;
}
```

`OnNavigated`への応答に実行されるイベント ハンドラー、`Navigated`イベントの発生は、ページ ナビゲーションが発生した場合に、タイトル バーの [戻る] ボタンの可視性を更新します。 これにより、タイトル バーの [戻る] ボタンが表示されるは、アプリに戻るスタックが空でない場合は、または、アプリに戻るスタックが空の場合は、タイトル バーから削除します。

UWP の戻るナビゲーション サポートの詳細については、次を参照してください。[ナビゲーション履歴内を後方に向かってと UWP アプリのナビゲーション](/windows/uwp/design/basics/navigation-history-and-backwards-navigation/)します。

## <a name="related-links"></a>関連リンク

- [NativeForms (サンプル)](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/native2forms)
- [ネイティブ ビュー](~/xamarin-forms/platform/native-views/index.md)
