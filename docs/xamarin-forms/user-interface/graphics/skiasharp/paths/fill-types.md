---
title: パスの塗りつぶしの種類
description: この記事では、SkiaSharp パスの塗りつぶしの種類、使用可能なさまざまな効果を検査し、サンプル コードを示します。
ms.prod: xamarin
ms.assetid: 57103A7A-49A2-46AE-894C-7C2664682644
ms.technology: xamarin-skiasharp
author: davidbritch
ms.author: dabritch
ms.date: 03/10/2017
ms.openlocfilehash: 98081ed1a9aef1260150671d4fd026dd64c20b62
ms.sourcegitcommit: db422e33438f1b5c55852e6942c3d1d75dc025c4
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/24/2020
ms.locfileid: "76723640"
---
# <a name="the-path-fill-types"></a>パスの塗りつぶしの種類

[![サンプルのダウンロード](~/media/shared/download.png)サンプルのダウンロード](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/skiasharpforms-demos)

_SkiaSharp path fill 型で可能なさまざまな効果を発見する_

パス内の 2 つの輪郭がオーバー ラップできるし、1 つの輪郭を構成する行が重複することができます。 任意の囲まれた領域は塗りつぶさ可能性があることができますが含まれているすべての領域を塗りつぶすしない可能性があります。 次に例を示します。

![](fill-types-images/filltypeexample.png "Five-pointed star partially filles")

この少しのコントロールがあります。 入力アルゴリズムは、`SKPath`の[`SKFillType`](xref:SkiaSharp.SKPath.FillType)プロパティによって制御されます。これは、 [`SKPathFillType`](xref:SkiaSharp.SKPathFillType)列挙体のメンバーに設定します。

- `Winding`、既定値
- `EvenOdd`
- `InverseWinding`
- `InverseEvenOdd`

ワインディングと偶の両方のアルゴリズムでは、任意の囲まれた領域が入力または無限にその領域から抽出された架空の行に基づいて入力されていないかどうかを決定します。 その行には、パスを構成する 1 つまたは複数の境界の行が交差します。 ワインディングのモードでは、逆方向の領域に描画される線の数を 1 つの方向のバランスに描画される境界線の数でない場合が入力されます。 それ以外の場合、領域が塗りつぶされます。 偶アルゴリズムは、境界の行の数が奇数の場合、領域を塗りつぶします。

多くの日常的なパスを持つワインディングのアルゴリズムは多くの場合、パスのすべての囲まれた領域を塗りつぶします。 一般に、偶アルゴリズムより興味深い結果が生成されます。

クラシックの例は、5つの**星**のページで説明されているように、星5つの星です。 [**Fivepointedstarpage .xaml**](https://github.com/xamarin/xamarin-forms-samples/blob/master/SkiaSharpForms/Demos/Demos/SkiaSharpFormsDemos/Paths/FivePointedStarPage.xaml)ファイルは、2つの `Picker` ビューをインスタンス化してパスの塗りつぶしの種類を選択し、パスがストロークされるか、または塗りつぶされるか、またはその両方を順番に指定します。

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:skia="clr-namespace:SkiaSharp;assembly=SkiaSharp"
             xmlns:skiaforms="clr-namespace:SkiaSharp.Views.Forms;assembly=SkiaSharp.Views.Forms"
             x:Class="SkiaSharpFormsDemos.Paths.FivePointedStarPage"
             Title="Five-Pointed Star">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>

        <Picker x:Name="fillTypePicker"
                Title="Path Fill Type"
                Grid.Row="0"
                Grid.Column="0"
                Margin="10"
                SelectedIndexChanged="OnPickerSelectedIndexChanged">
            <Picker.ItemsSource>
                <x:Array Type="{x:Type skia:SKPathFillType}">
                    <x:Static Member="skia:SKPathFillType.Winding" />
                    <x:Static Member="skia:SKPathFillType.EvenOdd" />
                    <x:Static Member="skia:SKPathFillType.InverseWinding" />
                    <x:Static Member="skia:SKPathFillType.InverseEvenOdd" />
                </x:Array>
            </Picker.ItemsSource>
            <Picker.SelectedIndex>
                0
            </Picker.SelectedIndex>
        </Picker>

        <Picker x:Name="drawingModePicker"
                Title="Drawing Mode"
                Grid.Row="0"
                Grid.Column="1"
                Margin="10"
                SelectedIndexChanged="OnPickerSelectedIndexChanged">
            <Picker.ItemsSource>
                <x:Array Type="{x:Type x:String}">
                    <x:String>Fill only</x:String>
                    <x:String>Stroke only</x:String>
                    <x:String>Stroke then Fill</x:String>
                    <x:String>Fill then Stroke</x:String>
                </x:Array>
            </Picker.ItemsSource>
            <Picker.SelectedIndex>
                0
            </Picker.SelectedIndex>
        </Picker>

        <skiaforms:SKCanvasView x:Name="canvasView"
                                Grid.Row="1"
                                Grid.Column="0"
                                Grid.ColumnSpan="2"
                                PaintSurface="OnCanvasViewPaintSurface" />
    </Grid>
</ContentPage>
```

分離コードファイルでは、両方の `Picker` 値を使用して、次の5つの星を描画します。

```csharp
void OnCanvasViewPaintSurface(object sender, SKPaintSurfaceEventArgs args)
{
    SKImageInfo info = args.Info;
    SKSurface surface = args.Surface;
    SKCanvas canvas = surface.Canvas;

    canvas.Clear();

    SKPoint center = new SKPoint(info.Width / 2, info.Height / 2);
    float radius = 0.45f * Math.Min(info.Width, info.Height);

    SKPath path = new SKPath
    {
        FillType = (SKPathFillType)fillTypePicker.SelectedItem
    };
    path.MoveTo(info.Width / 2, info.Height / 2 - radius);

    for (int i = 1; i < 5; i++)
    {
        // angle from vertical
        double angle = i * 4 * Math.PI / 5;
        path.LineTo(center + new SKPoint(radius * (float)Math.Sin(angle),
                                        -radius * (float)Math.Cos(angle)));
    }
    path.Close();

    SKPaint strokePaint = new SKPaint
    {
        Style = SKPaintStyle.Stroke,
        Color = SKColors.Red,
        StrokeWidth = 50,
        StrokeJoin = SKStrokeJoin.Round
    };

    SKPaint fillPaint = new SKPaint
    {
        Style = SKPaintStyle.Fill,
        Color = SKColors.Blue
    };

    switch ((string)drawingModePicker.SelectedItem)
    {
        case "Fill only":
            canvas.DrawPath(path, fillPaint);
            break;

        case "Stroke only":
            canvas.DrawPath(path, strokePaint);
            break;

        case "Stroke then Fill":
            canvas.DrawPath(path, strokePaint);
            canvas.DrawPath(path, fillPaint);
            break;

        case "Fill then Stroke":
            canvas.DrawPath(path, fillPaint);
            canvas.DrawPath(path, strokePaint);
            break;
    }
}
```

通常、パスの塗りつぶしの種類は塗りつぶしにのみ影響し、ストロークは影響しませんが、2つの `Inverse` モードは塗りつぶしとストロークの両方に影響します。 塗りつぶしの場合、2つの `Inverse` の種類は oppositely の外側の領域を塗りつぶすように塗りつぶします。 ストロークの場合、2つの `Inverse` の種類はストロークを除くすべての色になります。 これらの逆の塗りつぶしの種類を使用すると、iOS のスクリーン ショットに示すように、奇数の効果をいくつかが生成することができます。

[![](fill-types-images/fivepointedstar-small.png "Triple screenshot of the Five-Pointed Star page")](fill-types-images/fivepointedstar-large.png#lightbox "Triple screenshot of the Five-Pointed Star page")

Android のスクリーンショットには、通常の奇数とワインディングの効果が示されていますが、ストロークと塗りつぶしの順序も結果に影響します。

ワインディングのアルゴリズムでは、線が描画される方向に依存します。 通常、パスを作成する際を制御できますその方向の行が 1 つの点から描画される間を指定します。 ただし、`SKPath` クラスは、輪郭全体を描画する `AddRect` や `AddCircle` などのメソッドも定義します。 これらのオブジェクトの描画方法を制御するために、メソッドには[`SKPathDirection`](xref:SkiaSharp.SKPathDirection)型のパラメーターが含まれています。このパラメーターには、次の2つのメンバーがあります。

- `Clockwise`
- `CounterClockwise`

`SKPathDirection` パラメーターを含む `SKPath` 内のメソッドには、`Clockwise`の既定値が与えられます。

**重なった円**ページにより、偶数の円で始まるパスが作成されます。

```csharp
void OnCanvasViewPaintSurface(object sender, SKPaintSurfaceEventArgs args)
{
    SKImageInfo info = args.Info;
    SKSurface surface = args.Surface;
    SKCanvas canvas = surface.Canvas;

    canvas.Clear();

    SKPoint center = new SKPoint(info.Width / 2, info.Height / 2);
    float radius = Math.Min(info.Width, info.Height) / 4;

    SKPath path = new SKPath
    {
        FillType = SKPathFillType.EvenOdd
    };

    path.AddCircle(center.X - radius / 2, center.Y - radius / 2, radius);
    path.AddCircle(center.X - radius / 2, center.Y + radius / 2, radius);
    path.AddCircle(center.X + radius / 2, center.Y - radius / 2, radius);
    path.AddCircle(center.X + radius / 2, center.Y + radius / 2, radius);

    SKPaint paint = new SKPaint()
    {
        Style = SKPaintStyle.Fill,
        Color = SKColors.Cyan
    };

    canvas.DrawPath(path, paint);

    paint.Style = SKPaintStyle.Stroke;
    paint.StrokeWidth = 10;
    paint.Color = SKColors.Magenta;

    canvas.DrawPath(path, paint);
}
```

最小限のコードで作成した興味深いイメージであります。

[![](fill-types-images/overlappingcircles-small.png "Triple screenshot of the Overlapping Circles page")](fill-types-images/overlappingcircles-large.png#lightbox "Triple screenshot of the Overlapping Circles page")

## <a name="related-links"></a>関連リンク

- [SkiaSharp Api](https://docs.microsoft.com/dotnet/api/skiasharp)
- [SkiaSharpFormsDemos (サンプル)](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/skiasharpforms-demos)
