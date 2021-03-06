---
title: Xamarin. iOS のラベル
description: このドキュメントでは、Xamarin. iOS でラベルを使用する方法について説明します。 プログラムによって、また iOS Designer を使用してラベルを作成する方法について説明します。
ms.prod: xamarin
ms.assetid: 54DA1221-13E4-4D45-B263-5F22A0AC7B53
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 07/11/2017
ms.openlocfilehash: 04d33d986d76daf29fc7392206c62f77d34dd969
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73022020"
---
# <a name="labels-in-xamarinios"></a>Xamarin. iOS のラベル

`UILabel` コントロールは、単一行および複数行の読み取り専用のテキストを表示するために使用されます。

## <a name="implementing-a-label"></a>ラベルの実装

新しいラベルは、 [`UILabel`](xref:UIKit.UILabel)をインスタンス化することによって作成されます。

```csharp
UILabel label = new UILabel();
```

### <a name="labels-and-storyboards"></a>ラベルとストーリーボード

IOS Designer を使用しているときに、UI にラベルを追加することもできます。 **[ツールボックス]** で **[ラベル]** を検索し、ビューにドラッグします。

![ツールボックスのラベル](labels-images/image3.png)

プロパティパッドでは、次のプロパティを調整できます。

![ラベルプロパティパネル](labels-images/image2.png)

- **テキストコンテキスト**– Plain または属性付き。 テキスト形式では、文字列全体の[書式属性](#Formatting_Text_and_Label)を設定できます。 属性付きテキストを使用すると、書式設定を文字列内の異なる文字または単語に設定できます。
- **色、フォント、配置**–ラベルに適用できる書式設定属性。
- **Lines** –ラベルがまたがる行の数を設定します。 ラベルが必要な数の行を使用できるようにするには、これを0に設定します。
- **動作**-[有効] または [強調表示] のいずれかに設定できます。 [有効] が既定で設定されており、無効になっているテキストは薄い灰色の色で表示されます。 強調表示は既定で無効になっており、ユーザーが選択したときに、強調表示された状態でラベルを再描画できます。
- **Baselane と改行**–
  - Basline フォントサイズが指定されたサイズと異なる場合に、テキストの配置方法を決定します。
  - 改行は、1行より長い場合に文字列の折り返しまたは切り捨てを行う方法を決定します。
- 自動**圧縮**–必要に応じて、ラベル内でフォントサイズを最小化する方法を決定します。
- **強調表示、影、オフセット**– Hightlighted と影の色、および影のオフセットを設定できます。

## <a name="truncating-and-wrapping"></a>切り捨てと折り返し

IOS で改行を使用する方法の詳細については、「[テキストの切り捨てとラップ](https://github.com/xamarin/recipes/tree/master/Recipes/ios/standard_controls/labels/uilabel-truncate-wrap-text)」レシピを参照してください。

<a name="Formatting_Text_and_Label"/>

## <a name="formatting-text-and-label"></a>テキストとラベルの書式設定

ラベルで使用する文字列を書式設定するには、文字列全体の書式属性を設定するか、属性付きの文字列を使用できます。 これらの実装方法を次の例に示します。

```csharp
label = new UILabel(){
                Text = "Hello, this is a string",
                Font = UIFont.FromName("Papyrus", 20f),
                TextColor = UIColor.Magenta,
                TextAlignment = UITextAlignment.Center
            };
```

```csharp
label.AttributedText = new NSAttributedString(
                "This is some formatted text",
                font: UIFont.FromName("GillSans", 16.0f),
                foregroundColor: UIColor.Blue,
                backgroundColor: UIColor.White
            );
```

`NSAttributedString` を使用したテキストのスタイル設定の詳細については、「[スタイルテキスト](https://github.com/xamarin/recipes/tree/master/Recipes/ios/standard_controls/text_field/style_text)」レシピを参照してください。

既定では、ラベルには `Enabled` が true に設定されていますが、特定のコントロールが無効になっていることをユーザーにヒントとして設定することもできます。

```csharp
label.Enabled = false;
```

これにより、iOS の制限画面の次の図の例に示すように、ラベルが明るい灰色の色に設定されます。

![IOS の [無効] ボタン](labels-images/image1.png)

また、強調表示と影のテキストの色をラベルテキストに設定して、その他の効果を確認することもできます。

```csharp
label.Highlighted = true;
label.HighlightedTextColor = UIColor.Cyan;

label.ShadowColor = UIColor.Black;
label.ShadowOffset = new CoreGraphics.CGSize(1.0f, 1.0f);
```

次のようなテキストが表示されます。

![テキストの強調表示と影付き設定](labels-images/image4.png)

UILabel のフォントを変更する方法の詳細については、「[フォントの変更](https://github.com/xamarin/recipes/tree/master/Recipes/ios/standard_controls/labels/change_the_font)」レシピを参照してください。
