---
title: Xamarin. iOS のコレクションビュー
description: コレクションビューでは、任意のレイアウトを使用してコンテンツを表示できます。 これにより、カスタムレイアウトもサポートしながら、すぐに使用できるグリッドに似たレイアウトを簡単に作成できます。
ms.prod: xamarin
ms.assetid: F4B85F25-0CB5-4FEA-A3B5-D22FCDC81AE4
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 03/20/2017
ms.openlocfilehash: b7f8452f0f085a8a15f188534851e8926d13f377
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73021839"
---
# <a name="collection-views-in-xamarinios"></a>Xamarin. iOS のコレクションビュー

_コレクションビューでは、任意のレイアウトを使用してコンテンツを表示できます。これにより、カスタムレイアウトもサポートしながら、すぐに使用できるグリッドに似たレイアウトを簡単に作成できます。_

`UICollectionView` クラスで使用できるコレクションビューは、レイアウトを使用して複数の項目を画面に表示する、iOS 6 の新しい概念です。 `UICollectionView` にデータを提供して項目を作成し、それらの項目を操作するパターンは、iOS 開発でよく使用されるものと同じ委任とデータソースパターンに従います。

ただし、コレクションビューは、`UICollectionView` 自体に依存しないレイアウトサブシステムで動作します。 そのため、別のレイアウトを指定するだけで、コレクションビューの表示を簡単に変更できます。

iOS には `UICollectionViewFlowLayout` というレイアウトクラスが用意されており、これを使用すると、追加の作業なしでグリッドなどの行ベースのレイアウトを作成できます。 また、想像できるプレゼンテーションを可能にするカスタムレイアウトを作成することもできます。

## <a name="uicollectionview-basics"></a>UICollectionView の基礎

`UICollectionView` クラスは、次の3つの異なる項目で構成されます。

- **セル**–各項目のデータドリブンビュー
- **補助ビュー** –セクションに関連付けられているデータドリブンビュー。
- **装飾ビュー** –レイアウトによって作成される非データドリブンビュー

## <a name="cells"></a>セル

セルは、コレクションビューによって表示されるデータセット内の1つのアイテムを表すオブジェクトです。 各セルは `UICollectionViewCell` クラスのインスタンスであり、次の図に示すように、3つの異なるビューで構成されています。

 [![](uicollectionview-images/01-uicollectionviewcell.png "Each cell is composed of three different views, as shown here")](uicollectionview-images/01-uicollectionviewcell.png#lightbox)

`UICollectionViewCell` クラスには、これらの各ビューに対して次のプロパティがあります。

- `ContentView`: このビューには、セルに表示されるコンテンツが含まれます。 画面の最上位の z オーダーで表示されます。
- `SelectedBackgroundView` –セルには選択のサポートが組み込まれています。 このビューは、セルが選択されていることを視覚的に示すために使用されます。 セルが選択されると、`ContentView` のすぐ下に表示されます。
- `BackgroundView` –セルには、`BackgroundView` によって表示される背景を表示することもできます。 このビューは、`SelectedBackgroundView` の下に表示されます。

`BackgroundView` と `SelectedBackgroundView`よりも小さいように `ContentView` を設定することにより、`BackgroundView` を使用してコンテンツを視覚的に示すことができますが、次に示すように、セルを選択すると `SelectedBackgroundView` が表示されます。:

 [![](uicollectionview-images/02-cells.png "The different cell elements")](uicollectionview-images/02-cells.png#lightbox)

上のスクリーンショットのセルは、次のコードに示すように、`UICollectionViewCell` から継承し、`ContentView`、`SelectedBackgroundView`、および `BackgroundView` の各プロパティをそれぞれ設定することによって作成されます。

```csharp
public class AnimalCell : UICollectionViewCell
{
        UIImageView imageView;

        [Export ("initWithFrame:")]
        public AnimalCell (CGRect frame) : base (frame)
        {
            BackgroundView = new UIView{BackgroundColor = UIColor.Orange};

            SelectedBackgroundView = new UIView{BackgroundColor = UIColor.Green};

            ContentView.Layer.BorderColor = UIColor.LightGray.CGColor;
            ContentView.Layer.BorderWidth = 2.0f;
            ContentView.BackgroundColor = UIColor.White;
            ContentView.Transform = CGAffineTransform.MakeScale (0.8f, 0.8f);

            imageView = new UIImageView (UIImage.FromBundle ("placeholder.png"));
            imageView.Center = ContentView.Center;
            imageView.Transform = CGAffineTransform.MakeScale (0.7f, 0.7f);

            ContentView.AddSubview (imageView);
        }

        public UIImage Image {
            set {
                imageView.Image = value;
            }
        }
}
```

 <a name="Supplementary_Views" />

## <a name="supplementary-views"></a>補助ビュー

補助ビューは、`UICollectionView`の各セクションに関連付けられている情報を表示するビューです。 補助ビューは、セルと同様にデータドリブンです。 データソースからのアイテムデータをセルに表示する補助ビューには、ブックのカテゴリや音楽ライブラリ内の音楽のジャンルなどのセクションデータが表示されます。

たとえば、次の図に示すように、補助ビューを使用して特定のセクションのヘッダーを表示することができます。

 [![](uicollectionview-images/02a-supplementary-view.png "A Supplementary View used to present a header for a particular section, as shown here")](uicollectionview-images/02a-supplementary-view.png#lightbox)

補助ビューを使用するには、まず、`ViewDidLoad` メソッドに登録する必要があります。

```csharp
CollectionView.RegisterClassForSupplementaryView (typeof(Header), UICollectionElementKindSection.Header, headerId);
```

次に、`DequeueReusableSupplementaryView`を使用して作成した `GetViewForSupplementaryElement`を使用してビューを返し、`UICollectionReusableView`から継承する必要があります。 次のコードスニペットでは、上記のスクリーンショットに示されている SupplementaryView が生成されます。

```csharp
public override UICollectionReusableView GetViewForSupplementaryElement (UICollectionView collectionView, NSString elementKind, NSIndexPath indexPath)
        {
            var headerView = (Header)collectionView.DequeueReusableSupplementaryView (elementKind, headerId, indexPath);
            headerView.Text = "Supplementary View";
            return headerView;
        }

```

補助ビューは、単なるヘッダーおよびフッターよりも一般的です。
コレクションビュー内の任意の場所に配置することができ、任意のビューを構成して、外観を完全にカスタマイズできます。

 <a name="Decoration_Views" />

## <a name="decoration-views"></a>装飾ビュー

装飾ビューは、単に `UICollectionView`に表示できる視覚的なビューです。 セルや補助ビューとは異なり、データドリブンではありません。 これらは常にレイアウトのサブクラス内に作成され、その後、コンテンツのレイアウトとして変更できます。 たとえば、次に示すように、装飾ビューを使用すると、`UICollectionView`のコンテンツと共にスクロールする背景ビューを表示できます。

 [![](uicollectionview-images/02c-decoration-view.png "Decoration View with a red background")](uicollectionview-images/02c-decoration-view.png#lightbox)

 次のコードスニペットは、サンプル `CircleLayout` クラスの背景を赤に変更します。

 ```csharp
 public class MyDecorationView : UICollectionReusableView
  {
    [Export ("initWithFrame:")]
    public MyDecorationView (CGRect frame) : base (frame)
    {
      BackgroundColor = UIColor.Red;
    }
  }
 ```

## <a name="data-source"></a>データ ソース

`UITableView` や `MKMapView`などの iOS の他の部分と同様に、*データソース*からデータを取得 `UICollectionView` ます。これは、 **`UICollectionViewDataSource`** クラスを介して Xamarin で公開されています。 このクラスは、次のような `UICollectionView` にコンテンツを提供します。

- **セル**– `GetCell` メソッドから返されます。
- **補助ビュー** – `GetViewForSupplementaryElement` メソッドから返されます。
- `NumberOfSections` メソッドから返された**セクションの数**。 実装されていない場合、既定値は1です。
- **セクションごとの項目**数-`GetItemsCount` メソッドから返されます。

### <a name="uicollectionviewcontroller"></a>UICollectionViewController
便宜上、`UICollectionViewController` クラスを使用できます。これは、次のセクションで説明するデリゲートと、`UICollectionView` ビューのデータソースの両方に自動的に構成されます。

`UITableView`と同様に、`UICollectionView` クラスは、そのデータソースを呼び出して、画面上の項目のセルを取得します。
画面の外にスクロールするセルは、次の図に示すように、再利用のためにキューに配置されます。

 [![](uicollectionview-images/03-cell-reuse.png "Cells that scroll off the screen are placed in to a queue for reuse as shown here")](uicollectionview-images/03-cell-reuse.png#lightbox)

`UICollectionView` と `UITableView`により、セルの再利用が簡略化されました。 セルがシステムに登録されているため、再利用キューで使用できない場合、データソースに直接セルを作成する必要はなくなりました。 再利用キューからセルを逆キューにする呼び出しを行うときにセルを使用できない場合、iOS では、登録された種類または nib に基づいて、そのセルが自動的に作成されます。
補助ビューでも同じ手法を使用できます。

たとえば、`AnimalCell` クラスを登録する次のコードについて考えてみます。

```csharp
static NSString animalCellId = new NSString ("AnimalCell");
CollectionView.RegisterClassForCell (typeof(AnimalCell), animalCellId);
```

`UICollectionView` の項目が画面上にあるためにセルが必要な場合、`UICollectionView` はそのデータソースの `GetCell` メソッドを呼び出します。 この方法は、UITableView での動作と同様に、この方法では、バッキングデータからセルを構成します。この場合の `AnimalCell` クラスです。

次のコードは、`AnimalCell` インスタンスを返す `GetCell` の実装を示しています。

```csharp
public override UICollectionViewCell GetCell (UICollectionView collectionView, Foundation.NSIndexPath indexPath)
{
        var animalCell = (AnimalCell)collectionView.DequeueReusableCell (animalCellId, indexPath);

        var animal = animals [indexPath.Row];

        animalCell.Image = animal.Image;

        return animalCell;
}
```

`DequeReusableCell` の呼び出しでは、セルが再利用キューから解除されるか、またはキューでセルが使用できない場合に、`CollectionView.RegisterClassForCell`の呼び出しに登録された型に基づいて作成されます。

この場合、`AnimalCell` クラスを登録することによって、iOS は新しい `AnimalCell` を内部的に作成し、セルを逆キューにする呼び出しが行われたときにそれを返します。これにより、animal クラスに含まれるイメージで構成され、`UICollectionView`に表示されるように返されます。

 <a name="Delegate" />

### <a name="delegate"></a>delegate

`UICollectionView` クラスは、型 `UICollectionViewDelegate` のデリゲートを使用して、`UICollectionView`内のコンテンツとの対話をサポートします。 これにより、次の制御が可能になります。

- **セルの選択**–セルが選択されているかどうかを判断します。
- **セルの強調表示**–セルが現在タッチされているかどうかを判断します。
- **セルメニュー** –長いプレスジェスチャに応じてセルに表示されるメニューです。

データソースと同様に、`UICollectionViewController` は既定で `UICollectionView`のデリゲートとして構成されます。

 <a name="Cell_HighLighting" />

#### <a name="cell-highlighting"></a>セルの強調表示

セルを押すと、セルは強調表示された状態に遷移し、ユーザーがセルから指を離したときに選択されません。 これにより、実際に選択される前に、セルの外観を一時的に変更することができます。 選択すると、セルの `SelectedBackgroundView` が表示されます。 次の図は、選択が発生する直前の状態を示しています。

 [![](uicollectionview-images/04-cell-highlight.png "This figure shows the highlighted state just before the selection occurs")](uicollectionview-images/04-cell-highlight.png#lightbox)

強調表示を実装するには、`UICollectionViewDelegate` の `ItemHighlighted` および `ItemUnhighlighted` メソッドを使用できます。 たとえば、次のコードでは、上の図に示すように、セルが強調表示されている場合には `ContentView` の背景が黄色になり、強調表示されていない場合は白い背景が適用されます。

```csharp
public override void ItemHighlighted (UICollectionView collectionView, NSIndexPath indexPath)
{
        var cell = collectionView.CellForItem(indexPath);
        cell.ContentView.BackgroundColor = UIColor.Yellow;
}

public override void ItemUnhighlighted (UICollectionView collectionView, NSIndexPath indexPath)
{
        var cell = collectionView.CellForItem(indexPath);
        cell.ContentView.BackgroundColor = UIColor.White;
}
```

 <a name="Disabling_Selection" />

#### <a name="disabling-selection"></a>無効化 (選択を)

`UICollectionView`では、既定で選択が有効になっています。 選択を無効にするには、次のように `ShouldHighlightItem` をオーバーライドし、false を返します。

```csharp
public override bool ShouldHighlightItem (UICollectionView collectionView, NSIndexPath indexPath)
{
        return false;
}
```

強調表示が無効になっている場合は、セルを選択するプロセスも無効になります。 また、`ShouldHighlightItem` を実装して false を返す場合でも、選択を直接制御する `ShouldSelectItem` メソッドもありますが、`ShouldSelectItem` は呼び出されません。

 `ShouldSelectItem` を使用すると、`ShouldHighlightItem` が実装されていない場合に、項目ごとに選択をオンまたはオフにすることができます。 また、`ShouldHighlightItem` が実装されていて true を返し、`ShouldSelectItem` が false を返す場合は、選択せずに強調表示することもできます。

 <a name="Cell_Menus" />

#### <a name="cell-menus"></a>セルメニュー

`UICollectionView` 内の各セルは、必要に応じて、切り取り、コピー、および貼り付けを許可するメニューを表示できます。 セルに [編集] メニューを作成するには、次のようにします。

1. `ShouldShowMenu` をオーバーライドし、項目にメニューを表示する必要がある場合は true を返します。
1. `CanPerformAction` をオーバーライドし、項目が実行できるすべてのアクション (切り取り、コピー、または貼り付け) に対して true を返します。
1. `PerformAction` をオーバーライドして、編集、貼り付け操作のコピーを実行します。

次のスクリーンショットは、セルが長押されたときのメニューを示しています。

 [![](uicollectionview-images/04a-menu.png "This screenshot show the menu when a cell is long pressed")](uicollectionview-images/04a-menu.png#lightbox)

 <a name="Layout" />

## <a name="layout"></a>レイアウト

`UICollectionView` は、すべての要素、セル、補助ビュー、および装飾ビューの配置を、`UICollectionView` 自体とは別に管理できるレイアウトシステムをサポートしています。
レイアウトシステムを使用すると、アプリケーションは、この記事で説明したようなレイアウトをサポートしたり、カスタムレイアウトを提供したりすることができます。

 <a name="Layout_Basics" />

### <a name="layout-basics"></a>レイアウトの基本

`UICollectionView` 内のレイアウトは、`UICollectionViewLayout`から継承するクラスで定義されます。 レイアウト実装は、`UICollectionView`内のすべての項目のレイアウト属性を作成する役割を担います。 レイアウトを作成するには、次の2つの方法があります。

- 組み込みの `UICollectionViewFlowLayout` を使用します。
- `UICollectionViewLayout` から継承することにより、カスタムレイアウトを提供します。

 <a name="Flow_Layout" />

### <a name="flow-layout"></a>フロー レイアウト

`UICollectionViewFlowLayout` クラスは、表示されているセルのグリッド内のコンテンツの整列に適した線ベースのレイアウトを提供します。

フローレイアウトを使用するには:

- `UICollectionViewFlowLayout` のインスタンスを作成します。

```csharp
var layout = new UICollectionViewFlowLayout ();
```

- インスタンスを `UICollectionView` のコンストラクターに渡します。

```csharp
simpleCollectionViewController = new SimpleCollectionViewController (layout);
```

これは、グリッド内のコンテンツをレイアウトするために必要なすべてのものです。 また、次に示すように、向きが変わると、`UICollectionViewFlowLayout` によってコンテンツの再配置が適切に処理されます。

 [![](uicollectionview-images/05-layout-orientation.png "Example of the orientation changes")](uicollectionview-images/05-layout-orientation.png#lightbox)

 <a name="Section_Inset" />

#### <a name="section-inset"></a>セクションの埋め込み

`UIContentView`の周りに領域を確保するために、レイアウトには `UIEdgeInsets`型の `SectionInset` プロパティがあります。 たとえば、次のコードは、`UICollectionViewFlowLayout`によってレイアウトされた場合に、`UIContentView` の各セクションに50ピクセルのバッファーを提供します。

```csharp
var layout = new UICollectionViewFlowLayout ();
layout.SectionInset = new UIEdgeInsets (50,50,50,50);
```

この結果、次のように、セクションの周囲にスペースが表示されます。

 [![](uicollectionview-images/06-sectioninset.png "Spacing around the section as shown here")](uicollectionview-images/06-sectioninset.png#lightbox)

 <a name="Subclassing_UICollectionViewFlowLayout" />

#### <a name="subclassing-uicollectionviewflowlayout"></a>UICollectionViewFlowLayout のサブクラス化

`UICollectionViewFlowLayout` を直接使用するためのエディションでは、サブクラス化して、線に沿ったコンテンツのレイアウトをさらにカスタマイズすることもできます。 たとえば、次に示すように、この方法を使用して、セルをグリッドにラップしないレイアウトを作成できます。代わりに、次に示すように、水平スクロール効果を使用して1つの行を作成します。

 [![](uicollectionview-images/07-line-layout.png "A single row with a horizontal scrolling effect")](uicollectionview-images/07-line-layout.png#lightbox)

をサブクラス化して実装するには `UICollectionViewFlowLayout` が必要です。

- レイアウト自体またはコンストラクター内のレイアウト内のすべての項目に適用されるレイアウトプロパティを初期化します。
- `ShouldInvalidateLayoutForBoundsChange` をオーバーライドして true を返し、`UICollectionView` の範囲が変更されたときにセルのレイアウトが再計算されるようにします。 この場合、中央のセルに適用される変換のコードがスクロール中に適用されることを確認します。
- `TargetContentOffset` をオーバーライドして、中央のセルがスクロールを停止したときに `UICollectionView` の中央にスナップされるようにします。
- `LayoutAttributesForElementsInRect` をオーバーライドして `UICollectionViewLayoutAttributes` の配列を返します。 各 `UICollectionViewLayoutAttribute` には、`Center`、`Size`、`ZIndex`、`Transform3D` などのプロパティを含む、特定の項目のレイアウト方法に関する情報が含まれています。

次のコードは、このような実装を示しています。

```csharp
using System;
using CoreGraphics;
using Foundation;
using UIKit;
using CoreGraphics;
using CoreAnimation;

namespace SimpleCollectionView
{
  public class LineLayout : UICollectionViewFlowLayout
  {
    public const float ITEM_SIZE = 200.0f;
    public const int ACTIVE_DISTANCE = 200;
    public const float ZOOM_FACTOR = 0.3f;

    public LineLayout ()
    {
      ItemSize = new CGSize (ITEM_SIZE, ITEM_SIZE);
      ScrollDirection = UICollectionViewScrollDirection.Horizontal;
            SectionInset = new UIEdgeInsets (400,0,400,0);
      MinimumLineSpacing = 50.0f;
    }

    public override bool ShouldInvalidateLayoutForBoundsChange (CGRect newBounds)
    {
      return true;
    }

    public override UICollectionViewLayoutAttributes[] LayoutAttributesForElementsInRect (CGRect rect)
    {
      var array = base.LayoutAttributesForElementsInRect (rect);
            var visibleRect = new CGRect (CollectionView.ContentOffset, CollectionView.Bounds.Size);

      foreach (var attributes in array) {
        if (attributes.Frame.IntersectsWith (rect)) {
          float distance = (float)(visibleRect.GetMidX () - attributes.Center.X);
          float normalizedDistance = distance / ACTIVE_DISTANCE;
          if (Math.Abs (distance) < ACTIVE_DISTANCE) {
            float zoom = 1 + ZOOM_FACTOR * (1 - Math.Abs (normalizedDistance));
            attributes.Transform3D = CATransform3D.MakeScale (zoom, zoom, 1.0f);
            attributes.ZIndex = 1;
          }
        }
      }
      return array;
    }

    public override CGPoint TargetContentOffset (CGPoint proposedContentOffset, CGPoint scrollingVelocity)
    {
      float offSetAdjustment = float.MaxValue;
      float horizontalCenter = (float)(proposedContentOffset.X + (this.CollectionView.Bounds.Size.Width / 2.0));
      CGRect targetRect = new CGRect (proposedContentOffset.X, 0.0f, this.CollectionView.Bounds.Size.Width, this.CollectionView.Bounds.Size.Height);
      var array = base.LayoutAttributesForElementsInRect (targetRect);
      foreach (var layoutAttributes in array) {
        float itemHorizontalCenter = (float)layoutAttributes.Center.X;
        if (Math.Abs (itemHorizontalCenter - horizontalCenter) < Math.Abs (offSetAdjustment)) {
          offSetAdjustment = itemHorizontalCenter - horizontalCenter;
        }
      }
            return new CGPoint (proposedContentOffset.X + offSetAdjustment, proposedContentOffset.Y);
    }

  }
}
```

 <a name="Custom_Layout" />

### <a name="custom-layout"></a>カスタム レイアウト

`UICollectionViewFlowLayout`の使用に加えて、`UICollectionViewLayout`から直接継承することによってレイアウトを完全にカスタマイズすることもできます。

オーバーライドする主な方法は次のとおりです。

- `PrepareLayout` –レイアウトプロセス全体で使用される最初の幾何計算を実行するために使用されます。
- `CollectionViewContentSize` –コンテンツの表示に使用される領域のサイズを返します。
- `LayoutAttributesForElementsInRect` –前に示した UICollectionViewFlowLayout の例と同様に、このメソッドを使用して、各項目のレイアウト方法に関する情報を `UICollectionView` に提供します。 ただし、`UICollectionViewFlowLayout` とは異なり、カスタムレイアウトを作成する場合は、選択した項目を配置できます。

たとえば、次に示すように、同じコンテンツを円形のレイアウトで表示できます。

 [![](uicollectionview-images/08-circle-layout.png "A circular custom layout as shown here")](uicollectionview-images/08-circle-layout.png#lightbox)

レイアウトに関する強力な点は、グリッドに似たレイアウトから水平スクロールレイアウトに変更することです。その後、この循環レイアウトには、`UICollectionView` に提供されたレイアウトクラスのみを変更する必要があります。 `UICollectionView`には何もありません。そのデリゲートまたはデータソースコードはまったく変更されません。

## <a name="changes-in-ios-9"></a>IOS 9 での変更点

IOS 9 では、コレクションビュー (`UICollectionView`) で、新しい既定のジェスチャレコグナイザーといくつかの新しいサポートメソッドを追加することによって、すぐに項目をドラッグすることがサポートされるようになりました。

これらの新しいメソッドを使用すると、簡単にドラッグしてコレクションビュー内で順序を変更できます。また、並べ替え処理のいずれかの段階でアイテムの外観をカスタマイズすることもできます。

[![](uicollectionview-images/intro01.png "An example of the reordering process")](uicollectionview-images/intro01.png#lightbox)

この記事では、Xamarin. iOS アプリケーションでのドラッグツーオーダーの実装に加え、iOS 9 がコレクションビューコントロールに対して行ったその他の変更についても説明します。

- [項目を簡単に並べ替える](#Easy-Reordering-of-Items)
  - [単純な並べ替えの例](#Simple-Reordering-Example)
  - [カスタムジェスチャ認識エンジンの使用](#Using-a-Custom-Gesture-Recognizer)
  - [カスタムレイアウトと並べ替え](#Custom-Layouts-and-Reording)
- [コレクションビューの変更](#collection-view-changes)

<a name="Easy-Reordering-of-Items" />

## <a name="reordering-of-items"></a>項目の並べ替え

前述のように、iOS 9 でのコレクションビューの最も重要な変更点の1つは、すぐにドラッグして順序を変更する機能が追加されたことでした。

IOS 9 では、コレクションビューに並べ替えを追加する最も簡単な方法は、`UICollectionViewController`を使用することです。
コレクションビューコントローラーに `InstallsStandardGestureForInteractiveMovement` プロパティが追加されました。これにより、コレクション内の項目の順序を変更するためのドラッグをサポートする標準*ジェスチャレコグナイザー*が追加されます。
既定値は `true`ので、`UICollectionViewDataSource` クラスの `MoveItem` メソッドを実装して、ドラッグから順序変更をサポートする必要があります。 (例:

```csharp
public override void MoveItem (UICollectionView collectionView, NSIndexPath sourceIndexPath, NSIndexPath destinationIndexPath)
{
  // Reorder our list of items
  ...
}
```

<a name="Simple-Reordering-Example" />

### <a name="simple-reordering-example"></a>単純な並べ替えの例

簡単な例として、新しい Xamarin. iOS プロジェクトを開始し、**メインの storyboard**ファイルを編集します。 `UICollectionViewController` をデザイン画面にドラッグします。

[![](uicollectionview-images/quick01.png "Adding a UICollectionViewController")](uicollectionview-images/quick01.png#lightbox)

コレクションビューを選択します (これはドキュメントアウトラインから簡単に実行できます)。 次のスクリーンショットに示すように、Properties Pad の [レイアウト] タブで、次のサイズを設定します。

- **セルのサイズ**: 幅– 60 |高さ–60
- **ヘッダーのサイズ**: 幅– 0 |高さ–0
- **フッターのサイズ**: 幅– 0 |高さ–0
- **最小間隔**: セルの場合– 8 |行の場合–8
- **セクションのインセット**: 上-16 |下-16 |左-16 |右-16

[![](uicollectionview-images/quick04.png "Set the Collection View sizes")](uicollectionview-images/quick04.png#lightbox)

次に、既定のセルを編集します。

- 背景色を青に変更する
- セルのタイトルとして機能するラベルを追加します。
- 再利用識別子を**セル**に設定する

[![](uicollectionview-images/quick02.png "Edit the default Cell")](uicollectionview-images/quick02.png#lightbox)

サイズを変更するときにラベルをセルの中央に維持するための制約を追加します。

次のように、 _Collectionviewcell_の**プロパティパッド**で、**クラス**を `TextCollectionViewCell`に設定します。

[![](uicollectionview-images/quick05.png "Set the Class to TextCollectionViewCell")](uicollectionview-images/quick05.png#lightbox)

コレクションの**再利用可能なビュー**を `Cell`に設定します。

[![](uicollectionview-images/quick06.png "Set the Collection Reusable View to Cell")](uicollectionview-images/quick06.png#lightbox)

最後に、ラベルを選択し、`TextLabel`という名前を付けます。

[![](uicollectionview-images/quick07.png "name label TextLabel")](uicollectionview-images/quick07.png#lightbox)

`TextCollectionViewCell` クラスを編集し、次のプロパティを追加します。

```csharp
using System;
using Foundation;
using UIKit;

namespace CollectionView
{
  public partial class TextCollectionViewCell : UICollectionViewCell
  {
    #region Computed Properties
    public string Title {
      get { return TextLabel.Text; }
      set { TextLabel.Text = value; }
    }
    #endregion

    #region Constructors
    public TextCollectionViewCell (IntPtr handle) : base (handle)
    {
    }
    #endregion
  }
}
```

ここでは、ラベルの `Text` プロパティがセルのタイトルとして公開されるため、コードから設定できます。

新しいC#クラスをプロジェクトに追加し、`WaterfallCollectionSource`呼び出します。 ファイルを編集し、次のように表示します。

```csharp
using System;
using Foundation;
using UIKit;
using System.Collections.Generic;

namespace CollectionView
{
  public class WaterfallCollectionSource : UICollectionViewDataSource
  {
    #region Computed Properties
    public WaterfallCollectionView CollectionView { get; set;}
    public List<int> Numbers { get; set; } = new List<int> ();
    #endregion

    #region Constructors
    public WaterfallCollectionSource (WaterfallCollectionView collectionView)
    {
      // Initialize
      CollectionView = collectionView;

      // Init numbers collection
      for (int n = 0; n < 100; ++n) {
        Numbers.Add (n);
      }
    }
    #endregion

    #region Override Methods
    public override nint NumberOfSections (UICollectionView collectionView) {
      // We only have one section
      return 1;
    }

    public override nint GetItemsCount (UICollectionView collectionView, nint section) {
      // Return the number of items
      return Numbers.Count;
    }

    public override UICollectionViewCell GetCell (UICollectionView collectionView, NSIndexPath indexPath)
    {
      // Get a reusable cell and set {~~it's~>its~~} title from the item
      var cell = collectionView.DequeueReusableCell ("Cell", indexPath) as TextCollectionViewCell;
      cell.Title = Numbers [(int)indexPath.Item].ToString();

      return cell;
    }

    public override bool CanMoveItem (UICollectionView collectionView, NSIndexPath indexPath) {
      // We can always move items
      return true;
    }

    public override void MoveItem (UICollectionView collectionView, NSIndexPath sourceIndexPath, NSIndexPath destinationIndexPath)
    {
      // Reorder our list of items
      var item = Numbers [(int)sourceIndexPath.Item];
      Numbers.RemoveAt ((int)sourceIndexPath.Item);
      Numbers.Insert ((int)destinationIndexPath.Item, item);
    }
    #endregion
  }
}
```

このクラスは、コレクションビューのデータソースになり、コレクション内の各セルに関する情報を提供します。
`MoveItem` メソッドが実装されていることに注意してください。これにより、コレクション内の項目をドラッグして並べ替えることができます。

別の新しいC#クラスをプロジェクトに追加し、`WaterfallCollectionDelegate`呼び出します。 このファイルを編集して、次のように表示します。

```csharp
using System;
using Foundation;
using UIKit;
using System.Collections.Generic;

namespace CollectionView
{
  public class WaterfallCollectionDelegate : UICollectionViewDelegate
  {
    #region Computed Properties
    public WaterfallCollectionView CollectionView { get; set;}
    #endregion

    #region Constructors
    public WaterfallCollectionDelegate (WaterfallCollectionView collectionView)
    {

      // Initialize
      CollectionView = collectionView;

    }
    #endregion

    #region Overrides Methods
    public override bool ShouldHighlightItem (UICollectionView collectionView, NSIndexPath indexPath) {
      // Always allow for highlighting
      return true;
    }

    public override void ItemHighlighted (UICollectionView collectionView, NSIndexPath indexPath)
    {
      // Get cell and change to green background
      var cell = collectionView.CellForItem(indexPath);
      cell.ContentView.BackgroundColor = UIColor.FromRGB(183,208,57);
    }

    public override void ItemUnhighlighted (UICollectionView collectionView, NSIndexPath indexPath)
    {
      // Get cell and return to blue background
      var cell = collectionView.CellForItem(indexPath);
      cell.ContentView.BackgroundColor = UIColor.FromRGB(164,205,255);
    }
    #endregion
  }
}
```

これは、コレクションビューのデリゲートとして機能します。 メソッドは、ユーザーがコレクションビューでセルを操作するときに、そのセルを強調表示するためにオーバーライドされています。

プロジェクトに最後C#のクラスを1つ追加し、`WaterfallCollectionView`呼び出します。 このファイルを編集して、次のように表示します。

```csharp
using System;
using UIKit;
using System.Collections.Generic;
using Foundation;

namespace CollectionView
{
  [Register("WaterfallCollectionView")]
  public class WaterfallCollectionView : UICollectionView
  {

    #region Constructors
    public WaterfallCollectionView (IntPtr handle) : base (handle)
    {
    }
    #endregion

    #region Override Methods
    public override void AwakeFromNib ()
    {
      base.AwakeFromNib ();

      // Initialize
      DataSource = new WaterfallCollectionSource(this);
      Delegate = new WaterfallCollectionDelegate(this);

    }
    #endregion
  }
}
```

前の手順で作成した `DataSource` と `Delegate` は、そのストーリーボード (または**xib**ファイル) からコレクションビューを構築するときに設定されることに注意してください。

メインの**ストーリーボード**ファイルをもう一度編集し、コレクションビューを選択して、**プロパティ**に切り替えます。 **クラス**を、前に定義したカスタム `WaterfallCollectionView` クラスに設定します。

UI に加えた変更を保存し、アプリを実行します。
ユーザーが一覧から項目を選択して新しい場所にドラッグした場合、他の項目は項目の外に移動するときに自動的にアニメーション化されます。
ユーザーが新しい場所に項目をドロップすると、その場所が表示されます。 (例:

[![](uicollectionview-images/intro01.png "An example of dragging an item to a new location")](uicollectionview-images/intro01.png#lightbox)

<a name="Using-a-Custom-Gesture-Recognizer" />

### <a name="using-a-custom-gesture-recognizer"></a>カスタムジェスチャ認識エンジンの使用

`UICollectionViewController` を使用できず、通常の `UIViewController`を使用する必要がある場合、またはドラッグアンドドロップジェスチャをより細かく制御する必要がある場合は、独自のカスタムジェスチャ認識エンジンを作成し、ビューが読み込まれたときにコレクションビューに追加できます。 (例:

```csharp
public override void ViewDidLoad ()
{
  base.ViewDidLoad ();

  // Create a custom gesture recognizer
  var longPressGesture = new UILongPressGestureRecognizer ((gesture) => {

    // Take action based on state
    switch(gesture.State) {
    case UIGestureRecognizerState.Began:
      var selectedIndexPath = CollectionView.IndexPathForItemAtPoint(gesture.LocationInView(View));
      if (selectedIndexPath !=null) {
        CollectionView.BeginInteractiveMovementForItem(selectedIndexPath);
      }
      break;
    case UIGestureRecognizerState.Changed:
      CollectionView.UpdateInteractiveMovementTargetPosition(gesture.LocationInView(View));
      break;
    case UIGestureRecognizerState.Ended:
      CollectionView.EndInteractiveMovement();
      break;
    default:
      CollectionView.CancelInteractiveMovement();
      break;
    }

  });

  // Add the custom recognizer to the collection view
  CollectionView.AddGestureRecognizer(longPressGesture);
}
```

ここでは、ドラッグ操作を実装および制御するために、コレクションビューに追加されたいくつかの新しいメソッドを使用しています。

- `BeginInteractiveMovementForItem`-移動操作の開始をマークします。
- `UpdateInteractiveMovementTargetPosition`-項目の場所が更新されると送信されます。
- `EndInteractiveMovement`-項目の移動の終了を示します。
- `CancelInteractiveMovement`-ユーザーに移動操作の取り消しをマークします。

アプリケーションを実行すると、ドラッグ操作は、コレクションビューに付属する既定のドラッグジェスチャレコグナイザーと同じように動作します。

<a name="Custom-Layouts-and-Reording" />

### <a name="custom-layouts-and-reordering"></a>カスタムレイアウトと並べ替え

IOS 9 では、コレクションビューでドラッグツーオーダーおよびカスタムレイアウトを操作するために、いくつかの新しいメソッドが追加されています。 この機能を調べるには、コレクションにカスタムレイアウトを追加してみましょう。

最初に、`WaterfallCollectionLayout`とC#いう名前の新しいクラスをプロジェクトに追加します。 編集し、次のようにします。

```csharp
using System;
using Foundation;
using UIKit;
using System.Collections.Generic;
using CoreGraphics;

namespace CollectionView
{
  [Register("WaterfallCollectionLayout")]
  public class WaterfallCollectionLayout : UICollectionViewLayout
  {
    #region Private Variables
    private int columnCount = 2;
    private nfloat minimumColumnSpacing = 10;
    private nfloat minimumInterItemSpacing = 10;
    private nfloat headerHeight = 0.0f;
    private nfloat footerHeight = 0.0f;
    private UIEdgeInsets sectionInset = new UIEdgeInsets(0, 0, 0, 0);
    private WaterfallCollectionRenderDirection itemRenderDirection = WaterfallCollectionRenderDirection.ShortestFirst;
    private Dictionary<nint,UICollectionViewLayoutAttributes> headersAttributes = new Dictionary<nint, UICollectionViewLayoutAttributes>();
    private Dictionary<nint,UICollectionViewLayoutAttributes> footersAttributes = new Dictionary<nint, UICollectionViewLayoutAttributes>();
    private List<CGRect> unionRects = new List<CGRect>();
    private List<nfloat> columnHeights = new List<nfloat>();
    private List<UICollectionViewLayoutAttributes> allItemAttributes = new List<UICollectionViewLayoutAttributes>();
    private List<List<UICollectionViewLayoutAttributes>> sectionItemAttributes = new List<List<UICollectionViewLayoutAttributes>>();
    private nfloat unionSize = 20;
    #endregion

    #region Computed Properties
    [Export("ColumnCount")]
    public int ColumnCount {
      get { return columnCount; }
      set {
        WillChangeValue ("ColumnCount");
        columnCount = value;
        DidChangeValue ("ColumnCount");

        InvalidateLayout ();
      }
    }

    [Export("MinimumColumnSpacing")]
    public nfloat MinimumColumnSpacing {
      get { return minimumColumnSpacing; }
      set {
        WillChangeValue ("MinimumColumnSpacing");
        minimumColumnSpacing = value;
        DidChangeValue ("MinimumColumnSpacing");

        InvalidateLayout ();
      }
    }

    [Export("MinimumInterItemSpacing")]
    public nfloat MinimumInterItemSpacing {
      get { return minimumInterItemSpacing; }
      set {
        WillChangeValue ("MinimumInterItemSpacing");
        minimumInterItemSpacing = value;
        DidChangeValue ("MinimumInterItemSpacing");

        InvalidateLayout ();
      }
    }

    [Export("HeaderHeight")]
    public nfloat HeaderHeight {
      get { return headerHeight; }
      set {
        WillChangeValue ("HeaderHeight");
        headerHeight = value;
        DidChangeValue ("HeaderHeight");

        InvalidateLayout ();
      }
    }

    [Export("FooterHeight")]
    public nfloat FooterHeight {
      get { return footerHeight; }
      set {
        WillChangeValue ("FooterHeight");
        footerHeight = value;
        DidChangeValue ("FooterHeight");

        InvalidateLayout ();
      }
    }

    [Export("SectionInset")]
    public UIEdgeInsets SectionInset {
      get { return sectionInset; }
      set {
        WillChangeValue ("SectionInset");
        sectionInset = value;
        DidChangeValue ("SectionInset");

        InvalidateLayout ();
      }
    }

    [Export("ItemRenderDirection")]
    public WaterfallCollectionRenderDirection ItemRenderDirection {
      get { return itemRenderDirection; }
      set {
        WillChangeValue ("ItemRenderDirection");
        itemRenderDirection = value;
        DidChangeValue ("ItemRenderDirection");

        InvalidateLayout ();
      }
    }
    #endregion

    #region Constructors
    public WaterfallCollectionLayout ()
    {
    }

    public WaterfallCollectionLayout(NSCoder coder) : base(coder) {

    }
    #endregion

    #region Public Methods
    public nfloat ItemWidthInSectionAtIndex(int section) {

      var width = CollectionView.Bounds.Width - SectionInset.Left - SectionInset.Right;
      return (nfloat)Math.Floor ((width - ((ColumnCount - 1) * MinimumColumnSpacing)) / ColumnCount);
    }
    #endregion

    #region Override Methods
    public override void PrepareLayout ()
    {
      base.PrepareLayout ();

      // Get the number of sections
      var numberofSections = CollectionView.NumberOfSections();
      if (numberofSections == 0)
        return;

      // Reset collections
      headersAttributes.Clear ();
      footersAttributes.Clear ();
      unionRects.Clear ();
      columnHeights.Clear ();
      allItemAttributes.Clear ();
      sectionItemAttributes.Clear ();

      // Initialize column heights
      for (int n = 0; n < ColumnCount; n++) {
        columnHeights.Add ((nfloat)0);
      }

      // Process all sections
      nfloat top = 0.0f;
      var attributes = new UICollectionViewLayoutAttributes ();
      var columnIndex = 0;
      for (nint section = 0; section < numberofSections; ++section) {
        // Calculate section specific metrics
        var minimumInterItemSpacing = (MinimumInterItemSpacingForSection == null) ? MinimumColumnSpacing :
          MinimumInterItemSpacingForSection (CollectionView, this, section);

        // Calculate widths
        var width = CollectionView.Bounds.Width - SectionInset.Left - SectionInset.Right;
        var itemWidth = (nfloat)Math.Floor ((width - ((ColumnCount - 1) * MinimumColumnSpacing)) / ColumnCount);

        // Calculate section header
        var heightHeader = (HeightForHeader == null) ? HeaderHeight :
          HeightForHeader (CollectionView, this, section);

        if (heightHeader > 0) {
          attributes = UICollectionViewLayoutAttributes.CreateForSupplementaryView (UICollectionElementKindSection.Header, NSIndexPath.FromRowSection (0, section));
          attributes.Frame = new CGRect (0, top, CollectionView.Bounds.Width, heightHeader);
          headersAttributes.Add (section, attributes);
          allItemAttributes.Add (attributes);

          top = attributes.Frame.GetMaxY ();
        }

        top += SectionInset.Top;
        for (int n = 0; n < ColumnCount; n++) {
          columnHeights [n] = top;
        }

        // Calculate Section Items
        var itemCount = CollectionView.NumberOfItemsInSection(section);
        List<UICollectionViewLayoutAttributes> itemAttributes = new List<UICollectionViewLayoutAttributes> ();

        for (nint n = 0; n < itemCount; n++) {
          var indexPath = NSIndexPath.FromRowSection (n, section);
          columnIndex = NextColumnIndexForItem (n);
          var xOffset = SectionInset.Left + (itemWidth + MinimumColumnSpacing) * (nfloat)columnIndex;
          var yOffset = columnHeights [columnIndex];
          var itemSize = (SizeForItem == null) ? new CGSize (0, 0) : SizeForItem (CollectionView, this, indexPath);
          nfloat itemHeight = 0.0f;

          if (itemSize.Height > 0.0f && itemSize.Width > 0.0f) {
            itemHeight = (nfloat)Math.Floor (itemSize.Height * itemWidth / itemSize.Width);
          }

          attributes = UICollectionViewLayoutAttributes.CreateForCell (indexPath);
          attributes.Frame = new CGRect (xOffset, yOffset, itemWidth, itemHeight);
          itemAttributes.Add (attributes);
          allItemAttributes.Add (attributes);
          columnHeights [columnIndex] = attributes.Frame.GetMaxY () + MinimumInterItemSpacing;
        }
        sectionItemAttributes.Add (itemAttributes);

        // Calculate Section Footer
        nfloat footerHeight = 0.0f;
        columnIndex = LongestColumnIndex();
        top = columnHeights [columnIndex] - MinimumInterItemSpacing + SectionInset.Bottom;
        footerHeight = (HeightForFooter == null) ? FooterHeight : HeightForFooter(CollectionView, this, section);

        if (footerHeight > 0) {
          attributes = UICollectionViewLayoutAttributes.CreateForSupplementaryView (UICollectionElementKindSection.Footer, NSIndexPath.FromRowSection (0, section));
          attributes.Frame = new CGRect (0, top, CollectionView.Bounds.Width, footerHeight);
          footersAttributes.Add (section, attributes);
          allItemAttributes.Add (attributes);
          top = attributes.Frame.GetMaxY ();
        }

        for (int n = 0; n < ColumnCount; n++) {
          columnHeights [n] = top;
        }
      }

      var i =0;
      var attrs = allItemAttributes.Count;
      while(i < attrs) {
        var rect1 = allItemAttributes [i].Frame;
        i = (int)Math.Min (i + unionSize, attrs) - 1;
        var rect2 = allItemAttributes [i].Frame;
        unionRects.Add (CGRect.Union (rect1, rect2));
        i++;
      }

    }

    public override CGSize CollectionViewContentSize {
      get {
        if (CollectionView.NumberOfSections () == 0) {
          return new CGSize (0, 0);
        }

        var contentSize = CollectionView.Bounds.Size;
        contentSize.Height = columnHeights [0];
        return contentSize;
      }
    }

    public override UICollectionViewLayoutAttributes LayoutAttributesForItem (NSIndexPath indexPath)
    {
      if (indexPath.Section >= sectionItemAttributes.Count) {
        return null;
      }

      if (indexPath.Item >= sectionItemAttributes [indexPath.Section].Count) {
        return null;
      }

      var list = sectionItemAttributes [indexPath.Section];
      return list [(int)indexPath.Item];
    }

    public override UICollectionViewLayoutAttributes LayoutAttributesForSupplementaryView (NSString kind, NSIndexPath indexPath)
    {
      var attributes = new UICollectionViewLayoutAttributes ();

      switch (kind) {
      case "header":
        attributes = headersAttributes [indexPath.Section];
        break;
      case "footer":
        attributes = footersAttributes [indexPath.Section];
        break;
      }

      return attributes;
    }

    public override UICollectionViewLayoutAttributes[] LayoutAttributesForElementsInRect (CGRect rect)
    {
      var begin = 0;
      var end = unionRects.Count;
      List<UICollectionViewLayoutAttributes> attrs = new List<UICollectionViewLayoutAttributes> ();

      for (int i = 0; i < end; i++) {
        if (rect.IntersectsWith(unionRects[i])) {
          begin = i * (int)unionSize;
        }
      }

      for (int i = end - 1; i >= 0; i--) {
        if (rect.IntersectsWith (unionRects [i])) {
          end = (int)Math.Min ((i + 1) * (int)unionSize, allItemAttributes.Count);
          break;
        }
      }

      for (int i = begin; i < end; i++) {
        var attr = allItemAttributes [i];
        if (rect.IntersectsWith (attr.Frame)) {
          attrs.Add (attr);
        }
      }

      return attrs.ToArray();
    }

    public override bool ShouldInvalidateLayoutForBoundsChange (CGRect newBounds)
    {
      var oldBounds = CollectionView.Bounds;
      return (newBounds.Width != oldBounds.Width);
    }
    #endregion

    #region Private Methods
    private int ShortestColumnIndex() {
      var index = 0;
      var shortestHeight = nfloat.MaxValue;
      var n = 0;

      // Scan each column for the shortest height
      foreach (nfloat height in columnHeights) {
        if (height < shortestHeight) {
          shortestHeight = height;
          index = n;
        }
        ++n;
      }

      return index;
    }

    private int LongestColumnIndex() {
      var index = 0;
      var longestHeight = nfloat.MinValue;
      var n = 0;

      // Scan each column for the shortest height
      foreach (nfloat height in columnHeights) {
        if (height > longestHeight) {
          longestHeight = height;
          index = n;
        }
        ++n;
      }

      return index;
    }

    private int NextColumnIndexForItem(nint item) {
      var index = 0;

      switch (ItemRenderDirection) {
      case WaterfallCollectionRenderDirection.ShortestFirst:
        index = ShortestColumnIndex ();
        break;
      case WaterfallCollectionRenderDirection.LeftToRight:
        index = ColumnCount;
        break;
      case WaterfallCollectionRenderDirection.RightToLeft:
        index = (ColumnCount - 1) - ((int)item / ColumnCount);
        break;
      }

      return index;
    }
    #endregion

    #region Events
    public delegate CGSize WaterfallCollectionSizeDelegate(UICollectionView collectionView, WaterfallCollectionLayout layout, NSIndexPath indexPath);
    public delegate nfloat WaterfallCollectionFloatDelegate(UICollectionView collectionView, WaterfallCollectionLayout layout, nint section);
    public delegate UIEdgeInsets WaterfallCollectionEdgeInsetsDelegate(UICollectionView collectionView, WaterfallCollectionLayout layout, nint section);

    public event WaterfallCollectionSizeDelegate SizeForItem;
    public event WaterfallCollectionFloatDelegate HeightForHeader;
    public event WaterfallCollectionFloatDelegate HeightForFooter;
    public event WaterfallCollectionEdgeInsetsDelegate InsetForSection;
    public event WaterfallCollectionFloatDelegate MinimumInterItemSpacingForSection;
    #endregion
  }
}
```

このクラスを使用すると、カスタムの2列のウォーターフォール型レイアウトをコレクションビューに提供できます。
このコードでは、(`WillChangeValue` および `DidChangeValue` メソッドを使用して) キー値のコードを使用して、このクラスの計算されたプロパティのデータバインディングを提供します。

次に、`WaterfallCollectionSource` を編集し、次の変更と追加を行います。

```csharp
private Random rnd = new Random();
...

public List<nfloat> Heights { get; set; } = new List<nfloat> ();
...

public WaterfallCollectionSource (WaterfallCollectionView collectionView)
{
  // Initialize
  CollectionView = collectionView;

  // Init numbers collection
  for (int n = 0; n < 100; ++n) {
    Numbers.Add (n);
    Heights.Add (rnd.Next (0, 100) + 40.0f);
  }
}
```

これにより、一覧に表示される各項目の高さがランダムに作成されます。

次に、`WaterfallCollectionView` クラスを編集し、次のヘルパープロパティを追加します。

```csharp
public WaterfallCollectionSource Source {
  get { return (WaterfallCollectionSource)DataSource; }
}
```

これにより、カスタムレイアウトからデータソース (および項目の高さ) を簡単に取得できるようになります。

最後に、ビューコントローラーを編集し、次のコードを追加します。

```csharp
public override void AwakeFromNib ()
{
  base.AwakeFromNib ();

  var waterfallLayout = new WaterfallCollectionLayout ();

  // Wireup events
  waterfallLayout.SizeForItem += (collectionView, layout, indexPath) => {
    var collection = collectionView as WaterfallCollectionView;
    return new CGSize((View.Bounds.Width-40)/3,collection.Source.Heights[(int)indexPath.Item]);
  };

  // Attach the custom layout to the collection
  CollectionView.SetCollectionViewLayout(waterfallLayout, false);
}
```

これにより、カスタムレイアウトのインスタンスが作成され、各項目のサイズを指定して新しいレイアウトをコレクションビューにアタッチするようにイベントが設定されます。

Xamarin. iOS アプリを再度実行すると、コレクションビューは次のようになります。

[![](uicollectionview-images/custom01.png "The collection view will now look like this")](uicollectionview-images/custom01.png#lightbox)

前と同じように項目をドラッグして並べ替えることもできますが、項目がドロップされたときに新しい場所に合わせてサイズが変更されるようになりました。

## <a name="collection-view-changes"></a>コレクションビューの変更

以下のセクションでは、iOS 9 のコレクションビューで各クラスに加えられた変更について詳しく説明します。

### <a name="uicollectionview"></a>UICollectionView

IOS 9 の `UICollectionView` クラスには、次の変更または追加が行われています。

- `BeginInteractiveMovementForItem` –ドラッグ操作の開始をマークします。
- `CancelInteractiveMovement` –ユーザーがドラッグ操作を取り消したことをコレクションビューに通知します。
- `EndInteractiveMovement` –ユーザーがドラッグ操作を完了したことをコレクションビューに通知します。
- `GetIndexPathsForVisibleSupplementaryElements` –コレクションビューセクション内のヘッダーまたはフッターの `indexPath` を返します。
- `GetSupplementaryView` –指定されたヘッダーまたはフッターを返します。
- `GetVisibleSupplementaryViews` –表示されているすべてのヘッダーとフッターの一覧を返します。
- `UpdateInteractiveMovementTargetPosition` –ドラッグ操作中に、ユーザーが項目を移動または移動していることをコレクションビューに通知します。

### <a name="uicollectionviewcontroller"></a>UICollectionViewController

IOS 9 の `UICollectionViewController` クラスには、次の変更または追加が行われています。

- `InstallsStandardGestureForInteractiveMovement` –新しいジェスチャ認識エンジンが、ドラッグアンド並べ替えを自動的にサポートするように `true` 場合に使用されます。
- `CanMoveItem` –特定の項目をドラッグして並べ替えることができるかどうかをコレクションビューに通知します。
- `GetTargetContentOffset` –指定されたコレクションビューアイテムのオフセットを取得するために使用されます。
- `GetTargetIndexPathForMove` –ドラッグ操作に対して指定された項目の `indexPath` を取得します。
- `MoveItem` –リスト内の指定された項目の順序を移動します。

### <a name="uicollectionviewdatasource"></a>UICollectionViewDataSource

IOS 9 の `UICollectionViewDataSource` クラスには、次の変更または追加が行われています。

- `CanMoveItem` –特定の項目をドラッグして並べ替えることができるかどうかをコレクションビューに通知します。
- `MoveItem` –リスト内の指定された項目の順序を移動します。

### <a name="uicollectionviewdelegate"></a>UICollectionViewDelegate

IOS 9 の `UICollectionViewDelegate` クラスには、次の変更または追加が行われています。

- `GetTargetContentOffset` –指定されたコレクションビューアイテムのオフセットを取得するために使用されます。
- `GetTargetIndexPathForMove` –ドラッグ操作に対して指定された項目の `indexPath` を取得します。

### <a name="uicollectionviewflowlayout"></a>UICollectionViewFlowLayout

IOS 9 の `UICollectionViewFlowLayout` クラスには、次の変更または追加が行われています。

- `SectionFootersPinToVisibleBounds` –セクションフッターを、表示可能なコレクションビューの境界にします。
- `SectionHeadersPinToVisibleBounds` –セクションヘッダーを表示されているコレクションビューの境界にします。

### <a name="uicollectionviewlayout"></a>UICollectionViewLayout

IOS 9 の `UICollectionViewLayout` クラスには、次の変更または追加が行われています。

- `GetInvalidationContextForEndingInteractiveMovementOfItems` –ドラッグ操作の終了時に、ユーザーがドラッグまたはキャンセルを完了したときに、無効化コンテキストを返します。
- `GetInvalidationContextForInteractivelyMovingItems` –ドラッグ操作の開始時に無効化コンテキストを返します。
- `GetLayoutAttributesForInteractivelyMovingItem` –項目のドラッグ中に、指定された項目のレイアウト属性を取得します。
- `GetTargetIndexPathForInteractivelyMovingItem` –項目をドラッグするときに、指定したポイントにある項目の `indexPath` を返します。

### <a name="uicollectionviewlayoutattributes"></a>UICollectionViewLayoutAttributes

IOS 9 の `UICollectionViewLayoutAttributes` クラスには、次の変更または追加が行われています。

- `CollisionBoundingPath` –ドラッグ操作中に2つの項目の競合パスを返します。
- `CollisionBoundsType` –ドラッグ操作中に発生した競合の種類 (`UIDynamicItemCollisionBoundsType`として返されます) を返します。

### <a name="uicollectionviewlayoutinvalidationcontext"></a>UICollectionViewLayoutInvalidationContext

IOS 9 の `UICollectionViewLayoutInvalidationContext` クラスには、次の変更または追加が行われています。

- `InteractiveMovementTarget` –ドラッグ操作のターゲット項目を返します。
- `PreviousIndexPathsForInteractivelyMovingItems` –ドラッグ操作に関連する他の項目の `indexPaths` を返します。
- `TargetIndexPathsForInteractivelyMovingItems` –ドラッグアンド並べ替え操作の結果として並べ替えられる項目の `indexPaths` を返します。

### <a name="uicollectionviewsource"></a>UICollectionViewSource

IOS 9 の `UICollectionViewSource` クラスには、次の変更または追加が行われています。

- `CanMoveItem` –特定の項目をドラッグして並べ替えることができるかどうかをコレクションビューに通知します。
- `GetTargetContentOffset` –ドラッグアンドドロップ操作によって移動される項目のオフセットを返します。
- `GetTargetIndexPathForMove` –ドラッグアンド並べ替え操作中に移動される項目の `indexPath` を返します。
- `MoveItem` –リスト内の指定された項目の順序を移動します。

## <a name="summary"></a>まとめ

この記事では、iOS 9 でのコレクションビューの変更点と、それらを Xamarin で実装する方法について説明しました。
コレクションビューでのドラッグから順序変更の単純なアクションの実装について説明します。ドラッグして順序を変更し、カスタムジェスチャ認識エンジンを使用するまた、ドラッグアンドドロップ操作は、カスタムコレクションビューのレイアウトに影響します。

## <a name="related-links"></a>関連リンク

- [iOS 9 のサンプル](https://docs.microsoft.com/samples/browse/?products=xamarin&term=Xamarin.iOS+iOS9)
- [コレクションビューのサンプル](https://docs.microsoft.com/samples/xamarin/ios-samples/ios9-collectionview)
- [SimpleCollectionView (サンプル)](https://docs.microsoft.com/samples/xamarin/ios-samples/simplecollectionview)
- [イベント、プロトコル、デリゲート](~/ios/app-fundamentals/delegates-protocols-and-events.md)
- [テーブルとセルの操作](~/ios/user-interface/controls/tables/index.md)
