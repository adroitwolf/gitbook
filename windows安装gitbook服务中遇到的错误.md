# windows安装gitbook服务中遇到的错误

最近因为不太喜欢没有目录的Hexo网站了，又开始重新弄gitbook。这里记录一下安装过程中的错误。

## yarn安装gitbook服务

使用命令
```bash
    yarn global add gitbook-cli
```

提示node版本太低，这里我们使用nvm替换node版本
```bash
    nvm install 12
    nvm use 12.0.0
```
切换之后虽然说安装完毕，但是使用gitbook时候总是提示出现Check failed:U_sucess的错误代码。根据csdn的说法是node版本不能使用12的问题，这里我尝试退回到我的10.v版本


但是还是报错，提示错误：TypeError: Cannot read property 'commands' of null，经过排查发现是gitbook版本的问题，这里直接回退到gitbook3.0.0版本