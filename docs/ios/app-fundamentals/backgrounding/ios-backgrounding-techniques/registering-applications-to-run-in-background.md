---
title: バックグラウンドで実行する Xamarin iOS アプリを登録しています
description: このドキュメントでは、バックグラウンドで実行する Xamarin iOS アプリケーションを登録する方法について説明します。 オーディオアプリ、VoIP アプリ、外部のアクセサリ、bluetooth などについて説明します。
ms.prod: xamarin
ms.assetid: 8F89BE63-DDB5-4740-A69D-F60AEB21150D
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 03/18/2017
ms.openlocfilehash: 61b7926f28253acbcc45bc204c466d76a00c72b0
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73010830"
---
# <a name="registering-xamarinios-apps-to-run-in-the-background"></a>バックグラウンドで実行する Xamarin iOS アプリを登録しています

バックグラウンド特権に対する個々のタスクの登録は一部のアプリケーションでは機能しますが、アプリケーションが常に実行されていて、ユーザーが GPS 経由で指示を受け取るなど、長時間実行される重要なタスクを実行する場合はどうなるでしょうか。 そのようなアプリケーションは、代わりに、既知のバックグラウンドで必要なアプリケーションとして登録する必要があります。

アプリの通知を iOS に登録すると、バックグラウンドでタスクを実行するために必要な特別な特権をアプリケーションに付与する必要があります。

## <a name="application-registration-categories"></a>アプリケーションの登録カテゴリ

登録済みアプリはいくつかのカテゴリに分類されます。

- **オーディオと**オーディオコンテンツを操作するその他のアプリケーションは、アプリがフォアグラウンドになくなった場合でもオーディオの再生を継続するように登録できます。 このカテゴリのアプリが、バックグラウンドでオーディオの再生やダウンロード以外の操作を実行しようとすると、iOS はそれを終了します。
- **Voip** -ボイスオーバーインターネットプロトコル (voip) アプリケーションは、オーディオアプリケーションに与えられているのと同じ特権を取得して、オーディオをバックグラウンドで処理し続けます。 また、接続を維持するために、必要に応じて電力を供給する VoIP サービスに応答することもできます。
- **外部のアクセサリと bluetooth** : bluetooth デバイスやその他の外部ハードウェアのアクセサリと通信する必要があるアプリケーション用に予約されています。これらのカテゴリ下で登録すると、アプリはハードウェアに接続したままにすることができます。
- **Newsstand** -Newsstand アプリケーションは、バックグラウンドでコンテンツを引き続き同期できます。
- **場所**-GPS またはネットワークの場所データを使用するアプリケーションは、バックグラウンドで位置情報の更新を送受信できます。
- **Fetch (iOS 7 以降)** -バックグラウンドフェッチ権限用に登録されたアプリケーションでは、プロバイダーが定期的に新しいコンテンツを確認し、アプリケーションに戻ったときに更新されたコンテンツをユーザーに提示できます。
- **リモート通知 (iOS 7 以降)** -アプリケーションは、プロバイダーからの通知を受信するように登録できます。また、通知を使用して、ユーザーがアプリケーションを開く前に更新プログラムを開始します。 通知はプッシュ通知の形式で提供されます。または、アプリケーションをサイレントモードでスリープ解除することを選択できます。

アプリケーションを登録するには、アプリケーションの*情報*で **[必要なバックグラウンドモード]** プロパティを設定します。 アプリケーションは、必要な数だけカテゴリに登録できます。

 [![](registering-applications-to-run-in-background-images/bgmodes.png "Setting the background modes")](registering-applications-to-run-in-background-images/bgmodes.png#lightbox)

バックグラウンドの場所の更新用にアプリケーションを登録する手順については、「[バックグラウンドの場所」チュートリアル](~/ios/app-fundamentals/backgrounding/ios-backgrounding-walkthroughs/location-walkthrough.md)を参照してください。

## <a name="application-does-not-run-in-background-property"></a>アプリケーションはバックグラウンドプロパティでは実行されません

*情報 plist*で設定できるもう1つのプロパティは、*アプリケーションがバックグラウンドで実行されない*か、`UIApplicationExitsOnSuspend` プロパティです。

 [![](registering-applications-to-run-in-background-images/plist.png "Disabling Background Running")](registering-applications-to-run-in-background-images/plist.png#lightbox)

これは、iOS 7 以降では、バックグラウンドアプリの更新設定をオフに設定した場合とまったく同じ効果があります。ただし、この設定は、開発者側でのみ変更でき、iOS 4 以上で使用できます。 バックグラウンドの入力直後にアプリケーションは中断され、処理を実行することはできません。

このプロパティは、アプリケーションがバックグラウンド処理を処理するように設計されていない場合に使用します。これは、予期しない動作を回避するのに役立ちます。

## <a name="background-fetch-and-remote-notifications"></a>バックグラウンドフェッチとリモート通知

バックグラウンドフェッチとリモート通知は、iOS 7 で導入された特別な登録カテゴリです。 これらのカテゴリを使用すると、アプリケーションはプロバイダーから新しいコンテンツを受信し、バックグラウンドで更新できます。 次のセクションでは、フェッチとリモート通知について詳しく説明します。また、iOS 6 のバックグラウンドでアプリケーションを更新するための手段として位置情報の認識も導入されています。
