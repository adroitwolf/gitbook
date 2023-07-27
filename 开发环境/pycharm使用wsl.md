# pycharm使用wsl进行开发

* windows版本的pytorch就是狗屎！
* windows版本的pytorch就是狗屎！
* windows版本的pytorch就是狗屎！


> 因为windows版的python在进行代码开发的时候不出意外的会出现各种意外，所以干脆我们使用wsl进行开发

## 但是随之而来的会出现两个问题
1. c盘容量实在支持不住linux+ conda这两个大容量的家伙
解决办法：[wsl迁移盘符](https://www.bilibili.com/video/BV1Fx4y1j7yy)
2. wsl在pycharm中的适配确实不如vscode香
> 但是pycharm的debug模式太香了
    * pycharm连接到wsl后可以运行，但是无法debug
        解决办法：[解决wsl无法debug](https://blog.csdn.net/qq_38992249/article/details/122387097)