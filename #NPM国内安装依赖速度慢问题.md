# NPM国内安装依赖速度慢问题

不知道各位是否遇到这种情况，使用NPM（[Node.js](http://lib.csdn.net/base/nodejs)包管理工具）安装依赖时速度特别慢，为了安装Express，执行命令后两个多小时都没安装成功，最后只能取消安装，笔者20M带宽，应该不是我网络的原因，后来在网上找了好久才找到一种最佳解决办法，在安装时可以手动指定从哪个镜像服务器获取资源，我们可以使用阿里巴巴在国内的镜像服务器，命令如下：

```bash
npm install -gd express --registry=http://registry.npm.taobao.org11
```

只需要使用–registry参数指定镜像服务器地址，为了避免每次安装都需要`--registry`参数，可以使用如下命令进行永久设置：

```bash
npm config set registry http://registry.npm.taobao.org11
```

换了国内镜像，安装速度就很快了，特写篇博客分享这种方法，希望对大家有帮助