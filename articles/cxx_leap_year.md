---
title: "Leap (exercism)"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp"]
published: true
---


# 概要
C++リハビリ記事。  
Exercismの[Leap](https://exercism.org/tracks/cpp/exercises/leap)解法。  

# 問題
intの入力に対して、それが閏年かどうかを判定するコードを書く。

# テスト

```cpp
#include "leap.h"  
#ifdef EXERCISM_TEST_SUITE  
#include <catch2/catch.hpp>  
#else  
#include "test/catch.hpp"  
#endif  
TEST_CASE("not_divisible_by_4")
{
    REQUIRE(!leap::is_leap_year(2015));
}
#if defined(EXERCISM_RUN_ALL_TESTS)
TEST_CASE("divisible_by_2_not_divisible_by_4")
{
    REQUIRE(!leap::is_leap_year(1970));
}
TEST_CASE("divisible_by_4_not_divisible_by_100")
{
    REQUIRE(leap::is_leap_year(1996));
}
TEST_CASE("divisible_by_100_not_divisible_by_400")
{
    REQUIRE(!leap::is_leap_year(2100));
}
TEST_CASE("divisible_by_400")
{
    REQUIRE(leap::is_leap_year(2000));
}
TEST_CASE("divisible_by_200_not_divisible_by_400")
{
    REQUIRE(!leap::is_leap_year(1800));
}
#endif
```

# 解法1

```cpp:leap.cpp
namespace leap {
    int main(int){
        return 0;
    }
} 
```
```cpp:leap.h
#if !defined(LEAP_H)
#define LEAP_H

namespace leap {
    bool is_leap_year(int year){
        if (year % 400 == 0){
            return true;
        }
        if (year % 100 == 0){
            return false;
        }
        if (year % 4 == 0){
            return true;
        }
        return false;
    }
}  // namespace leap

#endif // LEAP_H
```

これでも通るが、ヘッダファイルには宣言を書いて、関数の定義とかはソースコードに書くものらしい。  
あまりに長い関数定義をヘッダファイルに書くと徒にコンパイル時間が伸びるからだそうだ([参考](https://qiita.com/agate-pris/items/1b29726935f0b6e75768))  
なので書き直す。

# 解法2

```cpp:leap.h
#if !defined(LEAP_H)
#define LEAP_H
namespace leap {
    bool is_leap_year(int year);
} 
#endif
```

```cpp:leap.cpp
#include "leap.h"
namespace leap {
    bool is_leap_year(int year){
        if (year % 400 == 0){
            return true;
        }
        if (year % 100 == 0){
            return false;
        }
        if (year % 4 == 0){
            return true;
        }
        return false;
    }
} 
```
ヘッダに関数のみ宣言しておいて、定義はソースに書いた。

# メモ

## 「#if !defined(LEAP_H)」 is 何
#ifで特定のシンボルが定義されているかどうかを調べる。  
#endifで括る必要がある。

