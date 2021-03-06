---
title: 'IBTool エラー: 操作を完了できませんでした。'
ms.topic: troubleshooting
ms.prod: xamarin
ms.assetid: A804EBC4-2BBF-4A98-A4E8-A455DB2E8A17
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 04/03/2018
ms.openlocfilehash: 4ff7b88ad63870246acbf2b29d01775f6711a8ce
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73031183"
---
# <a name="ibtool-error-the-operation-couldnt-be-completed"></a>IBTool エラー: 操作を完了できませんでした。

## <a name="fixed-in-xcode-611"></a>Xcode 6.1.1 で修正済み

Apple は Xcode 6.1.1 でこの `ibtool` バグを[修正](https://developer.apple.com/library/content/documentation/Xcode/Conceptual/RN-Xcode-Archive/Chapters/xc6_release_notes.html#//apple_ref/doc/uid/TP40016994-CH4-SW1)したので、Xcode 6.1.1 以降にアップグレードするのが最も簡単な修正です。

* * *

## <a name="description-of-the-problem"></a>問題の説明

Xcode 6.0 の `ibtool` コマンドに、OS X 10.10 ヨークのバグがありました。 Xamarin では、Xcode の `ibtool` を使用して、ストーリーボードと `XIB` ファイルをコンパイルします。

Xcode に関連するバグの詳細については、次の Stack Overflow 投稿を参照してください: [https://stackoverflow.com/questions/25754763/cant-open-storyboard](https://stackoverflow.com/questions/25754763/cant-open-storyboard)

### <a name="error-message"></a>エラー メッセージ

> ドキュメント "Mainstoryboard.storyboard ファイル" を開けませんでした。 操作を完了できませんでした。 (com. InterfaceBuilder エラー-1)。

## <a name="workarounds-for-xcode-60"></a>回避策 (Xcode 6.0 の場合)

### <a name="option-1-manage-all-uiimageviewimage-properties-in-code"></a>オプション 1: すべての `UIImageView.Image` プロパティをコードで管理する

ストーリーボードまたは `.xib` ファイルの `UIImageView` の `Image` プロパティを設定するのではなく、ビューコントローラーのビューライフサイクルオーバーライドメソッドのいずれかでプロパティを設定できます (`ViewDidLoad()`など)。 `UIImage.FromBundle()` と `UIImage.FromFile()`の使用に関するヒントについては、「[イメージの操作](~/ios/app-fundamentals/images-icons/index.md)」も参照してください。

### <a name="option-2-move-all-of-the-image-resources-to-the-top-level-resources-folder"></a>オプション 2: すべてのイメージリソースを最上位レベルの `Resources` フォルダーに移動する

イメージを最上位の `Resources` フォルダーに移動したら、新しいイメージパスを使用するように、ストーリーボードと `.xib` ファイルを更新する必要があります。

### <a name="option-3-set-the-logicalname-for-any-problematic-image-assets-so-they-are-copied-to-the-top-level-of-theapp-bundle"></a>オプション 3: 問題のあるイメージ資産の `LogicalName` を設定して、`.app` バンドルの最上位レベルにコピーする

たとえば、元の `.csproj` ファイルに次のエントリが含まれているとします。

`<BundleResource Include="Resources\Images\image.png" />`

この要素を変更し、`LogicalName` を追加して、イメージが `.app` バンドルの最上位レベルにコピーされるようにすることができます。

```xml
<BundleResource Include="Resources\Images\image.png">
    <LogicalName>image.png</LogicalName>
</BundleResource>
```

Visual Studio for Mac では、[> 表示] の下にある画像の `Resource ID` フィールドを使用して `LogicalName` を設定することもできます **> プロパティ**です。 (「 [https://stackoverflow.com/questions/16938250/xamarin-studio-folder-structure-issue-in-ios-project/16951545#16951545](https://stackoverflow.com/questions/16938250/xamarin-studio-folder-structure-issue-in-ios-project/16951545#16951545))」も参照してください。

この変更を行った後、新しい最上位レベルのイメージパスを使用するには、ストーリーボードと `.xib` ファイルを更新する必要があります。 Visual Studio for Mac によって、iOS Designer の `Image` プロパティの autocompletions の一覧が自動的に更新されます。 Visual Studio では、パスを手動で編集する必要があります。 IOS デザイナーでは、これは見つからないイメージとして表示されますが、プロジェクトは正常にビルドされ、実行されます。

### <a name="next-steps"></a>次のステップ

詳細については、お問い合わせください。または、上記の情報を利用した後もこの問題が発生する場合は、「 [Xamarin で使用できるサポートオプション](~/cross-platform/troubleshooting/support-options.md)」を参照してください。連絡先オプション、提案、および必要に応じて新しいバグをファイルに登録する方法については、こちらを参照してください. 
