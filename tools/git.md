# Git

## 代理

设置代理
```bash
git config --global http.proxy 'socks5://127.0.0.1:1088' 
git config --global https.proxy 'socks5://127.0.0.1:1088'
```

取消代理
```bash
git config --global --unset http.proxy  
git config --global --unset https.proxy
```

## 凭证

设置保存凭证
```bash
git config credential.helper store
```
注意，这会将你的 git 明文密码保存在本地磁盘上。

重设凭证
```bash
vim ~/.git-credentials
```
通过编辑凭证文件来更新。


## git rebase 合并 commit 或 变基

使用 git rebase 可以合并多个提交，并优雅地合并。


想要合并1-3条，有两个方法
1. 从HEAD版本开始往过去数3个版本
```bash
git rebase -i HEAD~3
```

2. 指名要合并的版本之前的版本号
```bash
git rebase -i 3a4226b
```
