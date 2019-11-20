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