---
dg-publish: false
---

---
---
dg-publish: true
---


当一个文件已经被加入到git的管理(也就是已经提交到git仓库了)  这时候我们再在 `.gitignore` 文件中 添加此文件 是不奏效的 需要 
```
git rm -rf --cached .
git add .
```

然后重新提交 之后就可以了