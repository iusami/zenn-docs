---
title: "Reverse string(by Exercism) ~C++ãªãããª~"
emoji: "ð»"
type: "tech" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: ["cpp"]
published: true
---

# æ¦è¦
C++ãªãããªè¨äºã  
Exercismã®[Reverse string](https://exercism.org/tracks/cpp/exercises/reverse-string)è§£æ³ã


# åé¡
å¥åãããæå­åãéé ã«ãã¦è¿ãé¢æ°ãä½æããã

# è§£æ³
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

# ã¡ã¢
## stringã«ã¤ãã¦
`#include <string>`ã§ä½¿ç¨ã§ããæå­åã¯ã©ã¹ã ååç©ºéã¯stdã  

### ç¹å®ã®ã¤ã³ããã¯ã¹ã®æå­ã®æ½åº
`str[0]`ãªã©ã¤ã³ããã¯ã¹ãç´æ¥æå®ããã


### æå­åã®çµå
+ã§çµåã§ããã

### æå­åã®é·ãã®åå¾
size()ã§åå¾å¯è½