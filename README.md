# abgelehnt.github.io

# 运行流程

## 1. 初始化git仓库

```bash
git submodule update
```

## 2. 下载hugo

```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.147.9/hugo_0.147.9_linux-amd64.tar.gz
tar -xf hugo_0.147.9_linux-amd64.tar.gz hugo
```

## 2. 启动 Hugo 的开发服务器以查看网站

1. 在本地查看网站

``` bash
./hugo -D server
```

2. 在局域网查看网站

```bash
./hugo server -D --bind 192.168.2.3 -p 1313 --baseURL=http://192.168.2.3:1313
```