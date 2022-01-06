---
date: 2020-06-27 
title: 不使用原生的Number和parseInt实现StringToNumber
tags:
  - 2020
  - js进阶
describe: 巧用Array.reduce实现string to number
---

```js
let StringToNumber = (str)=>{
    let arr = str.trim().split('')
    let sign = arr[0] === '-' ? -1 : 1
    if (sign === -1 || str[0] === '+') {
        arr.shift()
    }
    return sign * arr.reduce((total, cur)=>(
        total * 10 + (cur >= '0' && cur <= '9' ? (cur - '0') : NaN)
    ))
}
```