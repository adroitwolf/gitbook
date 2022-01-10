# ubuntu安装oh-my-zsh配置强大终端

普通的终端总是不那么智能，所以今天安装一下zsh

## 下载oh-my-zsh

```bash
    git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
    cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```
> 注意这里可能因为版本问题导致命令不一样，建议直接在Github上面搜索oh-my-zsh然后根据文档安装
## 下载zsh

    ```bash
        apt install zsh -y    
        chsh -s /bin/zsh # 切换默认终端
    ```

    完成后重启即可，之后会变成oh-my-zsh的主题形式
## 安装插件
    这里推荐两个比较实用的插件
    * zsh-syntax-highlighting：高亮你的zsh命令，正确显示绿色，错误显示红色。
    ```bash
        git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    ```
    * zsh-autosuggestions:自动匹配你历史的输入命令。
    ```bash
        git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
    ```

    下载完成后需要打开.zshrc文件并在其中加入 plugins=(... zsh-syntax-highlighting zsh-autosuggestions)
    保存退出，更新一下文件source ./zshrc，之后重新打开终端即可
## 设置命令别名
    1. 还是找到zsh的配置文件.zshrc，（～/.zshrc），添加命令别名：

    > 例如：alias install="sudo apt-get install"
    2. 应用配置文件，使配置生效 source ～/.zshrc
    3. 查看当前shell现有别名，终端下输入： ~ alias 
    Bash 里一些常用的别名：
    ```bash
        alias la='ls -Fa'          # 列出所有文件
        alias ll='ls -Fls'         # 列出文件详细信息
        alias rm='rm -i'           # 删除前需确认
        alias cp='cp -i'           # 覆盖前需确认
        alias mv='mv -i'           # 覆盖前需确认
        alias vi='vim'             # 输入 vi 命令时使用 vim 编辑器
    ```

## 特别鸣谢
   *  [minifullc的博客](https://minifullc.github.io/2018/03/14/oh-my-zsh-配置zsh-终端环境)
