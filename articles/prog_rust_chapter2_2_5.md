---
title: "Python使用者が読むプログラミングRust 第2章 2.5"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "Python"]
published: true
---
# 概要
Rustを勉強したく[プログラミングRust](https://www.oreilly.co.jp/books/9784873119786/)を買ったので、コードを写経しつつ、自分が慣れているpythonでの実行結果と比較しつつ理解していく。
2章を前後半で分けようとしたが、後半が長くなりすぎそうだったのでさらに細かくわける。

[コードはここ](https://github.com/iusami/programming_rust)

# Webページ公開
Rustのパッケージはcrateと呼ばれ、その内のW一つにWebフレームワークのactix-webがある、これを用いて簡単なWebサーバーを作る。また、途中シリアライズのためにシリアライズ用のserdeも利用する。

まず、Cargo.tomlを以下のように編集して、actix-webやserdeを使えるようにする。


```toml
[package] 
name = "actix-gcd" 
version = "0.1.0" 
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-web = "1.0.8"
serde = { version = "1.0", features = ["derive"]}
```
pythonにおけるpyproject.tomlのようなものなので何となく馴染みがある感じがする。
以下、actix-webとserdeを使って、入力された2つの数字の最大公約数を求めるWebサーバーを作る。

```rust
use serde::Deserialize;
use actix_web::{web, App, HttpResponse, HttpServer};
```
まず、actix-webとserdeから必要なものを使えるようにuse宣言している。


```rust
fn main() {
    let server = HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(get_index))
            .route("/gcd", web::post().to(post_gcd))
    });
    
    println!("Serving on http://localhost:3000");
    server.bind("127.0.0.1:3000").expect("error binding server to address")
    .run().expect("error running server");    
}
```
main関数はHttpServer::newを呼び出しパスを指定して、そのパスへのリクエストに対応するサーバーを作っている。  
ルートにgetリクエストが来るとget_index関数が呼ばれ、"/gcd"　にpostリクエストが来るとpost_gcd関数が呼ばれる。
HttpServer::newの引数として渡すものはRustのクロージャ式である。クロージャ式は関数のように呼び出すことのできる値である。詳細は後の章に譲る。  


```rust
fn gcd(mut n: u64, mut m: u64) -> u64 {
    assert!(n != 0 && m != 0);
    while m != 0 {
        if m < n {
            let t = m;
            m = n;
            n = t;
        }
        m = m % n;
    }
    n
}

fn get_index() -> HttpResponse {
    HttpResponse::Ok()
    .content_type("text/html")
    .body(
        r#"
        <title>GCD Calculator</title>
        <form action="/gcd" method="post">
            <input type="text" name="n" />
            <input type="text" name="m" />
            <button type="submit">Compute GCD</button>
        </form>
        "#,
    )
}
```
続いてはgcd関数とget_index関数の定義である。gcdに関しては入力した2つの整数から最大公約数を求める関数である。  
get_indexでは`HttpResponse::Ok()`を返すことになっているが、これはHTTPの200 OKステータスを表す。続いて`content_type`メソッドと`body`メソッドを呼び出し必要な情報を記載する。  
body部分の値が`get_index`関数の返り値となる。  
レスポンスのテキストにはraw string構文が使われている。これは文字rの後に0個以上の#が続いた後にダブルクォートで始まる文字列で、最後はダブルクォートと同じ数の#で閉じられる。  
これを利用すると任意の文字をエスケープ無しで利用することができるようになる。"や\などを多用するときに便利。

```rust
#[derive(Deserialize)]
struct GcdParameters{
    n: u64,
    m: u64,
}
```
続いてRustの構造体型を定義する。u64のnとmの2つのフィールドを持つ。  
型の宣言の前に`derive`属性が付けられていて、これを付けていればserdeがコンパイル時に型を解析し、HTMLフォームがPOSTリクエストに使用する形式のデータからその型のデータ値を取り出すコードを自動的に生成する。    また、POSTのみならず、JSON、YAML、TOMLなどからも読み取れる。  
実際には`serde-json`などjsonなどに対応するcrateを利用する。  


```rust
fn post_gcd(form: web::Form<GcdParameters>) -> HttpResponse {
    if form.n == 0 || form.m == 0{
        return HttpResponse::BadRequest()
        .content_type("text/html")
        .body("Computing the GCD with zero is boring.");
    }
    let response =
        format!("The greatest common divisor of the numbers {} and {} \
                is <b>{}</b>\n",
                form.n, form.m, gcd(form.n, form.m));
    
    HttpResponse::Ok()
        .content_type("text/html")
        .body(response)
}
```
最後に、post_gcd関数を定義する。  
文字通り、 POSTリクエストを受け取ったとき、その内容を解析して、GcdParameters型のデータ値を取り出して、最大公約数を計算して結果を表示する関数である。  
入力は`web::Form<GcdParameters>`型であるが、`web::Form<T>`は型Tをデシリアライズする方法がわかっていれば、`web::Form<T>`の値を取り出すことができる。ドキュメントには`To extract typed information from request's body, the type T must implement the Deserialize trait from serde.`と書かれていることから、serdeを使ってデシリアライズできるように定義しておくことが前提になっている。  
`format!`マクロは`println!`マクロに似ているが、こちらは標準出力ではなく文字列を返す。そして、その文字列をレスポンスのbodyに与えて返している。

# Python実装
PythonのWebフレームワークとしてFastAPIを自分はよく使うので、FastAPIで書いてみる。


:::details コード全体
```python
from fastapi import FastAPI, Form
from fastapi.responses import HTMLResponse

app = FastAPI()
@app.get("/", response_class=HTMLResponse)
async def get_index():
    data = """
    <title>GCD Calculator</title>
        <form action="/gcd" method="post">
            <input type="text" name="n" />
            <input type="text" name="m" />
            <button type="submit">Compute GCD</button>
        </form>
    """
    return data

@app.post("/gcd", response_class=HTMLResponse)
async def compute_gcd(n: int = Form(...), m: int = Form(...)):
    result = await gcd(n, m)
    data = f"The greatest common divisor of the numbers {n} and {m} \
                is <b>{result}</b>\n)"
    return data

async def gcd(n: int, m: int) -> int:
    assert (n != 0) & (m != 0), "n and m must be non-zero"
    while m != 0:
        if m < n :
            t = m
            m = n
            n = t
        m = m % n
    return n
```
:::

## 個人的な発見
```python
app = FastAPI()
@app.get("/", response_class=HTMLResponse)
async def get_index():
    data = """
    <title>GCD Calculator</title>
        <form action="/gcd" method="post">
            <input type="text" name="n" />
            <input type="text" name="m" />
            <button type="submit">Compute GCD</button>
        </form>
    """
    return data
```
HTMLをレスポンスのボディとしたいときは、`response_class=HTMLResponse`とすればよい。

```python
@app.post("/gcd", response_class=HTMLResponse)
async def compute_gcd(n: int = Form(...), m: int = Form(...)):
    result = await gcd(n, m)
    data = f"The greatest common divisor of the numbers {n} and {m} \
                is <b>{result}</b>\n)"
    return data
```
Formデータを入力としたいときは`Form`を用いる。入力値の名称はフィールドの名称と一致していなければならない。  



