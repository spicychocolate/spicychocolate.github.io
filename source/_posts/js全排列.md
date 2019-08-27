---
title: js全排列
date: 2019-08-15 14:30:03
tags:
    - LeetCode
    - js全排列
---

```javascript
function permutations(string) {
    if (string.length == 1) {
        return [string]
    } else {
        const array = string.split('') 
         return array.map((e, i) => {
             // 取出当前元素之外的其他元素
            let newString = string.slice(0,i) + string.slice(i+1)
            // 其他元素进行全排列
            return permutations(newString).map((e2) => e+e2)
        })
        .reduce((r,e) => r.concat(e), [])
        .sort()
        .filter((e,i,a) => (i==0) || a[i-1] != e)
    }
}
permutations('aabb')
```
<!-- more -->