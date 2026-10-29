# bot-portal

Bot（UBSLEEPY など）の運用をブラウザから行う小さなポータルです。`services.yaml` に登録した
サービスについて、状態表示・ログ閲覧・再起動・`manage.py` の定型操作（status / update / up /
down / backup）・データベースの閲覧・クイズログの集計を行います。

操作できる内容は `services.yaml` の許可リストだけで決まり、実行した操作は監査ログに残ります。
認証は入口のリバースプロキシ（Forward Auth）に任せ、このアプリ自体はループバックにだけ
公開する前提です。Docker ソケットと対象アプリのディレクトリをマウントするため、実質的に
ホストと同じ権限を持ちます。外部へ直接公開しないでください。

## 構成

- `app/` … FastAPI アプリ（`main.py` が画面、`actions.py` が操作の組み立て、`quizlog.py` が集計）
- `services.yaml` … ポータルに出すサービスの登録（コンテナ名・操作・DB・リンクなど）
- `compose.yaml` / `Dockerfile` … 実行用

## 動かし方

```sh
cp .env.example .env      # 必要なら PORTAL_PORT などを調整
docker compose up -d --build
```

`http://127.0.0.1:8095/` で開きます。Forward Auth が付ける `Remote-User` ヘッダーが
無い場合は 401 になります。

## 開発

```sh
python3 -m pip install -r requirements.txt
python3 -m unittest discover -s tests
python3 -m ruff check --select E9,F .
```

テストは `app/actions.py` と `app/quizlog.py` を直接読み込み、許可リスト・DB 閲覧・
クイズ集計のロジックと、`compose.yaml`・`services.yaml` の設定を確認します。
