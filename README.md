# KimZhangJinMing.github.io
### 迁移到新电脑

1.git clone hexo分支

2.全局安装hexo 

```js
npm install hexo-cli -g
```

3.在主目录下执行：

```txt
npm install
hexo g
hexo d
```



### 提交主题文件到github

先删除主题文件夹下的.git文件

```txt
git rm --cache theme/hexo-theme-next
git add theme/hexo-theme-next
git commit -m ""
git push
```

