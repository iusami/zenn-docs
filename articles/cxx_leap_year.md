---
title: "Leap (by exercism) ~C++ãªãããª~"
emoji: "ð¥"
type: "tech" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: ["cpp"]
published: true
---


# æ¦è¦
C++ãªãããªè¨äºã  
Exercismã®[Leap](https://exercism.org/tracks/cpp/exercises/leap)è§£æ³ã  

# åé¡
intã®å¥åã«å¯¾ãã¦ããããéå¹´ãã©ãããå¤å®ããã³ã¼ããæ¸ãã

# ãã¹ã

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

# è§£æ³1

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

ããã§ãéãããããããã¡ã¤ã«ã«ã¯å®£è¨ãæ¸ãã¦ãé¢æ°ã®å®ç¾©ã¨ãã¯ã½ã¼ã¹ã³ã¼ãã«æ¸ããã®ãããã  
ãã¾ãã«é·ãé¢æ°å®ç¾©ãããããã¡ã¤ã«ã«æ¸ãã¨å¾ã«ã³ã³ãã¤ã«æéãä¼¸ã³ãããã ããã ([åè](https://qiita.com/agate-pris/items/1b29726935f0b6e75768))  
ãªã®ã§æ¸ãç´ãã

# è§£æ³2

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
ãããã«é¢æ°ã®ã¿å®£è¨ãã¦ããã¦ãå®ç¾©ã¯ã½ã¼ã¹ã«æ¸ããã

# ã¡ã¢

## ã#if !defined(LEAP_H)ã is ä½
#ifã§ç¹å®ã®ã·ã³ãã«ãå®ç¾©ããã¦ãããã©ãããèª¿ã¹ãã  
#endifã§æ¬ãå¿è¦ãããã

