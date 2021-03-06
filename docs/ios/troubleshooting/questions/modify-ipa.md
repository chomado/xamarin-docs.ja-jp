---
title: Visual Studio でビルドした後、IPA ファイルにファイルを追加したり、ファイルを削除したりすることはできますか。
ms.topic: troubleshooting
ms.prod: xamarin
ms.assetid: 6C3082FB-C3F1-4661-BE45-64570E56DE7C
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 04/03/2018
ms.openlocfilehash: f63cc544b251ca284a8d087e09f9cbc4dfb3f769
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73030953"
---
# <a name="can-i-add-files-to-or-remove-files-from-an-ipa-file-after-building-it-in-visual-studio"></a>Visual Studio でビルドした後、IPA ファイルにファイルを追加したり、ファイルを削除したりすることはできますか。

はい、できますが、通常は、変更を行った後に `.app` バンドルに再署名する必要があります。

通常の使用では、`.ipa` ファイルを変更する必要はありません。 この記事は、あくまで情報提供を目的として提供されています。

## <a name="example-removing-a-file-from-a-ipa-archive"></a>例: `.ipa` アーカイブからのファイルの削除

この例では、Xamarin. iOS プロジェクトの名前が `iPhoneApp1`、`generated session id` が `cc530d20d6b19da63f6f1c6f67a0a254`

1. `.ipa` ファイルを Visual Studio から通常どおりにビルドします。

2. Mac ビルドホストに切り替えます。

3. `~/Library/Caches/Xamarin/mtbs/builds` フォルダーでビルドを見つけます。 このパスを Finder に貼り付けることができます **> [フォルダーに移動**] を選択して finder でフォルダーを参照 > ます。 プロジェクト名と一致するフォルダーを探します。 そのフォルダー内で、ビルドの `generated session id` に一致するフォルダーを探します。 これは、ほとんどの場合、最新の変更時刻を持つサブフォルダーです。

4. 新しい `Terminal.app` ウィンドウを開きます。

5. [`cd`] ウィンドウに「」と入力し、`generated session id` フォルダーを `Terminal.app` ウィンドウにドラッグ & ます。

    ![](modify-ipa-images/session-id-folder.png "Locating the generated session id folder in Finder")

6. [戻る] キーを入力して、ディレクトリを `generated session id` フォルダーに変更します。

7. 次のコマンドを使用して、`.ipa` ファイルを一時 `old/` フォルダーに解凍します。 特定のプロジェクトで必要に応じて、`Ad-Hoc` と `iPhoneApp1` の名前を調整します。

    > ditto-xk bin/iPhone/Ad-Hoc/iPhoneApp1-1.0. ipa old/

8. `Terminal.app` ウィンドウを開いたままにします。

9. `.ipa`から目的のファイルを削除します。 Finder を使用してごみ箱に移動するか、`Terminal.app`を使用してコマンドラインで削除することができます。 Finder で `Payload/iPhone` ファイルの内容を表示するには、ファイルを管理し、 **[パッケージの内容を表示]** を選択します。

10. 手順3と同じ一般的な方法を使用して、名前にプロジェクト名と `generated session id` の両方を含む `~/Library/Logs/Xamarin/MonoTouchVS/` のログファイルを探します。![](modify-ipa-images/build-log.png "Finder でプロジェクトビルドログを検索する")

11. 手順10のビルドログを開きます。たとえば、ダブルクリックします。

12. `tool /usr/bin/codesign execution started with arguments: -v --force --sign`が含まれている行を探します。

13. 手順8の [アプリ] ウィンドウに `/usr/bin/codesign` を入力します。

14. 手順12の行の `-v` で始まるすべての引数をコピーし、それらをターミナルアプリウィンドウに貼り付けます。

15. 最後の引数を `old/Payload/` フォルダー内にある `.app` バンドルに変更し、コマンドを実行します。

    ```bash
    /usr/bin/codesign -v --force --sign SOME_LONG_STRING in/iPhone/Ad-Hoc/iPhoneApp1.app/ResourceRules.plist --entitlements obj/iPhone/Ad-Hoc/Entitlements.xcent old/Payload/iPhoneApp1.app
    ```

16. ターミナルの `old/` ディレクトリに変更します。

    ```bash
    cd old
    ```

17. `zip` コマンドを使用して、ディレクトリの内容を新しい `.ipa` ファイルに圧縮します。 `"$HOME/Desktop/iPhoneApp1-1.0.ipa"` 引数を変更して、任意の場所で `.ipa` ファイルを出力することができます。

    ```bash
    zip -yr "$HOME/Desktop/iPhoneApp1-1.0.ipa" *
    ```

## <a name="common-error-messages"></a>一般的なエラーメッセージ

`Invalid Signature. A sealed resource is missing or invalid.`が表示される場合は、通常、`.app` バンドル内で何らかの変更が行われており、その後、`.app` バンドルが正しく再署名されていないことを意味します。 配布プロファイルを使用して `.ipa` を作成する場合は、配布プロファイルを使用して元の `.ipa` を構築する_必要が_あることにも注意してください。 それ以外の場合、`Entitlements.xcent` は正しくありません。

このエラーが発生する具体的な例を示すために、手順9の後にターミナルウィンドウで次の `codesign --verify` コマンドを実行すると、エラーの正確な原因と共にエラーが表示されます。

```bash
$ codesign -dvvv --no-strict --verify old/Payload/iPhoneApp1.app
old/Payload/iPhoneApp1.app: a sealed resource is missing or invalid
file missing: /Users/macuser/Library/Caches/Xamarin/mtbs/builds/iPhoneApp1/cc530d20d6b19da63f6f1c6f67a0a254/old/Payload/iPhoneApp1.app/MyFile.png
```

また、アプリストアの検証プロセスでも、次のようなエラーメッセージが報告されます。

> エラー ITMS-90035: "署名が無効です。 封印されたリソースがないか、無効です。 パス [iPhoneApp1/iPhoneApp1] のバイナリに無効な署名が含まれています。 アドホック証明書または開発証明書ではなく、配布証明書を使用してアプリケーションに署名していることを確認します。 Xcode のコード署名設定がターゲットレベルで正しいことを確認します (プロジェクトレベルのすべての値をオーバーライドします)。 また、アップロードするバンドルが、シミュレーターターゲットではなく、Xcode のリリースターゲットを使用してビルドされていることを確認します。 コード署名の設定が正しいことがわかっている場合は、Xcode で [すべてクリーン] を選択し、Finder の "build" ディレクトリを削除して、リリースターゲットをリビルドします。 詳細については、「 [https://developer.apple.com/library/ios/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html](https://developer.apple.com/library/ios/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)」を参照してください。
