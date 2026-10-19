> Prometheus ServerのAlertmanager設定でiSeeNowのサービスアドレスを設定し、iSeeNowを使用してAlertmanagerを置き換え、Prometheus Serverのアラート情報を受信および処理します。

### Prometheus サービスの設定

- Prometheusの設定ファイル prometheus.yml を編集し、iSeeNowをアラートの受信先として追加します。
```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - {hertzbeat_host}:1157
      authorization:
        type: 'Bearer'
        credentials: '{token}'

```
- `{hertzbeat_host}:1157` は、iSeeNow Server のアドレスとポートです。実際の環境に合わせて設定を修正し、ネットワーク接続が確立されていることを確認してください。
- `{token}` は、iSeeNow Server の認証トークンです。新しいトークンを申請した後、置き換えてください。
- Prometheus Server を再起動してください。

## 設定の検証

1. Prometheus の設定が正しいことを確認し、設定を再読み込みしてください。
    ```bash
    curl -X POST http://localhost:9090/-/reload
    ```
2. Prometheusアラートルールの状態を確認する。
    ```bash
    curl http://localhost:9090/api/v1/rules
    ```
3. テストアラートをトリガーし、iSeeNowアラートセンターで表示を確認する 。

## FAQ

- PrometheusサーバーからiSeeNowのURLへアクセス可能であること。
- Prometheusログ内にアラート送信失敗のエラーメッセージがないか確認する。
- アラートルールの式が正しいことを検証する。

詳細情報は [Prometheusアラート設定ドキュメント](https://prometheus.io/docs/alerting/latest/configuration/) を参照してください
