# GPT-3.5で作るLINE Chatbot「AI am アイアンマン」
## はじめに
### 「AI am アイアンマン」の特徴と使い方
* まるでアイアンマンと会話しているかのようなチャット体験ができます
* 会話履歴を溜めており過去に発言した内容も記憶しているので、違和感なく会話ができます
* ChatGPTでおなじみの「GPT-3.5」を使用しています
* 「会話をリセット」と送信すると、会話履歴をすべて削除することができます

__※会話履歴は発話から1時間後に自動的に削除します（OpenAIの課金額がエグいので・・・）__

### 「AI am アイアンマン」の友だち追加はこちらから
![S_gainfriends_2dbarcodes_GW](https://user-images.githubusercontent.com/71242610/236373265-ade7b5de-26e6-48ef-bb68-94134f11d78b.png)

<a href="https://lin.ee/lo9Rca8"><img src="https://scdn.line-apps.com/n/line_add_friends/btn/ja.png" alt="友だち追加" height="36" border="0"></a>

## 用意するもの
* OpenAI API
* LINE 公式アカウント
* LINE Messaging API
* AWS Lambda関数
* AWS DynamoDB（会話履歴保存用）

## 構築手順
構築にあたり参考にした資料：<https://qiita.com/michitomo/items/a10465b12bcca32bf63a>

1. __OpenAIでキーを発行__
    * リンク：<https://platform.openai.com/signup>
    * シークレットキー、オーガナイゼーションを控えておく
2. __LINE公式アカウント（Messaging API Channel）を開設、各種キーを発行__
    * リンク：<https://www.linebiz.com/jp/entry/>
    * チャンネルシークレット、チャンネルアクセストークンを控えておく
3. __AWSで新規のLambda関数を作成する__
    * ランタイムはPython3.10
    * API GWを通さず直接Lambda関数を叩けるようにするため、関数URLを有効にする（権限はNONE）
        * 作成した関数URLを、LINE公式アカウントのWebhook URLに設定する
    * 控えておいた各種キーを環境変数に設定する
    * デフォルトのタイムアウトが3秒と短いため、2分に設定を変更する
4. __AWSでDynamoDBを作成する__
    * 作成時の設定
        * パーティションキー：user_id（送信者のユーザーID）
        * ソートキー：send_at（日付）
    * 項目の作成（属性の追加）
        * del_time（数値）：毎日0時の削除用、翌0時のunixtimeを設定
        * GPT_reply（文字列）：GPT-3.5のレスポンス
        * message_id（文字列）：ユーザーからのメッセージID
        * user_content（文字列）：ユーザーの発話内容
    * TTL（自動削除）の設定
        * 対象属性：del_time
5. __Lambda関数をデプロイする__
    * 各種ライブラリを使用するため、カスタムレイヤーを追加する
        * （Docker、AWS CLI、AWS SAMを使って構築するため、超面倒・・・）
        * requirements.txtに沿って必要なライブラリをすべてインストールする
        * 参考：<https://nisshingeppo.com/ai/aws-lambda-library-install/>
    * ロールにDynamoDBへのアクセス権限を追加する
        * 追加するポリシー：AmazonDynamoDBFullAccess
    * lambda_function.pyに、作成したコードを貼り付けてデプロイする

## 作成者
* 作成者：K-Ryosuke
* Twitter：[@rsk_142](https://twitter.com/rsk_142)
