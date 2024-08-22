# DocumentTranslation

Power Apps で作成したドキュメント全体を別の言語に翻訳するアプリです。

* ドキュメントをPower Apps にアップロードして、翻訳先の言語を指定します。
* 実行すると翻訳され、結果のファイルのリンクが提供されます。
* また、チャットにも翻訳の実行完了とともにファイルのリンクが通知されます。
* ファイルはOneDrive のルードフォルダに保存されるので、必要に応じて別の場所に移動します。

Azure ドキュメント翻訳: [ドキュメント変換とは - Azure AI services | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/ai-services/translator/document-translation/overview)


> [!NOTE]
> Teams への通知も何故なされるかというと、Power Automate をPower Apps から呼び出した際の最大の待機時間の2分間であるための対処で、その時間を超える処理が経過した際にも完了通知が来るようにするため保険的に行われるようになっています。
> また、これにより大きいファイルの翻訳にも対応できる仕様としています。

https://github.com/user-attachments/assets/2169afb1-f34e-4717-8306-da3b15413526

# アーキテクチャ

ドキュメント翻訳(Azure Translation) にて翻訳処理を行いつつ、非同期処理のためPower Automate で処理を監視しています。処理が完了したらOneDrive に翻訳後のファイルを保存して、変換が終了したら、一時的に利用したストレージ(BLOB) は削除します。

![image](https://github.com/user-attachments/assets/c100d248-6595-4655-ae0c-6a6d6137b181)

Power Automate のフローはこのようにしています。

![image](https://github.com/user-attachments/assets/fae7ae7a-065b-45d7-bf86-359a27bbfd4f)

Do untilのところはバッチジョブのステータスを確認して成功したらループを抜けるようになっています。

![image](https://github.com/user-attachments/assets/50cbb806-624a-4974-8a3e-6c333ca3e314)


# デザイン

レスポンシブデザインに対応しています。

自由な画面サイズで利用いただけます。

![image](https://github.com/user-attachments/assets/0615da76-8042-4b5b-a930-7b0ee43d0887)




# 前提条件
## 対応しているファイル形式

対応している形式は以下のとおりです。こちらはAzure Translation サービスに依存しています。

pdf, csv, html, htm, xlf, markdown, mdown, mkdn, md, mkd, mdwn, mdtxt, mdtext, rmd, mthml, mht, xls, xlsx, msg, ppt,  pptx, doc, docx, odt, odp, ods, rtf, tsv/tab, txt

[Azure Translation Serivice | Mcirosoft Learn
](https://learn.microsoft.com/ja-jp/azure/ai-services/translator/document-translation/overview#batch-supported-document-formats)

## 対応しているファイルサイズ

* 最大40MBです。
  - もしこれを超える場合、ファイルを分割するか、ファイル内のコンテンツを一度削除したりしてください。
  - 例えば動画が入っている場合、動画は翻訳されませんので、一旦削除して、変換が終わったらまた追加するようにしてください。

## リソース
以下が必要です。

* Power Apps Premium ライセンス
* Azure サブスクリプション
* Azure ストレージアカウントの管理権利
* Azure AI Translator の管理権利


## 事前準備

### Azure ストレージアカウントの準備

Azure ポータルにて、ストレージアカウントを作成します。

マーケットプレイスから`Storage account`を探して作成します。Standardのレベルで作成するようにします。

![image](https://github.com/user-attachments/assets/d577f990-b26f-4ea4-9b7e-de674627188f)

2個のコンテナを作成します。

コンテナ名は`input`と`output`とします。

![image](https://github.com/user-attachments/assets/8f065eda-2d32-4564-a2d0-7dfecd03d75c)

作成できましたら、Access keyとStorage Account name を取得しておきます。

![image](https://github.com/user-attachments/assets/3894c6f2-9527-4f82-9dce-f299e48e1d9d)

> [!NOTE]
> ソリューションをインポートする際の接続情報を作成する際に必要となります。

### AI Translator の準備

Translator と検索します。表示されたサービスを選択して作成します。

![image](https://github.com/user-attachments/assets/b3047051-3a1e-44b6-b30b-3e09c90482b7)

リソースの名前、リージョンはご自身のお住いに近いところ、Pricing tier は利用量に応じて選択ください。

![image](https://github.com/user-attachments/assets/38f6d25b-0b67-47b7-8ee2-18df3827dd94)

> [!NOTE]
> Azure AI 翻訳の価格は[こちら](https://azure.microsoft.com/pricing/details/cognitive-services/translator/)に記載があります。

デプロイされたTranslator のリソースにて、Identity を選択します。Status を有効にして一度保存します。保存後に現れるPermissions からAzure role assignments を行います。

![image](https://github.com/user-attachments/assets/601818a3-1190-492a-96d9-fe91bdf9e5b9)

Add role assignment からロールを追加します。

![image](https://github.com/user-attachments/assets/584697c2-17e0-404e-8f16-779055c53ebe)

Roleの設定は以下のとおりに行います。Resource のところは先ほど作成したストレージアカウント名を指定するようにしてください。

![image](https://github.com/user-attachments/assets/1c8fac7f-de1b-41ce-85ca-161d9db6eb12)

> [!NOTE]
> 1-2 分程立つとストレージアカウントが追加されます。少し待ってから更新を行って反映されるのを確認してください。

Identity の設定がこれで完了しました。

次に、Keys and Endpoint に移ります。こちらで、Document Translation のエンドポイント名とKey を取得しておきます。メモ帳などに残しておいてください。

![image](https://github.com/user-attachments/assets/9aad50dd-d5dd-47e9-be8c-6fbf51c0ac38)

ここまでで、以下の項目をメモ帳などに残しているか確認してください。

1. BLOB Access key
2. BLOB Storage Account name
3. Document Translation Endpoint Name
4. Document Translation Key

## インポート方法

[最新のソリューション](https://github.com/geekfujiwara/DocumentTranslation/releases/tag/DocumentTranslationApp)をダウンロードします。

ソリューションをインポートを選択します。

![image](https://github.com/user-attachments/assets/a5c1e654-2dd4-40cd-8054-60168953b308)

参照からダウンロードしたソリューションをアップロードします。

![image](https://github.com/user-attachments/assets/e760ec61-db7f-4648-9157-8e1fe503e407)

次へ進みます。

![image](https://github.com/user-attachments/assets/68954a47-9dff-47ce-b055-b66f191415ed)

接続情報を作成します。

![image](https://github.com/user-attachments/assets/ed54b3ee-cf4c-4a68-9740-b20cc406a92b)

こちらでBLOBの接続情報はAccess Key として、先ほど取得したBLOB Storage アカウント名とBLOBのアクセスキーをこちらにペーストして作成ボタンをクリックします。

他のコネクタについてもサインインしていないものがありましたらサインインします。接続がすべて作成できましたら次へ移ります。

![image](https://github.com/user-attachments/assets/703b596b-495a-44f4-9d45-749fdc0197b5)

ストレージアカウントのアカウント名は、接続だけでなくDocument Translation でも利用するため、ストレージアカウント名はもう一度同じものを入力します。その他。先ほど入手したKey とエンドポイント名を入力します。

![image](https://github.com/user-attachments/assets/cca81e6a-3bcf-46ee-b743-18566271fb11)

こちらでインポートの作業は完了です。あとはインポート処理が完了するのを待ちます。

> [!NOTE]
> 少々複雑なアプリのためか、私の環境ではおよそ30分ほどインポートにかかりました。

# テスト実行
インポートが完了すると、緑のような成功メッセージが表示されます。

> [!NOTE]
> 環境によっては言語ラベルについて警告が出ることもありますが、ラベルの警告は問題ありません。

マネージドソリューションとしてインポートされるため、マネージドのタブに切り替えます。

![image](https://github.com/user-attachments/assets/c136db67-a56e-4076-b7e0-5c603bbc9f3b)

アプリを選択して再生します。

![image](https://github.com/user-attachments/assets/9443afb7-91dc-4ab6-9351-aa1821bb6448)

アプリのアクセス許可を行ってください。

![image](https://github.com/user-attachments/assets/c2c3cf96-1e24-4738-a92a-0ff498682e3b)

ドキュメントをアップロードして、翻訳先の言語を選択して、翻訳ボタンを押せばドキュメント全体を翻訳してくれます。

![image](https://github.com/user-attachments/assets/e86cfaf0-62b1-4d09-b65d-2519b5f3e073)

翻訳が成功するとメッセージが表示されます。`翻訳結果を開く`ボタンを押せば、その言語での翻訳結果を見ることができます。

![image](https://github.com/user-attachments/assets/2ab4d94c-66f0-4708-baf2-ef09c8e62fae)

OneDrive のルートフォルダにファイルは保存されています。

![image](https://github.com/user-attachments/assets/1002afdf-95e8-4cac-b4f6-cdf4ecc35145)

ファイル名は、翻訳先の言語コード2文字 + 処理を行った時刻 + 元のファイル名 で出力されます。

> [!NOTE]
> フランス語だと私は読めないのでうまくいっているかわかりませんが、日本語からフランス語にPowerPoint のファイルを変換してくれた例です。
> ![image](https://github.com/user-attachments/assets/28d73858-90a0-475b-8ee3-a5bbdfecd138)

# 制限事項

* Azure Translation サービスの制限は[こちら](https://learn.microsoft.com/ja-jp/azure/ai-services/translator/service-limits#document-translation)に記載があります。「非同期 (バッチ) 操作の制限」を御覧ください。
* こちらのアプリはPower Apps 上であえて制限をかけているのは、ファイル数1とファイルサイズ10MBです。AttachmentControl にてプロパティを変更することで上限を変更することができます。

![image](https://github.com/user-attachments/assets/1624c3d4-fe16-484f-bc60-a33d73874c7b)


以上、ご参考になれば幸いです。




