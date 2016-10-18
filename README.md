# This Repository is for Anicloud Github Pages
- author zhangzhaoyu@anicloud.com
- date 2016-10-18

# Introduction
Anicloud Github Page 是采用Hexo 静态博客工具生成，支持markdown 语法，站内搜索，博客归档等功能。

# How To
- __安装 Node__  hexo 的运行要依赖于Node 环境，Node 的安装参见[Node光网](https://nodejs.org/en/)。
- __安装 hexo__ hexo 是静态博客的生成程序，[安装过程参见](https://hexo.io/zh-cn/)。
```
npm install -g hexo-cli
```

- __Clone 源文件__ 
```
git clone https://github.com/anicloud/anicloud.github.io.source.git
```

- __新建博客__ 新建的博客再source/_posts 目录中，新建之后可以用任何文本编辑器编辑，推荐Emacs。
```
hexo new hello-worl
```
- __生成发布博客__
```
hexo clean
hexo generate
hexo deploy
```

- __提交源文件__
```
git add -A
git commmit -a -m 'message'
git push
```

# Enjoy it !!!
