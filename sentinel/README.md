# Cybozu Audit Logs - Microsoft Sentinel Data Connector

> **[日本語](#日本語)** | **[English](#english)**

---

## 日本語

Cybozu の監査ログを Microsoft Sentinel に継続的に取り込むデータコネクタです。Codeless Connector Platform (CCP) を利用し、Azure テナント内で完結して動作します。

接続後、コネクタは約 10 分間隔でポーリングし、ページネーション・レート制限・差分同期を自動的に処理します。

### アーキテクチャ

```
Cybozu  ──►  Microsoft Sentinel (CCP / RestApiPoller)  ──►  CybozuAuditLogs_CL テーブル
                                  ▲
                           約10分間隔でポーリング
                           カーソル: last_run_time
```

> `last_run_time` は CCP が内部的に管理するカーソルで、前回ポーリングが成功した時刻を保持します。これにより、毎回のポーリングで前回以降の新規ログのみを取得します。ただし、ポーリングのタイミングによっては重複して取り込まれる可能性があります。

- アクティブな **Azure サブスクリプション**（リソースのデプロイ権限あり）
- **Microsoft Sentinel** が有効化された Log Analytics ワークスペース（または新規作成）
- **Cybozu** 環境と監査ログへのアクセス
  - 監査ログ読み取り権限を持つ API トークン
- デプロイ対象のリソースグループに対するリソース作成権限

### セットアップ手順

セットアップは **デプロイ** と **接続** の 2 段階です。

#### 1. デプロイ

以下のボタンをクリックすると、Azure ポータルでテンプレートのデプロイ画面が開きます。

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/PLACEHOLDER_DEPLOY_URL)

デプロイ時に以下のリソースが作成されます:
- **Log Analytics ワークスペース**（新規作成、または既存を使用）
- **Microsoft Sentinel** の有効化
- **Data Collection Endpoint (DCE)**
- **Sentinel データコネクタ定義**（Sentinel UI へのコネクタ登録）

| パラメータ          | 説明                              | デフォルト値                  |
|-----------------|----------------------------------|-------------------------|
| `Workspace Name` | Log Analytics ワークスペース名          | *(必須)*                  |
| `Dce Name`      | Data Collection Endpoint の名前     | `dce-cybozu-audit-logs` |
| `Dcr Name`      | Data Collection Rule の名前         | `dcr-cybozu-audit-logs` |

#### 2. 接続

デプロイ完了後、Microsoft Sentinel のデータコネクタ画面から接続します。

1. Microsoft Sentinel > **データコネクタ** を開く
2. **Cybozu Audit Logs** を選択し、コネクタページを開く
3. Domain URL と API トークンを入力して **Connect** をクリック

接続時に以下のリソースが作成されます:
- **CybozuAuditLogs_CL** カスタムログテーブル
- **Data Collection Rule (DCR)**
- **RestApiPoller**（ポーリング開始）

切断する場合は **Disconnect** をクリックしてください。再接続する場合は Domain URL と API トークンを再入力して **Connect** をクリックしてください。

### ログのクエリ

接続後、数分でデータの取り込みが始まります。Log Analytics で以下のクエリを実行して確認できます:

```kusto
CybozuAuditLogs_CL
| sort by TimeGenerated desc
| take 50
```

### ライセンス

MIT License - Copyright (c) 2026 Cybozu

[MIT](../LICENSE)

---

## English

A Microsoft Sentinel data connector that continuously ingests Cybozu audit logs into your Log Analytics Workspace using the Codeless Connector Platform (CCP). The connector runs entirely within your Azure tenant.

Once connected, the connector polls Cybozu at approximately 10-minute intervals, automatically handling pagination, rate limiting, and incremental synchronization.

### Architecture

```
Cybozu  ──►  Microsoft Sentinel (CCP / RestApiPoller)  ──►  CybozuAuditLogs_CL table
                                  ▲
                           Polls every ~10 min
                           Cursor: last_run_time
```

> `last_run_time` is an internal cursor managed by CCP that stores the timestamp of the last successful poll. This ensures each polling cycle only retrieves new logs since the previous run. Note that depending on polling timing, duplicate ingestion may occur.

- An active **Azure subscription** with permissions to deploy resources
- A **Microsoft Sentinel**-enabled Log Analytics Workspace (or create a new one)
- **Cybozu** environment with audit log access
  - API token with audit log read permissions
- Permissions to create resources in the target resource group

### Setup

Setup consists of two steps: **Deploy** and **Connect**.

#### 1. Deploy

Click the button below to open the deployment form in the Azure portal.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/PLACEHOLDER_DEPLOY_URL)

The following resources will be created on deploy:
- **Log Analytics Workspace** (new or existing)
- **Microsoft Sentinel** onboarding
- **Data Collection Endpoint (DCE)**
- **Sentinel data connector definition** (registers the connector in the Sentinel UI)

| Parameter        | Description                         | Default                 |
|------------------|-------------------------------------|-------------------------|
| `Workspace Name` | Name of the Log Analytics Workspace | *(required)*            |
| `Dce Name`       | Name of the Data Collection Endpoint | `dce-cybozu-audit-logs` |
| `Dcr Name`       | Name of the Data Collection Rule    | `dcr-cybozu-audit-logs` |

#### 2. Connect

After deployment, connect the connector from the Microsoft Sentinel data connectors page.

1. Open Microsoft Sentinel > **Data connectors**
2. Select **Cybozu Audit Logs** and open the connector page
3. Enter the Domain URL and API token, then click **Connect**

The following resources will be created on connect:
- **CybozuAuditLogs_CL** custom log table
- **Data Collection Rule (DCR)**
- **RestApiPoller** (polling starts)

To disconnect, click **Disconnect**. To reconnect, re-enter the Domain URL and API token, then click **Connect**.

### Querying Logs

After connecting, data should start appearing within a few minutes. Run the following query in Log Analytics to verify:

```kusto
CybozuAuditLogs_CL
| sort by TimeGenerated desc
| take 50
```

### License

MIT License - Copyright (c) 2026 Cybozu

[MIT](../LICENSE)