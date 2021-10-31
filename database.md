## データベースへの接続とSQLの実行
### データベース
- データを永続化するためのソフトウェア
- RDB
  - 表形式
- SQL
  - データベース用の問い合わせ言語
  - データをCRUDするためのクエリを記述するための言語


### database/sqlパッケージ
- RDBにアクセスするためのパッケージ
  - 共通機能を提供
    - クエリの発行
    - トランザクション
  - データベースの種類毎のにドライバが存在

### ドライバの登録
- ドライバ
  - 各種RDBに対応したドライバ
  - MySQLなど
- インポートするたけで登録される
  - init処理の中で登録される
  - パッケージ自体は直接使わない

``` go
import _ "modernc.org/sqlite"
```

### SQLite
- ファイルベースのDB
  - 軽量なRDB

### DBのオープン
- Open関数を使用する

``` go
db, err := sql.Open("sqlite", "database.db")
```

- *sql.DBの特徴
  - 複数のゴールーチンから使用可能
  - コネクションプール機能
  - 一度開いたら使い回す
  - めったにcloseしない

### SQLの実行
- *sql.DBのメソッドを使用

``` go
// INSERTやDELETEなど
func (db *DB) Exec(query string, args ...interface{}) (Result, error)

// SELECTなどで複数レコードを取得する場合
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)

// SELECTなどで1つのレコードを取得する場合
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

### テーブルの作成
- (*sql.DB).Execを使う

### レコードの挿入
- AUTOINCREMENTのIDは*sql.Resultから取得できる

### レコードの更新
- 更新したレコード数は*sql.Resultから取得

## トランザクション
### トランザクションとは
- トランザクション
  - 分割できないDB上の一連の処理
  - 複数のクエリにまたがる場合
- コミット
  - トランザクションの処理を確定させる
- ロールバック
  - トランザクションの処理をキャンセルさせる
- トランザクションとロック
  - データの一貫性を保つためにロックをとる。

### トランザクションの開始
- (*sql.DB).Beginを呼ぶ

``` go
// トランザクションを開始する
func (db *DB) Begin() (*Tx, error)
```

### トランザクションに対する処理
- *sql.Txのメソッドを使用

``` go
// INSERTやDELETEなど
func (tx *Tx) Exec(query string, args ...interface{}) (Result, error)

// SELECTなどで複数レコードを取得する場合
func (tx *Tx) Query(query string, args ...interface{}) (*Rows, error)

// SELECTなどで1つのレコードを取得する場合
func (tx *Tx) QueryRow(query string, args ...interface{}) *Row

// コミット
func (tx *Tx) Commit() error

// ロールバック
func (tx *Tx) Rollback() error
```
