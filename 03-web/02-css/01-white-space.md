---
dg-publish: true
---
## css white-space属性    
```html
<style>
p {
  width: 100px;
  background: red;
}	
</style>
<p>  hellohellohello hello
world
</p>
```
属性|描述
---|---
normal|默认值，空格，换行省略，超出div宽度换行(word-break属性有关，长单词默认不换行，会超出文本框)
nowrap|不换行
pre|按照pre方式
pre-wrap|pre方式，超出宽度换行

## word-break
值	|描述
---|---
normal	|使用浏览器默认的换行规则。
break-all|	允许在单词内换行。
keep-all	|只能在半角空格或连字符处换行。

## 更多
https://developer.mozilla.org/en-US/docs/Web/CSS/white-space