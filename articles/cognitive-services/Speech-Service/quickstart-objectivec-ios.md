---
title: 'クイック スタート: iOS で Objective-C と Speech Service SDK を使用して音声を認識する'
titleSuffix: Azure Cognitive Services
description: iOS で Objective-C と Speech Service SDK を使用して音声を認識する方法について説明します
services: cognitive-services
author: chlandsi
manager: cgronlun
ms.service: cognitive-services
ms.component: speech-service
ms.topic: quickstart
ms.date: 11/06/2018
ms.author: chlandsi
ms.openlocfilehash: 7d1e05e13e55b8b7bc07eda71d63f96f12c81ff9
ms.sourcegitcommit: 1b186301dacfe6ad4aa028cfcd2975f35566d756
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/06/2018
ms.locfileid: "51219190"
---
# <a name="quickstart-recognize-speech-in-objective-c-on-ios-using-the-speech-service-sdk"></a>クイック スタート: iOS で Objective-C と Speech Service SDK を使用して音声を認識する

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

この記事では、音声が録音された音声ファイルをテキストに変換するために、Cognitive Services Speech SDK を使用して Objective-C で iOS アプリを作成する方法について説明します。

## <a name="prerequisites"></a>前提条件

* Speech サービスのサブスクリプション キー。 「[Speech サービスを無料で試す](get-started.md)」を参照してください。
* iOS 開発環境として Xcode 9.4.1 がインストールされている Mac。 このチュートリアルは iOS バージョン 11.4 を対象としています。 まだ Xcode をお持ちでない場合は、[App Store](https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12) からインストールできます。

## <a name="get-the-speech-sdk-for-ios"></a>iOS 用の Speech SDK を取得する

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

Cognitive Services Speech SDK の現在のバージョンは `1.1.0` です。

Mac および iOS 用の Cognitive Services Speech SDK は、現在は Cocoa フレームワークとして配布されています。
これは https://aka.ms/csspeech/iosbinary からダウンロードできます。 ファイルをホーム ディレクトリにダウンロードします。

## <a name="create-an-xcode-project"></a>Xcode プロジェクトの作成 

Xcode を起動し、**[File]** > **[New]** > **[Project]** の順にクリックして新しいプロジェクトを開始します。
テンプレートの選択ダイアログで、"iOS Single View App" テンプレートを選択します。

以降のダイアログで、次のように選択します。

1. [Project Options] ダイアログ
    1. クイック スタート アプリの名前を入力します (例: `helloworld`)。
    1. 既に Apple Developer アカウントをお持ちの場合は、適切な組織名と組織 ID を入力します。 テスト目的の場合は、単に `testorg` のような名前を選択できます。 アプリに署名するためには、適切なプロビジョニング プロファイルも必要です。 詳細については、[Apple の開発者向けサイト](https://developer.apple.com/)を参照してください。
    1. プロジェクトの言語として Objective-C が選択されていることを確認します。
    1. テストとコア データに関するすべてのチェック ボックスを無効にします。
    ![プロジェクトの設定](media/sdk/qs-objectivec-project-settings.png)
1. プロジェクト ディレクトリの選択
    1. ホーム ディレクトリを選択して、プロジェクトを配置します。 これにより、ホーム ディレクトリ内に `helloworld` ディレクトリが作成され、Xcode プロジェクトのすべてのファイルが入れられます。
    1. このサンプル プロジェクトの Git リポジトリの作成を無効にします。
    1. *[Project Settings]* で、SDK へのパスを調整します。
        1. **[General]** タブの **[Embedded Binaries]** ヘッダーの下で、フレームワークとして SDK ライブラリを追加し (**[Add embedded binaries]** > **[Add other...]** >)、ホーム ディレクトリに移動して、ファイル `MicrosoftCognitiveServicesSpeech.framework` を選択します。 これにより、SDK のライブラリは、ヘッダー **[Linked Framework and Libraries]** にも自動的に追加されます。
        ![追加されたフレームワーク](media/sdk/qs-objectivec-framework.png)
        1. **[Build Settings]** タブに移動し、**[All]** の設定をアクティブにします。
        1. ディレクトリ `$(SRCROOT)/..` を、**[Search Paths]** 見出しの下にある *[Framework Search Paths]* に追加します。
        ![[Framework Search Paths] の設定](media/sdk/qs-objectivec-framework-search-paths.png)

## <a name="set-up-the-ui"></a>UI の設定

このサンプル アプリの UI はごく単純なものです。音声認識を開始するためのボタンが 2 つ (ファイルから開始するためのボタンとマイク入力から開始するためのボタン) と、結果を表示するためのテキスト ラベルが 1 つあるだけです。
UI はプロジェクトの `Main.storyboard` 部分に設定されます。
プロジェクト ツリーの `Main.storyboard` エントリーを右クリックし、**[Open As...]** > **[Source Code]** の順に選択して、ストーリーボードの XML ビューを開きます。
自動生成された XML をこれに置き換えます。

[!code-xml[](~/samples-cognitive-services-speech-sdk/quickstart/objectivec-ios/helloworld/helloworld/Base.lproj/Main.storyboard)]

## <a name="add-the-sample-code"></a>サンプル コードを追加する

1. リンクを右クリックし、**[Save target as...]** を選択して、[サンプルの wav ファイル](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-speech-sdk/f9807b1079f3a85f07cbb6d762c6b5449d536027/samples/cpp/windows/console/samples/whatstheweatherlike.wav)をダウンロードします。Finder ウィンドウからプロジェクト ビューのルート レベルにドラッグすることで、wav ファイルをプロジェクトにリソースとして追加します。
設定を変更せずに、次のダイアログで **[Finish]** をクリックします。
1. 次の操作によって、自動生成された `ViewController.m` ファイルの内容を置き換えます。

   [!code-objectivec[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/objectivec-ios/helloworld/helloworld/ViewController.m#code)]
1. 文字列 `YourSubscriptionKey` をサブスクリプション キーに置き換えます。
1. 文字列 `YourServiceRegion` を、サブスクリプションに関連付けられた[リージョン](regions.md) (たとえば、無料試用版サブスクリプションでは `westus`) に置き換えます。
1. マイクへのアクセスの要求を追加します。 プロジェクト ツリーの `Info.plist` のエントリを右クリックし、**[Open As...]** > **[Source Code]** の順に選択します。 `<dict>` セクションに以下の行を追加してから、ファイルを保存します。
    ```xml
    <key>NSMicrophoneUsageDescription</key>
    <string>Need microphone access for speech recognition from microphone.</string>
    ```

## <a name="building-and-running-the-sample"></a>サンプルのビルドと実行

1. デバッグ出力が表示されるようにします (**[View]** > **[Debug Area]** > **[Activate Console]**)。
1. **[Product]\(製品\)** -> **[Destination]\(ターゲット\)** メニューの一覧から、お使いの開発マシンに接続された iOS デバイスまたは iOS シミュレーターをアプリのターゲットとして選択します。
1. コード例をビルドし、メニューから **[Product]** -> **[Run]** の順に選択するか、**[Play]** ボタンをクリックして、iOS シミュレーターで実行します。
現在、Speech SDK は、64 ビット iOS プラットフォームのみをサポートしています。
1. アプリの [Recognize (File)]\(認識 (ファイル)\) ボタンをクリックすると、画面の下部にオーディオ ファイルの内容である "What's the weather like?" が表示されます。 表示されます。

 ![シミュレートされた iOS アプリ](media/sdk/qs-objectivec-simulated-app.png)

1. アプリの [Recognize (Microphone)]\(認識 (マイク)\) ボタンをクリックして何か話すと、発話した内容のテキストが画面の下部に表示されます。

[!INCLUDE [Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
このサンプルは、`quickstart/objectivec-ios` フォルダーで探してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [サンプルを入手する](speech-sdk.md#get-the-samples)

