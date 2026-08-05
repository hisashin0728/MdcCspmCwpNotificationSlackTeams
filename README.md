# Microsoft Defender for Cloud notifications for Slack and Microsoft Teams

Microsoft Defender for Cloud の Workflow Automation から Logic Apps を起動し、CWP セキュリティアラートと CSPM セキュリティ推奨事項を Slack または Microsoft Teams のチャネルへ通知する ARM テンプレートです。

- Slack: Incoming Webhook を使用した Block Kit 通知
- Microsoft Teams: 標準 Teams コネクタを使用した Adaptive Card 通知
- CWP: 通知するアラート重要度を選択可能
- CSPM: すべて、または指定した Assessment ID の unhealthy な推奨事項を通知
- Workflow Automation の対象範囲: デプロイ先リソースグループが属するサブスクリプション全体

## 通知イメージ

### CWP セキュリティアラート

| Slack Block Kit | Microsoft Teams Adaptive Card |
| --- | --- |
| ![Slack CWP notification](samples/cwp-slack-notification.png) | ![Teams CWP notification](samples/cwp-teams-notification-sample.png) |

### CSPM セキュリティ推奨事項

| Slack Block Kit | Microsoft Teams Adaptive Card |
| --- | --- |
| ![Slack CSPM notification](samples/cspm-slack-notification.png) | ![Teams CSPM notification](samples/cspm-teams-notification-sample.png) |

画像内のリソース名、サブスクリプション名、日時などはサンプルデータです。

## テンプレートをデプロイ

通知先とイベント種別を選び、対応する **Deploy to Azure** ボタンからデプロイしてください。Azure portal でサブスクリプション、リソースグループ、リージョン、通知先などのパラメーターを入力できます。

| 通知先 | イベント | 通知条件 | ARM テンプレート | デプロイ |
| --- | --- | --- | --- | --- |
| Slack | CWP セキュリティアラート | `alertSeverities` で指定した重要度 | [cwp-template.json](cwp-template.json) | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FMdcCspmCwpNotificationSlackTeams%2Fmain%2Fcwp-template.json) |
| Slack | CSPM セキュリティ推奨事項 | unhealthy、必要に応じて Assessment ID を指定 | [cspm-template.json](cspm-template.json) | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FMdcCspmCwpNotificationSlackTeams%2Fmain%2Fcspm-template.json) |
| Microsoft Teams | CWP セキュリティアラート | `alertSeverities` で指定した重要度 | [teams-cwp-template.json](teams-cwp-template.json) | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FMdcCspmCwpNotificationSlackTeams%2Fmain%2Fteams-cwp-template.json) |
| Microsoft Teams | CSPM セキュリティ推奨事項 | unhealthy、必要に応じて Assessment ID を指定 | [teams-cspm-template.json](teams-cspm-template.json) | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FMdcCspmCwpNotificationSlackTeams%2Fmain%2Fteams-cspm-template.json) |

> [!IMPORTANT]
> Deploy to Azure ボタンは、テンプレートが GitHub の `main` ブランチへ公開された後に利用できます。

## デプロイされるリソース

| リソース | Slack | Microsoft Teams | 用途 |
| --- | ---: | ---: | --- |
| `Microsoft.Logic/workflows` | 1 | 1 | 通知ワークフロー |
| `Microsoft.Web/connections` | 1 | 2 | Defender for Cloud トリガー、Teams コネクタ |
| `Microsoft.Security/automations` | 1 | 1 | Defender for Cloud Workflow Automation |

Teams テンプレートでは Defender for Cloud と Microsoft Teams の API 接続を1つずつ作成します。

## 前提条件

- 対象サブスクリプションで必要な Microsoft Defender for Cloud プランが有効であること
- リソースグループへ Logic App と API 接続を作成できる権限
- Defender for Cloud の Workflow Automation を作成できる権限
- Slack: Slack アプリで発行した Incoming Webhook URL
- Microsoft Teams: 対象チーム/チャネルへ投稿できる Microsoft 365 アカウント、Team ID、Channel ID

## デプロイパラメーター

### 共通

| パラメーター | 必須 | 説明 | 既定値 |
| --- | --- | --- | --- |
| `location` | いいえ | Logic Apps、API 接続、Workflow Automation のリージョン | リソースグループのリージョン |
| `namePrefix` | いいえ | 作成するリソース名のプレフィックス | Slack: `mdc-slack`、Teams: `mdc-teams` |
| `tags` | いいえ | 作成するリソースへ付与するタグ | テンプレート内の既定タグ |

### Slack

| パラメーター | 必須 | 説明 |
| --- | --- | --- |
| `slackWebhookUrl` | はい | Slack Incoming Webhook URL。ARM の `securestring` として扱われます。 |

### Microsoft Teams

| パラメーター | 必須 | 説明 |
| --- | --- | --- |
| `teamId` | はい | 投稿先チャネルを含む Team ID (Microsoft 365 Group ID) |
| `channelId` | はい | 投稿先の Channel ID |

### CWP

| パラメーター | 必須 | 説明 | 既定値 |
| --- | --- | --- | --- |
| `alertSeverities` | いいえ | 通知する重要度。`High`、`Medium`、`Low` を指定できます。 | 3重要度すべて |

### CSPM

| パラメーター | 必須 | 説明 | 既定値 |
| --- | --- | --- | --- |
| `recommendationIds` | いいえ | 通知対象の Assessment ID (GUID) の配列。空配列の場合はすべての unhealthy な推奨事項を通知します。 | `[]` |

Assessment ID は Azure Resource Graph Explorer で確認できます。

```kusto
securityresources
| where type =~ 'microsoft.security/assessments'
| project recommendationId = name, recommendation = properties.displayName
| order by recommendation asc
```

## Teams API 接続を認証

ARM デプロイだけでは Microsoft Teams コネクタの OAuth 認証は完了しません。Teams テンプレートをデプロイした後、次の手順を実行してください。

1. Azure portal でデプロイ先のリソースグループを開きます。
2. `<namePrefix>-teams` という名前の API 接続を開きます。
3. 「API 接続の編集」から Microsoft 365 アカウントでサインインします。
4. 接続を保存し、状態が「接続済み」であることを確認します。

CWP と CSPM を同じリソースグループへ同じ `namePrefix` でデプロイした場合は、同じ Teams API 接続を使用します。

## Azure CLI でデプロイ

パラメーターファイルを編集し、Azure CLI からデプロイすることもできます。

```powershell
$resourceGroup = '<RESOURCE-GROUP-NAME>'
$location = '<AZURE-REGION>'

az group create --name $resourceGroup --location $location

az deployment group create `
  --resource-group $resourceGroup `
  --template-file .\teams-cwp-template.json `
  --parameters .\teams-cwp-template.parameters.json
```

Slack の場合は Webhook URL をソース管理へ保存せず、実行時に渡してください。

```powershell
$slackWebhookUrl = Read-Host 'Slack Incoming Webhook URL'

az deployment group create `
  --resource-group $resourceGroup `
  --template-file .\cwp-template.json `
  --parameters .\cwp-template.parameters.json `
  --parameters slackWebhookUrl=$slackWebhookUrl
```

## 動作確認

1. Defender for Cloud の「ワークフローの自動化」で Automation が有効になっていることを確認します。
2. Teams の場合は、Teams API 接続が「接続済み」であることを確認します。
3. Defender for Cloud のアラートまたは推奨事項から「ロジック アプリのトリガー」を実行します。
4. Logic Apps の実行履歴が成功していることを確認します。
5. 対象の Slack または Teams チャネルに通知されたことを確認します。

## セキュリティ上の注意

- Slack Incoming Webhook URL はシークレットです。実値をパラメーターファイルやソース管理へコミットしないでください。
- API 接続には通知に使用する専用アカウントを利用し、対象チームへの必要最小限の権限を付与してください。
- デプロイ前にテンプレートと Workflow Automation のサブスクリプションスコープを確認してください。

## ファイル構成

| ファイル | 説明 |
| --- | --- |
| [cwp-template.json](cwp-template.json) | Slack CWP ARM テンプレート |
| [cspm-template.json](cspm-template.json) | Slack CSPM ARM テンプレート |
| [teams-cwp-template.json](teams-cwp-template.json) | Teams CWP ARM テンプレート |
| [teams-cspm-template.json](teams-cspm-template.json) | Teams CSPM ARM テンプレート |
| [samples/](samples/) | Slack/Teams の通知サンプル画像と再現用 HTML |

## 参考資料

- [Defender for Cloud のワークフローの自動化](https://learn.microsoft.com/azure/defender-for-cloud/workflow-automation)
- [Azure Resource Manager テンプレートのデプロイ](https://learn.microsoft.com/azure/azure-resource-manager/templates/deploy-to-azure-button)
- [Microsoft Teams コネクタ](https://learn.microsoft.com/connectors/teams/)
- [Slack Block Kit](https://api.slack.com/block-kit)
- [Adaptive Cards](https://adaptivecards.io/)