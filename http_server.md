## HTTPサーバをたてる
- net/httpパッケージ
- HTTPサーバ
- HTTPクライアント

### HTTPサーバ作成の流れ
- HTTPハンドラの作成
  - 1つのHTTPリクエストを処理する関数(=ハンドラ)を作成する
  - 1リクエストあたりの処理は並行に動く(go routine)
- HTTPハンドラとエントリポイントの結びつけ
  - "/index"などのエントリポイントとハンドラを結びつける
  - ルーティングとも呼ばれる
- HTTPサーバの起動
  - ホスト名(IP)とポート番号、ハンドラを指定してHTTPサーバを起動する

### HTTPハンドラ
- 引数にレスポンスを書き込む先とリクエストを取る
  - 第一引数はレスポンスを書き込む先
    - 書き込みにはfmtパッケージの関数などが使える
  - 第二引数はクライアントからのリクエスト
- HTTPハンドラはインターフェースとして定義されている
  - ServeHTTPメソッドを実装していればハンドラとして扱われる

``` go
type Handler interface {
  ServerHTTP(ResponseWriter, *Request)
}
```

###  http.Handleでハンドラを登録
- パターンとhttp.Handlerを指定して登録する
  - 第一引数としてパターンを指定する
  - 第二引数としてhttp.Handlerを指定する
  - http.DefaultServeMuxに登録される

``` go
func Handle(pattern string, handler Handler)
```

### http.HandleFuncでハンドラを登録
- パターンと関数を指定して登録する
  - 第一引数としてパターンを指定する
  - 第二引数として関数を指定する
  - http.DefaultServeMuxに登録される

``` go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

### http.HandlerFunc
- 関数にhttp.Handlerを実装させている

``` go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP (w ResponseWriter, r *Request) {
  f(w, r)
}
```

- http.HandlerFunc
  - 引数で受け取った関数をhttp.HandlerFuncに変換(キャスト)する
  - http.Handleでhttp.Handlerとして登録する

  ``` go 
  func HandleFucn(pattern string, handler func (ResponseWriter, *Request)){

  }
  ```
- Handerは登録されるもの
- Handleは登録する関数

### http.ServeMux
  - 複数のハンドラをまとめる
  - パスによって使うハンドラを切り替える
  - 自身もhttp.Handlerを実装している
  - http.Handleとhttp.HandleFuncはデフォルトのhttp.ServeMuxであるhttp.DefaultServeMuxを使用している

### HTTPサーバの起動
- http.ListernAndServeを使う
  - 第一引数でホスト名とポート番号を指定
    - ホスト名を省略した場合localhost
  - 第二引数でHTTPハンドラを指定
    - nilで省略した場合はhttp.HandleFuncなどで登録したハンドラが使用される

``` go 
http.ListernAndServe(":8080", nil)
```

## HTTPレスポンスとリクエストについて

### http.ResponseWriter
- http.ResponseWriterインターフェース
  - io.Writerと同じWriteメソッドをもつ
    - ResponseWriterを満たすとio.Writerを満たす
  - io.Writerとしても振る舞える
    - fmt.Fprint*の引数に取れる
    - json.NewEncoderの引数に取れる
- インターフェースなのでモックも作りやすい=テスト簡単

### エラーを返す
- http.Error関数を使う
  - エラーメッセージとステータスコードを指定する
  - ステータスコードは定数としてhttpパッケージで定義されている
    - http.StatusOKやhttp.StatusInternalServerError

``` go 
func Error(w ResponseWriter, error string, code int) {}
```

### JSONを返す
- encoding/jsonパッケージを使う
  - 機械的に処理しやすいJSONをレスポンスに用いる場合も多い
  - JSONエンコーダを使ってGoの値をJSONに変換する
  - 構造体やスライスをJSONのオブジェクトや配列にできる

### JSONエンコード
- json.Encoder型を使う

``` go 
type Person struct {
  Name string `json: "name"`
  Age int `json: "age"`
}

p := &Person{Name: "tenntenn", Age: 32}

var buf bytesBuffer
enc := json.NewEncoder(&buf)
if err := enc.Encode(p); err != nil {log.Fatal(err)}
fmt.Println(buf.String())

```

### JSONデコード
- json.Decoderを使う

``` go 
var p2 Person
dec := json.NewDecoder(&buf)
if err := dec.Decode(&p2); err != nil {
  log.Fatal(err)
}
fmt.Println(p2)
```

### レスポンスヘッダーを設定する
- ResponseWriterのHeaderメソッドを使う
  - WriteやWriteHeaderを呼び出した後に設定しても効果がない

``` go 
func handler(w http.ResponseWriter, req *http.Request){
  w.Header().Set("Content-Type", "application/json; charset=utf-8")
  v := struct {
    Msg string `json: "msg"`
  }{
    Msg: "hello",
  }
  if error := json.NewEncoder(w).Encode(v); err != nil {
    log.Println("Error:",err)
  }
}
```

### リクエストパラメータの取得
- (*http.Request).FormValueから名前を指定して取得
  - パラメータ指定の例: http://localhost:8080?msg=Gophers
  - 複数ある場合は&でつなぐ
    - http://localhost:8080?a=100&b=200

``` go 
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
  fmt.Fprintln(w, "hello", r.FormValue("msg"))
})
```

### リクエストボディの取得
- (*http.Request).Bodyから取得する

### リクエストヘッダの取得
- RequestのHeaderフィールドを使う
  - Getメソッドを使うとヘッダー名を指定して取得する

### テンプレートエンジンの使用
- html/template
  - Go標準のテンプレートエンジン
  - text/templateのHTML特化版
- テンプレートの生成

``` go 
template.Must(template.New("sign").Parse("<html><body>{{.}}</body></html>"))
```

- テンプレートに埋め込む

``` go 
tmpl.Execute(w, r.FormValue("content"))
```


## HTTPハンドラのテスト
### net/http/httptestを使う
- ハンドラのテストのための機能などを提供
- httptest.ResponseRecorder
  - http.ResponseWriterインターフェースを実装している
- NewRequestメソッド
  - 簡単にテスト用のリクエストが作れる

## HTTPクライアント
### HTTPリクエストを送る 
- http.DefaultClientを用いる
  - デフォルトのHTTPクライアント
  - http.Getやhttp.Postはhttp.DefaultClientのラッパー

``` go 
resp, err := http.Get("http://example.com/")

resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)

v := url.Values{"key": {"Value"}, "id": {"123"}}
resp, err := http.PostForm("http://example.com/form", v)
```

### レスポンスを読み取る
- (*http.Response).Bodyを使う
  - ioReadCloserを実装している
  - 読み込んだらCloseメソッドを呼ぶ

### リクエストを指定する
- http.Client.Doを用いる

### リクエストとコンテキスト
- *http.Requestから取得する

```go
ctx := req.Context() 
```

### http.Clientとhttp.Transport
- http.Transport型
  - http.RoundTripperを実装した型
  - HTTP/HTTPS/HTTPプロキシに対応している
  - コネクションのキャッシュを行う
  - 実際のHTTP通信のところ

### http.DefaultTransport
- デフォルトで設定されている定数

### http.RountTripper
- HTTPのトランザクションを行うインターフェース
  - 実装しているもの
    - http.Transport
    http.NewFileTransportで取得できる値
  - ※ラウンドトリップ： 行って帰ってくる
