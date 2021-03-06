---
title: C#6新機能の概要
description: C#言語のバージョン6では、言語を進化させながら定型コードを小さくし、わかりやすくし、一貫性を高めることができます。 クリーン初期化構文、catch/finally ブロックで await を使用する機能、および null 条件を使用するかどうかを確認できます。 演算子は特に便利です。
ms.prod: xamarin
ms.assetid: 4B4E41A8-68BA-4E2B-9539-881AC19971B
ms.custom: xamu-video
author: davidortinau
ms.author: daortin
ms.date: 03/22/2017
ms.openlocfilehash: 1db7ee95ec261739463fa2584f4acf493ac71217
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73014819"
---
# <a name="c-6-new-features-overview"></a>C#6新機能の概要

_C#言語のバージョン6では、言語を進化させながら定型コードを小さくし、わかりやすくし、一貫性を高めることができます。クリーン初期化構文、catch/finally ブロックで await を使用する機能、および null 条件を使用するかどうかを確認できます。演算子は特に便利です。_

> [!NOTE]
> 言語の最新バージョン (バージョン 7) については、「 [7.0 の新機能」 C# ](/dotnet/csharp/whats-new/csharp-7)をご覧ください。 C#

このドキュメントでは、6のC#新機能について説明します。 Mono コンパイラによって完全にサポートされており、開発者はすべての Xamarin ターゲットプラットフォームで新機能の使用を開始できます。

> [!VIDEO https://youtube.com/embed/7UdV7zGPfMU]

**6ビデオのC#新機能**

## <a name="using-c-6"></a>6 C#を使用する

6 C#つのコンパイラは、Visual Studio for Mac の最近のすべてのバージョンで使用されます。
コマンドラインコンパイラを使用して、`mcs --version` が4.0 以上を返すことを確認する必要があります。
Visual Studio for Mac ユーザーは、 **Visual Studio for Mac > Visual Studio for Mac > に関する情報**を参照して Mono 4 (またはそれ以降) がインストールされているかどうかを確認し、詳細を表示することができます。

## <a name="less-boilerplate"></a>Less 定型句
### <a name="using-static"></a>using static
列挙型と、`System.Math`などの特定のクラスは、主に静的な値と関数の所有者です。 6 C#では、1つの`using static`ステートメントを使用して、型のすべての静的メンバーをインポートできます。 5とC# C# 6 の一般的な三角関数を比較します。

```csharp
// Classic C#
class MyClass
{
    public static Tuple<double,double> SolarAngleOld(double latitude, double declination, double hourAngle)
    {
        var tmp = Math.Sin (latitude) * Math.Sin (declination) + Math.Cos (latitude) * Math.Cos (declination) * Math.Cos (hourAngle);
        return Tuple.Create (Math.Asin (tmp), Math.Acos (tmp));
    }
}

// C# 6
using static System.Math;

class MyClass
{
    public static Tuple<double, double> SolarAngleNew(double latitude, double declination, double hourAngle)
    {
        var tmp = Asin (latitude) * Sin (declination) + Cos (latitude) * Cos (declination) * Cos (hourAngle);
        return Tuple.Create (Asin (tmp), Acos (tmp));
    }
}
```

`using static` は、`Math.PI` や `Math.E`などのパブリック `const` フィールドに直接アクセスできません。

```csharp
for (var angle = 0.0; angle <= Math.PI * 2.0; angle += Math.PI / 8) ... 
//PI is const, not static, so requires Math.PI
```

### <a name="using-static-with-extension-methods"></a>拡張メソッドでの static の使用

`using static` ファシリティは、拡張メソッドを使用して若干異なる方法で動作します。 拡張メソッドは `static`を使用して記述されますが、操作するインスタンスがないと意味がありません。 したがって、拡張メソッドを定義する型で `using static` を使用すると、その拡張メソッドはターゲット型 (メソッドの `this` 型) で使用できるようになります。 たとえば、`using static System.Linq.Enumerable` を使用すると、すべての LINQ 型を使用せずに `IEnumerable<T>` オブジェクトの API を拡張できます。

```csharp
using static System.Linq.Enumerable;
using static System.String;

class Program
{
    static void Main()
    {
        var values = new int[] { 1, 2, 3, 4 };
        var evenValues = values.Where (i => i % 2 == 0);
        System.Console.WriteLine (Join(",", evenValues));
    }
}
```

前の例では、動作の違いを示しています。拡張メソッド `Enumerable.Where` が配列に関連付けられていますが、静的メソッド `String.Join` は `String` 型への参照なしで呼び出すことができます。

### <a name="nameof-expressions"></a>すべてのの表記
場合によっては、変数またはフィールドを指定した名前を参照する必要があります。 6 C#では、`nameof(someVariableOrFieldOrType)`は文字列`"someVariableOrFieldOrType"`を返します。 たとえば、`ArgumentException` をスローする場合は、どの引数が無効であるかを指定するのが一般的です。

```csharp
throw new ArgumentException ("Problem with " + nameof(myInvalidArgument))
```

`nameof` 式の最高の利点は、それらが型チェックされ、ツールを使用したリファクタリングと互換性があることです。 `nameof` 式の型チェックは、`string` を使用して型を動的に関連付ける場合に特に歓迎します。 たとえば、iOS では、`UITableView`内の `UITableViewCell` オブジェクトのプロトタイプを設定するために使用される型を指定するために `string` が使用されます。 `nameof` は、スペルミスまたは粗雑なのリファクタリングによって、この関連付けが失敗しないようにすることができます。

```csharp
public override UITableViewCell GetCell (UITableView tableView, NSIndexPath indexPath)
{
    var cell = tableView.DequeueReusableCell (nameof(CellTypeA), indexPath);
    cell.TextLabel.Text = objects [indexPath.Row].ToString ();
    return cell;
}
```

修飾名を `nameof`に渡すことはできますが、最後の要素 (最後の `.`の後) だけが返されます。 たとえば、Xamarin にデータバインディングを追加できます。

```csharp
var myReactiveInstance = new ReactiveType ();
var myLabelOld.BindingContext = myReactiveInstance;
var myLabelNew.BindingContext = myReactiveInstance;
var myLabelOld.SetBinding (Label.TextProperty, "StringField");
var myLabelNew.SetBinding (Label.TextProperty, nameof(ReactiveType.StringField));
```

`SetBinding` の2つの呼び出しで同一の値が渡されています。 `nameof(ReactiveType.StringField)` は `"StringField"`であり、最初に予想される `"ReactiveType.StringField"` ではありません。

## <a name="null-conditional-operator"></a>Null 条件演算子
の以前のC#更新では、null 許容の値を処理するときに定型コードの量を減らすために、null 許容型と null 合体演算子`??`の概念が導入されました。 C#6このテーマは、"null 条件演算子"`?.`で続行されます。 Null 条件演算子は、式の右辺のオブジェクトで使用されている場合、オブジェクトが `null` ない場合はメンバー値を返し、それ以外の場合は `null` を返します。

```csharp
var ss = new string[] { "Foo", null };
var length0 = ss [0]?.Length; // 3
var length1 = ss [1]?.Length; // null
var lengths = ss.Select (s => s?.Length ?? 0); //[3, 0]
```

(`length0` と `length1` の両方が `int?`型であると推論されます)

前の例の最後の行は、null 条件演算子 `?` を `??` の null 合体演算子と組み合わせて示しています。 新しいC# 6 つの null 条件演算子は、配列内の2番目の要素に対して`null`を返します。この時点で、null 合体演算子が開始され、`lengths`配列に0が渡されます (これが適切かどうかにかかわらず、問題に固有のものであるかどうか)。

Null 条件演算子を指定すると、多くの多くのアプリケーションで必要となる定型的な null チェックの量を大幅に削減できます。

あいまいさのために、null 条件演算子にはいくつかの制限があります。 デリゲートを使用する場合と同じように、かっこで囲まれる引数リストを使用して、`?` にすぐに従うことはできません。

```csharp
SomeDelegate?("Some Argument") // Not allowed
```

ただし、`Invoke` は、引数リストから `?` を分離するために使用できます。また、定型句の `null`チェックブロックよりも改善されています。

```csharp
public event EventHandler HandoffOccurred;
public override bool ContinueUserActivity (UIApplication application, NSUserActivity userActivity, UIApplicationRestorationHandler completionHandler)
{
    HandoffOccurred?.Invoke (this, userActivity.UserInfo);
    return true;
}
```

## <a name="string-interpolation"></a>文字列補間
`String.Format` 関数では、従来は、`String.Format("Expected: {0} Received: {1}.", expected, received`などの書式文字列のプレースホルダーとしてインデックスが使用されていました。 もちろん、新しい値を追加すると、常に、引数をカウントし、プレースホルダーの番号を指定し、引数リストの右側のシーケンスに新しい引数を挿入する面倒な作業が発生します。

C#6の新しい文字列補間機能は `String.Format`で大幅に改善されています。 これで、`$`で始まる文字列の変数に直接名前を付けることができます。 たとえば、次のようになります。

```csharp
$"Expected: {expected} Received: {received}."
```

もちろん、変数がチェックされ、スペルミスまたは使用できない変数があると、コンパイラエラーが発生します。

プレースホルダーは、単純な変数である必要はなく、任意の式にすることができます。 これらのプレースホルダー内では、引用符をエスケープ*せず*に引用符で囲むことができます。 たとえば、次の `"s"` に注意してください。

```csharp
var s = $"Timestamp: {DateTime.Now.ToString ("s", System.Globalization.CultureInfo.InvariantCulture )}"
```

文字列補間では、`String.Format`のアラインメントと書式設定の構文がサポートされます。 前に `{index, alignment:format}`を書き込んだのとC#同じように、6で`{placeholder, alignment:format}`を記述します。

```csharp
using static System.Linq.Enumerable;
using System;

class Program
{
    static void Main ()
    {
        var values = new int[] { 1, 2, 3, 4, 12, 123456 };
        foreach (var s in values.Select (i => $"The value is { i,10:N2}.")) {
            Console.WriteLine (s);
        }
    Console.WriteLine ($"Minimum is { values.Min(i => i):N2}.");
    }
}
```

結果:

```
The value is       1.00.
The value is       2.00.
The value is       3.00.
The value is       4.00.
The value is      12.00.
The value is 123,456.00.
Minimum is 1.00.
```

文字列補間は `String.Format`の構文砂糖です。プレースホルダーが使用されていない場合でも、文字列リテラル `@""` と共に使用することはできません。また、`const`と互換性がありません。

```csharp
const string s = $"Foo"; //Error : const requires value
```

文字列補間を使用して関数の引数を構築する一般的なユースケースでは、エスケープ、エンコーディング、およびカルチャの問題について注意する必要があります。 もちろん、SQL と URL のクエリは、サニタイズに不可欠です。 `String.Format`と同様に、文字列補間では `CultureInfo.CurrentCulture`が使用されます。 `CultureInfo.InvariantCulture` の使用方法はもう少しです。

```csharp
Thread.CurrentThread.CurrentCulture  = new CultureInfo ("de");
Console.WriteLine ($"Today is: {DateTime.Now}"); //"21.05.2015 13:52:51"
Console.WriteLine ($"Today is: {DateTime.Now.ToString(CultureInfo.InvariantCulture)}"); //"05/21/2015 13:52:51"
```

## <a name="initialization"></a>初期化

C#6では、プロパティ、フィールド、およびメンバーを指定する簡潔な方法がいくつか用意されています。

### <a name="auto-property-initialization"></a>自動プロパティ初期化

自動プロパティは、フィールドと同じ簡潔な方法で初期化できるようになりました。 変更できない自動プロパティは、getter だけを使用して書き込むことができます。

```csharp
class ToDo
{
    public DateTime Due { get; set; } = DateTime.Now.AddDays(1);
    public DateTime Created { get; } = DateTime.Now;
```

コンストラクターでは、getter のみの自動プロパティの値を設定できます。

```csharp
class ToDo
{
    public DateTime Due { get; set; } = DateTime.Now.AddDays(1);
    public DateTime Created { get; } = DateTime.Now;
    public string Description { get; }

    public ToDo (string description)
    {
        this.Description = description; //Can assign (only in constructor!)
    }
```

この自動プロパティの初期化は、一般に領域を節約する機能であり、オブジェクトの不変性を重視する開発者にとっても重要です。

### <a name="index-initializers"></a>インデックス初期化子

C#6では、インデックス初期化子が導入されています。これにより、インデクサーを持つ型のキーと値の両方を設定できます。 通常、これは `Dictionary`スタイルのデータ構造に対して行います。

```csharp
partial void ActivateHandoffClicked (WatchKit.WKInterfaceButton sender)
{
    var userInfo = new NSMutableDictionary {
        ["Created"] = NSDate.Now,
        ["Due"] = NSDate.Now.AddSeconds(60 * 60 * 24),
        ["Task"] = Description
    };
    UpdateUserActivity ("com.xamarin.ToDo.edit", userInfo, null);
    statusLabel.SetText ("Check phone");
}
```

### <a name="expression-bodied-function-members"></a>式形式の関数メンバー

ラムダ関数にはいくつかの利点があります。その1つは、単に領域を節約することです。 同様に、式形式のクラスメンバーを使用すると、以前のバージョンのC# 6 では実現できなかった小さい関数を少し簡潔に表現できます。

式形式の関数メンバーは、従来のブロック構文ではなく、ラムダの矢印構文を使用します。

```csharp
public override string ToString () => $"{FirstName} {LastName}";
```

ラムダ矢印の構文では、明示的な `return`を使用しないことに注意してください。 `void`を返す関数では、式もステートメントである必要があります。

```csharp
public void Log(string message) => System.Console.WriteLine($"{DateTime.Now.ToString ("s", System.Globalization.CultureInfo.InvariantCulture )}: {message}");
```

式形式のメンバーは、メソッドではサポートされているがプロパティではサポートされていない `async` する規則の対象となります。

```csharp
//A method, so async is valid
public async Task DelayInSeconds(int seconds) => await Task.Delay(seconds * 1000);
//The following property will not compile
public async Task<int> LeisureHours => await Task.FromResult<char> (DateTime.Now.DayOfWeek.ToString().First()) == 'S' ? 16 : 5;
```

## <a name="exceptions"></a>例外

これについては2つの方法はありません。例外処理は適切に処理されません。 6の新C#機能により、例外処理の柔軟性と一貫性が向上します。

### <a name="exception-filters"></a>例外フィルター

定義上、例外的な状況では例外が発生し、特定の型の例外が発生する可能性がある理由やコード*については、* その理由やコードが非常に複雑になることがあります。 C#6では、ランタイムによって評価されるフィルターを使用して実行ハンドラーを保護する機能が導入されています。 これを行うには、通常の `catch(ExceptionType)` 宣言の後に `when (bool)` パターンを追加します。 次の例では、フィルターは、他の解析エラーとは対照的に、`date` パラメーターに関連する解析エラーを識別します。

```csharp
public void ExceptionFilters(string aFloat, string date, string anInt)
{
    try
    {
        var f = Double.Parse(aFloat);
        var d = DateTime.Parse(date);
        var n = Int32.Parse(anInt);
    } catch (FormatException e) when (e.Message.IndexOf("DateTime") > -1) {
        Console.WriteLine ($"Problem parsing \"{nameof(date)}\" argument");
    } catch (FormatException x) {
        Console.WriteLine ("Problem parsing some other argument");
    }
}
```

### <a name="await-in-catchfinally"></a>catch の await...最終的に。。。

5でC#導入された `async` 機能は、その言語のゲームチェンジャーでした。 5 C#では、`await`は`catch`および`finally`ブロックでは許可されていませんでしたが、`async/await`機能の値が指定されています。 C#6は、次のスニペットに示すように、この制限を解除して、非同期の結果をプログラムを通じて一貫して待機できるようにします。

```csharp
async void SomeMethod()
{
    try {
        //...etc...
    } catch (Exception x) {
        var diagnosticData = await GenerateDiagnosticsAsync (x);
        Logger.log (diagnosticData);
    } finally {
        await someObject.FinalizeAsync ();
    }
}
```

## <a name="summary"></a>まとめ

このC#言語は、開発者の生産性を高めながら、優れたプラクティスやサポートツールを促進するために進化し続けています。 このドキュメントでは、6のC#新しい言語機能の概要について説明し、その使用方法を簡単に説明しました。

## <a name="related-links"></a>関連リンク

- [6のC#新しい言語機能](https://github.com/dotnet/roslyn/wiki/New-Language-Features-in-C%23-6)
