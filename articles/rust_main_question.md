---
title: "rustのmainで?を使う"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
---

csvをreadする練習をしたくてコードを書いたが、[公式のexample](https://docs.rs/csv/1.0.0/csv/struct.Reader.html#method.from_path)通りに書くと、main関数内でエラーを吐く。
```rust
fn main(){
    let mut rdr = Reader::from_path("../data/test_data.csv")?;
    for result in rdr.records() {
        let record = result?;
        println!("{:?}", record);
    }
}
```
エラー内容としては
``` this function should return `Result` or `Option` to accept `?` ```
らしいので戻り値がResultもしくはOption型でないと?は使っちゃいけない。

なのでこう書く

```rust
fn main()-> Result<(), Box<Error>>{
    let mut rdr = Reader::from_path("../data/test_data.csv")?;
    for result in rdr.records() {
        let record = result?;
        println!("{:?}", record);
    }
    Ok(())
}
```

これで動くが、まだ?をここまでして使う旨みがわかってない。  
エラー処理が簡略化できるとかなんとか。

# 参考文献
https://qiita.com/termoshtt/items/d7e3ad2bb0b51fa9b58c