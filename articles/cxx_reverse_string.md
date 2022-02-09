---
title: "Reverse string(by Exercism) ~C++リハビリ~"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp"]
published: true
---

# 概要
C++リハビリ記事。  
Exercismの[Reverse string](https://exercism.org/tracks/cpp/exercises/reverse-string)解法。


# 問題
入力された文字列を逆順にして返す関数を作成する。

# 解法
```cpp:reverse_string.h
#if !defined(REVERSE_STRING_H)
#define REVERSE_STRING_H
#include <string>
namespace reverse_string {
    std::string reverse_string(std::string);
}  // namespace reverse_string
#endif // REVERSE_STRING_H
```

```cpp:reverse_string.cpp
#include "reverse_string.h"
namespace reverse_string {
    std::string reverse_string(std::string str) {
        int str_length = str.size();
        std::string str_return="";
        for (int i=str_length-1; i>=0; i--){
            str_return = str_return+str[i];
        }
        return str_return;
    }
}  // namespace reverse_string
```

# メモ
## stringについて
`#include <string>`で使用できる文字列クラス。 名前空間はstd。  

### 特定のインデックスの文字の抽出
`str[0]`などインデックスを直接指定する。


### 文字列の結合
+で結合できる。

### 文字列の長さの取得
size()で取得可能