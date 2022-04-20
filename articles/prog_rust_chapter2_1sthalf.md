---
title: "python使用者が読むプログラミングRust 第2章 前半"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "Python"]
published: true
---
# 概要
Rustを勉強したく[プログラミングRust](https://www.oreilly.co.jp/books/9784873119786/)を買ったので、コードを写経しつつ、自分が慣れているpythonでの実行結果と比較しつつ理解していく。


# Toyプログラム

2つの整数の最大公約数を計算するプログラムを書いてみる。

```rust
fn gcd(mut n: u64, mut m: u64) -> u64{
    assert!(n !=0 && m !=0);
    while m !=0 {
        if m < n{
            let t = m;
            m = n;
            n = t;
        }
        m = m % n;
    }
    n
}
```
mutで可変であることを示す。数値の型は以下の通り。
|型名|意味|
|--| --|
|i32|符号あり32ビット整数|
|u64|符号なし64ビット整数|
|u8|符号なし8ビット整数|
|f32|単精度浮動小数点|
|f64|倍精度浮動小数点|

「!」が付くものはマクロ呼び出しであることを示す。assert!は引数が真であることを確認する。真でなければメッセージを出してプログラムを突然終了させる。このような終了はパニックと呼ばれる。  
while文において、条件式を()で囲む必要はない。制御対象文は{}で囲われる。

pythonで書いたらこんな感じか

```python
def gcd(n: int, m: int) -> int:
    assert (n != 0) & (m != 0), "n and m must be non-zero"
    while m != 0: 
        if m < n :
            t = m
            m = n
            n = t
        m = m % n
    return n
```


# ユニットテスト

Rustではテスト機構が言語に組み込まれていて、コードに同梱する形で書くことで実現する。

```rust
#[test]
fn test_gcd() {
    assert_eq!(gcd(14, 15), 1);
    assert_eq!(gcd(2 * 3 * 5 * 11 * 17, 3 * 7 * 11 * 13 * 19), 3 * 11);
    assert_ne!(gcd(10,5,4))
}
```

これを先ほどのプログラムと同じファイルの末尾に書けば良い。  
`Cargo test`を実行すれば以下のような結果が返ってくる。

```bash
cargo test
    Finished test [unoptimized + debuginfo] target(s) in 0.04s
     Running unittests (target/debug/deps/rust-67ad38ae0a14e5b1)

running 1 test
test test_gcd ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

pythonでやるなら組み込みのunittestを使うか、もしくはpytestを使うか。  
自分はpytestを基本的に使うので、pytestでテストを書いてみる。


```python
from python.main import gcd
import pytest


@pytest.mark.parametrize(
    "x, y, answer", [
        (10, 5, 5),
        (36, 4, 4)
    ]
)
def test_gcd(x, y, answer):
    assert gcd(x, y) == answer, f"gcd(10,5) should be {answer}"

@pytest.mark.parametrize(
    "x, y, answer", [
        (10, 5, 4),
        (36, 4, 9)
    ]
)
def test_gcd_error(x, y, answer):
    with pytest.raises(AssertionError):
        assert gcd(x, y) == answer, f"gcd(10,5) should be {answer}"

```

# コマンドライン引数
コマンドライン引数をプログラムに与える方法について、下記のプログラムを実装してみる。

```rust
use std::str::FromStr;
use std::env;

fn main(){
    let mut numbers = Vec::new();
    for arg in env::args().skip(1) {
        numbers.push(u64::from_str(&arg)
            .expect("error parsing argument"));
    }
    if numbers.len() == 0 {
        println!("Usage : gcd NUMBER ...");
        std::process::exit(1);
    }

    let mut d = numbers[0];
    for m in &numbers[1..] {
        d = gcd(d, *m);
    }
    println!("The greatest common divisor of {:?} is {}", numbers, d);
}

```
main.rsをこう書き換える。  
use宣言で標準ライブラリのトレイト`FromStr`をスコープに取り込む。トレイトとは型が実装できるメソッドの集合のことを指す。FromStrトレイトを実装した型は全て`from_str`というメソッドを持つことになり、これは文字列を解析してその型の値に変換しようとするメソッドである。`FromStr`自体はこのプログラムには出てこないが、このトレイトのメソッドである`from_str`を使うために必要である。  
2つ目のuse宣言ではstd::envモジュールを取り込んでいる。このプログラムではenvモジュールが持つargs関数を用いてコマンドライン引数を取得する。

```rust
let mut numbers = Vec::new();
```
で可変なローカル変数`number`を宣言し、空のベクタで初期化している。Vecはサイズ可変のベクタ型で、pythonでいうlistに相当する。ただし、mutを付けなければ動的に変化させることはできない。
実際にはnumbersの型は`u64`つまり`Vec<u64>`だが、後段で`u64`をpushしているのでそこから型推論できるので、ここでは明示的に書く必要はない。  

```rust
for arg in env::args().skip(1) {
```
ここでコマンドライン引数を処理している。`env::args`はイテレータを返すので、forループで一つ一つの引数を処理することができる。イテレータのskipメソッドは与えられた数値までのインデックスを省いたイテレータを生成する。従って、ここでは1つ目の値を省いたイテレータを生成している。

```rust
numbers.push(u64::from_str(&arg)
            .expect("error parsing argument"));
```
ここで`u64::from_str`を呼び出すことで、argに格納されたコマンドライン引数を64ビット整数にパースしている。`from_str`が返すものはu64そのものではなく、パースが成功したかどうかを示す`Result`値である。`Result`値は次の2つの値を取る。

- Ok(v):パースが成功したことを示し、vは生成した値。
- Err(e):パースが失敗したことを示し、eはその理由を説明するエラー値。

失敗する可能性のあることを行う関数は全て`Result`型を返すようになっている。Rustは例外処理を持たないため、エラーはResultまたはパニックで処理される。

パースが成功したかどうかのチェックにはResultのexpectメソッドを用いる。Err(e)ならばexpectはeの説明をするメッセージを出力し、プログラムの実行を中止する。Ok(v)ならばvを返す。

```rust
if numbers.len() == 0 {
        println!("Usage : gcd NUMBER ...");
        std::process::exit(1);
    }
```
ベクタには最低1つの要素が必要なので、そのチェックを行っている。eprint!マクロで標準エラー出力にエラーメッセージを書き出す。

```rust
let mut d = numbers[0];
    for m in &numbers[1..] {
        d = gcd(d, *m);
    }
```
ここで注目すべきは&numbersの&演算子と、*mの\*演算子である。  
これまでのコードでは整数のようなメモリ上の固定超ブロックに収まるような単純な値しか操作してこなかったが、ベクタのサイズは任意であり、とても大きなサイズになる可能性がある。  
このような値を操作するには、ベクタの所有権はnumbersが持っているが、ループではその要素を借用しているだけだということをRustに教える必要がある。`&numbers[1..]` ではベクタの2番目以降の要素の参照を借用していることを示す。*mはmを参照解決する演算子で、参照される値を返す。

```rust
println!("The greatest common divisor of {:?} is {}", numbers, d);
```

最後に、結果を標準出力に書き出す。

実際に実行してみると、

```bash
cargo run 46 23
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/rust 46 23`
The greatest common divisor of [46, 23] is 23
```

上で借用の話が出たので、試しに借用せずにベクタを操作してみる。  
つまり`&numbers[1..]`の&を消して実行してみる。

```bash
cargo run 46 23
   Compiling rust v0.1.0 (/Users/ippei/workspace/programming_rust/chapter2/2.4/rust)
error[E0277]: the size for values of type `[u64]` cannot be known at compilation time
  --> src/main.rs:16:14
   |
16 |     for m in numbers[1..] {
   |              ^^^^^^^^^^^^ expected an implementor of trait `IntoIterator`
   |
   = note: the trait bound `[u64]: IntoIterator` is not satisfied
   = note: required because of the requirements on the impl of `IntoIterator` for `[u64]`
help: consider borrowing here
   |
16 |     for m in &numbers[1..] {
   |              +
16 |     for m in &mut numbers[1..] {
   |              ++++

error[E0277]: `[u64]` is not an iterator
  --> src/main.rs:16:14
   |
16 |     for m in numbers[1..] {
   |              ^^^^^^^^^^^^ expected an implementor of trait `IntoIterator`
   |
   = note: the trait bound `[u64]: IntoIterator` is not satisfied
   = note: required because of the requirements on the impl of `IntoIterator` for `[u64]`
help: consider borrowing here
   |
16 |     for m in &numbers[1..] {
   |              +
16 |     for m in &mut numbers[1..] {
   |              ++++

For more information about this error, try `rustc --explain E0277`.
error: could not compile `rust` due to 2 previous errors
```

結果はこうなる。確かに、ベクターが不定長であることから、コンパイル時間をがわからないという旨のエラーが発生した。合わせて、`[u64] is not an iterator`というエラーが発生している。  
u64がイテレータではないというのはその通りだと思うが、借用しなかった場合にはVecがu64として参照されるのだろうか。4章以降で借用については説明があるようなので、後に譲る。
また、もう一つ面白いのは、&をつけて借用することも考えたら？というコンパイラからのヘルプが出ているところである。これはコードを書く側からすると非常にありがたい。
続いて、\*mの*を消して、参照解決せずに実行してみる。

```bash
cargo run 46 23
   Compiling rust v0.1.0 (/Users/ippei/workspace/programming_rust/chapter2/2.4/rust)
error[E0308]: mismatched types
  --> src/main.rs:17:20
   |
17 |         d = gcd(d, m);
   |                    ^ expected `u64`, found `&u64`
   |
help: consider dereferencing the borrow
   |
17 |         d = gcd(d, *m);
   |                    +

For more information about this error, try `rustc --explain E0308`.
error: could not compile `rust` due to previous error
```
この場合は参照解決していないので参照自体が渡されてしまい、`gcd`関数に渡す値の型としては不適としてエラーを吐いている。  
ここでも参照解決するようなヘルプが出ている。

ここまでをpythonで再現すると以下の通り。

```python
import sys

def gcd(n: int, m: int) -> int:
    assert (n != 0) & (m != 0), "n and m must be non-zero"
    while m != 0: 
        if m < n :
            t = m
            m = n
            n = t
        m = m % n
    return n

def main():
    numbers = sys.argv
    if len(numbers) == 1:
        raise Exception("No arguments")
    d = numbers[1]
    for m in numbers[2:]:
        d = gcd(int(d), int(m))
    print(f"The greatest common divisor of {numbers[1:]} is {d}")

if __name__ == '__main__':
    main()
```
個人的にはpythonでコマンドライン引数を与えるようなスクリプトを作るなら[typer](https://typer.tiangolo.com/)か[fire](https://github.com/google/python-fire)などを使うだろう。

