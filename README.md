AfterEffectsスクリプトパッケージ管理システム
AfterEffectScriptを統合的に管理するものを作る

## 解決したいこと
### ユーザー
- スクリプトの管理が面倒
  - 他の端末に移動した時とかにつらい
- スクリプトのインストールを簡単にしたい
- AEのバージョン上げた時に面倒

### 開発者
- 自動でアップデートさせたい
- スクリプトをもっと使われやすくしたい

## やりたいこと
### ユーザー
- GUIでスクリプトを管理、実行できる
  1. 追加、一覧、単一／一括アップデート
  2. 詳細情報を見れる(WEBに飛ぶでよいかな)
  3. よく使うものや検索なども付けたい
  4. スクリプト同士の依存関係は考えない
  5. 一覧からrunしたい(ScriptUIかどうか問題)(実行のときにどうする？)
  6. フィードバックとか出来てもいいかもね

- 雑に「このzxpいれてこの設定コピペしてinstallしてね」で終わるUXにしたい
- 「このパッケージ名入れてinstallしてね」で終わるUXにしたい

### 開発者
- 開発者がスクリプトの情報を閲覧/更新できる
  1. WEBから新規登録、更新ができる
  2. インストール数の情報、フィードバックとかがみたい
  3. 特定バージョン使用者にアラート表示とか？

- 開発側はちょっと敷居高くてもいいかな

## 実装
### AE
- 基本的にユーザーが使う `.zxp`
- 拡張機能(HTML5)で実装
- ~~localstorageにpackage.json的なものを持たせる~~
- AEのバージョン変わった時にコピペさせるよりかはローカルにpackage.json的なもの持たせたほうが良さそう
- いい感じに全バージョン共通で同じjsxファイル呼び出せるようにできれば最高ではあるかも

- API駆動、vue.js +  ExtendScript(CSInterface)
- オフラインでも動作はさせる `navigator.onLine `

### WEB
- RailsでWEBとAPIを実装
- ユーザー: スクリプトの情報が見れる(、インストール方法がわかる)
- 開発者: スクリプトの登録、情報更新を行える
- サーバーはHerokuとかで作るかレンタルする
- (定期クロールして同じものがDLできるかを担保したい)

## Database
### Developers
|名前|データ型|説明|
|---|---|---|
|id|int||
|name|varchar(100)|表示名|
|password| varchar(255) |暗号化パスワード|
|email|varchar(100)|メールアドレス|
|url|varchar(100)|ユーザー情報用URL|
|created|datetime||
|modified|datetime||

### Products
|名前|データ型|説明|
|---|---|---|
|id|int||
|name|varchar(100)|case-insensitive/[a-zA-Z0-9_\-]|
|brief|varchar(100)|完結な説明|
|description|text|説明|
|url|varchar(100)|URL|
|developer_id|int|製作者のid|
|source_url|varchar(255)|jsxが置いてあるURL|
|version|varchar(16)|4.2.3|
|installed_count|int|インストールされた回数|
|created|datetime||
|modified|datetime||

```
旧バージョン対応はしたくない
```

## URLs
### API
- `GET` /api/v1/fetch/
  - `param` json 使用中のpackages
  - `return` json 更新されるべきpackages
- `GET` /api/v1/fetch/:name
  - `param` :name パッケージ名
  - `return` 合致したpackageかfailなjson
- `GET` /api/v1/developers/:id
  - 表示用
- `GET` /api/v1/products/
  - 表示用
- `GET` /api/v1/products/:id
  - 表示用

### WEB
- `GET` /
  - トップページ、概要とインストール方法とか
- `GET` /products/
  - 登録されたスクリプト一覧
- `GET` /products/:id
  - スクリプトの詳細表示
- `GET` /developers/:id
  - 開発者情報
- `GET` /products/
  - 登録されたスクリプト一覧
- `GET` /products/:id
  - スクリプトの詳細表示
- `GET` /developers/:id
  - 開発者情報
- `GET` /search/:q
  - 検索 開発者名/スクリプト名/(あいまい)

### 管理側
- `GET` /cms/login
  - 検索 開発者名/スクリプト名/(あいまい)
- `GET` /cms/
  - 開発者コンソール
- `GET` /cms/products/:id
  - スクリプト情報
- `GET` /cms/products/:id/edit
  - 更新
- `GET` /cms/account/edit
  - アカウント情報更新
