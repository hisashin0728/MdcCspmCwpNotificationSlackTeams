# 目的
- Slack 向け通知について、MDC の推奨事項やアラートを読み取り、複数のチャネルに振り分けて通知を実装したい

# Slack側の環境
- Slack 側は通知する channel を各サブスクリプション毎に cspm / cwpp 分作成している
- 各チャネルのイメージ
  #sub-subscription1-cspm
  #sub-subscription1-cwpp
  #sub-subscription2-cspm
  #sub-subscription2-cwpp
- Incoming Webhook はSlackアプリを用いて、各チャネル用に払い出し済み

# ロジックアプリのイメージ
- Case 分岐を用いて、ASC 推奨事項/アラートから送られるサブスクリプション情報が xxxx であれば RESTAPI で xxxx に渡す・・といった方式で作成する
- サブスクリプションは30個程度なので、case文に直接サブスクリプション情報と IncomingWebhook先を振り分けるように設定する

# 現環境
admin@azurecsa.net
CSPM用
 - subscription : <SUBSCRIPTION_1_NAME>
  - Incoming Webhook URL : <SUBSCRIPTION_1_CSPM_SLACK_WEBHOOK_URL>
 - subscription : <SUBSCRIPTION_2_NAME>
  - Incoming Webhook URL : <SUBSCRIPTION_2_CSPM_SLACK_WEBHOOK_URL>

CWPP用(URLは同一だが、テストのため)
 - subscription : <SUBSCRIPTION_1_NAME>
  - Incoming Webhook URL : <SUBSCRIPTION_1_CWP_SLACK_WEBHOOK_URL>
 - subscription : <SUBSCRIPTION_2_NAME>
  - Incoming Webhook URL : <SUBSCRIPTION_2_CWP_SLACK_WEBHOOK_URL>

> 複数サブスクリプション版では編集性を優先して Webhook URL を string パラメーターとして扱う。デプロイ履歴、リソースグループ、Logic App を参照できる RBAC ロールを必要最小限に制限する。
