# ddd-aws-nodejs-samples (Docker Only)

---

## ダウンロード
- 最新版（ZIP）
  - 基本サンプル一式：  
    [ddd-aws-nodejs-samples.zip](https://github.com/K-Kain/practical-ddd-aws-nodejs/releases/latest/download/ddd-aws-nodejs-samples.zip)
  - 4層本番形スターター：  
    [ddd-4layer-demo.zip](https://github.com/K-Kain/practical-ddd-aws-nodejs/releases/latest/download/ddd-4layer-demo.zip)


## 付録ダウンロード
- すべてまとめて: [appendices-all.zip](https://github.com/K-Kain/practical-ddd-aws-nodejs/releases/latest/download/appendices-all.zip)


DDD×AWS（DynamoDB / Lambda / EventBridge / API Gateway）の最小サンプルを **Docker だけ**で動かします。  
AWSアカウント／資格情報の設定は不要です。

## 前提
- Docker Desktop がインストール済み
- ポート **4566** が空いていること（LocalStack が使用）

## 起動
```bash
unzip ddd-aws-nodejs-samples.zip -d ddd-aws-nodejs-samples
cd ddd-aws-nodejs-samples
docker compose up --build
```
> ログに `Ready.` と **Host endpoint**（例：`http://localhost:4566/_aws/execute-api/<API_ID>/dev/orders`）が表示されたら準備完了です。

## 叩いてみる（別ターミナル）
```bash
cd ddd-aws-nodejs-samples
API_URL=$(docker compose exec -T localstack cat /shared/endpoint-host.txt)
curl -i -X POST "$API_URL" \
  -H 'Content-Type: application/json' \
  -d '{"customerId":"cust_local","amount":264,"currency":"JPY"}'
```
- 201 Created が返れば成功です。
- `endpoint-host.txt` は起動時に LocalStack が自動で書き出します。

## 動作確認（任意）
```bash
# DynamoDB に保存されたレコードを確認
docker compose exec localstack awslocal dynamodb scan --table-name Orders --max-items 5

# Lambda のログを追う
docker compose exec localstack awslocal logs tail /aws/lambda/place-order --since 5m --follow
```

## 後片付け
```bash
# 停止
docker compose down

# まっさらからやり直す（ボリュームも削除）
# docker compose down -v && docker compose up --build
```

## トラブルシュート
- **ポート 4566 が使用中**  
  他の LocalStack/プロセスを停止してください（例：`lsof -i :4566`）。
- **起動直後の 502 / 接続拒否**  
  初回だけ Lambda が `Pending` の場合があります。数秒待ってから再度 `curl` を実行してください。
- **エンドポイントが分からない**  
  ```bash
  docker compose logs --tail=200 localstack | sed -nE 's#.*(http://localhost:4566/_aws/execute-api/[a-z0-9]+/dev/orders).*#\1#p' | tail -1
  ```

## 補足
- `src/interface/handlers/http/placeOrder.ts` を配置すると自動ビルドされます（未配置時は同等のサンプル実装で起動）。
- EventBridge にルールは未設定のため、ログに `No rules attached to event_bus` が出ても正常です。
- 本サンプルは LocalStack の **REST API (v1)** を利用し、URLは新形式 `/_aws/execute-api/<API_ID>/dev/orders` を案内します。

