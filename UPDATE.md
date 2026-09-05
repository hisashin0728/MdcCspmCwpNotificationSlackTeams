# 目的
- Slack 向け通知について、MDC の推奨事項やアラートを読み取り、複数のチャネルに振り分けて通知を実装したい

# Slack側の環境
- Slack 側は通知する channel を各サブスクリプション毎に cspm / cwpp 分作成している
- 各チャネルのイメージ
    #sub-azuremgmt-cspm
    #sub-azuremgmt-cwpp
    #sub-azurevnet-cspm
    #sub-azurevnet-cwpp
- Incoming Webhook はSlackアプリを用いて、各チャネル用に払い出し済み

# ロジックアプリのイメージ
- Case 分岐を用いて、ASC 推奨事項/アラートから送られるサブスクリプション情報が xxxx であれば RESTAPI で xxxx に渡す・・といった方式で作成する
- サブスクリプションは30個程度なので、case文に直接サブスクリプション情報と IncomingWebhook先を振り分けるように設定する

# 現環境
admin@azurecsa.net
CSPM用
 - subscription : ME-MngEnvMCAP780637-AzureMgmt
  - Incoming Webhook URL : <AzureMgmt CSPM channel webhook URL>
 - subscription : ME-MngEnvMCAP780637-AzureVnet
  - Incoming Webhook URL : <AzureVnet CSPM channel webhook URL>

CWPP用(URLは同一だが、テストのため)
 - subscription : ME-MngEnvMCAP780637-AzureMgmt
  - Incoming Webhook URL : <AzureMgmt CWP channel webhook URL>
 - subscription : ME-MngEnvMCAP780637-AzureVnet
  - Incoming Webhook URL : <AzureVnet CWP channel webhook URL>

> 複数サブスクリプション版では編集性を優先して Webhook URL を string パラメーターとして扱う。デプロイ履歴、リソースグループ、Logic App を参照できる RBAC ロールを必要最小限に制限する。
