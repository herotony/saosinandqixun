### 公司服务器
* 在mdpaygate-常规环境中搭建自己的vim练习环境，后续转到公司笔记本，最后，迁移到家里的mac pro上。九月完成。
#### 添加普通账户
* 以root登录后，用useradd wangdalin添加普通账户wangdalin
* 然后用passwd wangdalin为普通账户wangdalin设置密码
* 用指令visudo打开/etc/sudoers文件，搜索找到"root ALL=(ALL) ALL"着一行
    * 利用yyp指令，直接拷贝这一行并在其下生成新的一行并粘贴此行。
    * 选择新行上的<font color=Teal>root</font>，利用<font color=Teal>cw</font>来清除root并输入wangdain后，:wq保存即可，此时我们就能在wangdalin账号下利用sudo 执行相关指令了。
    * 若要执行sudo时无需输入密码，请在上述的新行中将**最后**的ALL，替换为**NOPASSWD:ALL**即可。
#### 删除普通用户
* userdel -r wangdalin,参数 -r 代表同时删除该用户的home目录。
* 目测tmux不会因为用户删除而删除，其会一直存在...
### vim插件安装
#### 入口插件：插件管理器vundle
* [网文介绍](http://www.jianshu.com/p/8d416ac4ad11)
* 该插件最大的优点就是整合了git，使得插件的管理，更新，卸载极大得到简化。
* bundle分为三类：
    * 在Github vim-scripts 用户下的repos,只需要写出repos名称
    * 在Github其他用户下的repos, 需要写出”用户名/repos名”
    * 不在Github上的插件，需要写出git全路径
* 如何安装Bundle指定的插件
    * 在完成vimrc的编辑后，随便打开一个vim，运行":PluginInstall"
        * 相应地：在vim里输入 ":PluginUpdate" 完成插件更新
        * ":PluginClean"清除不再使用的插件，不知如何判断。
        * ":PluginSearch"查找插件
        * ":PluginList"列举所有插件
    * 或者命令行模式下输入"vim +BundleInstall +qall"
##### 下载及安装
    * 下载： git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/vundle.vim
    * 安装： 新建编辑 ~/.vimrc文件，输入如下内容，至此具备了初级模板，需要什么插件往里填写即可。
```
set nocompatible              " be iMproved, required
filetype off                  " required

" 启用vundle来管理vim插件
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" 安装插件写在这之后

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" sample for nerdtree
Plugin 'scrooloose/nerdtree'

" 安装插件写在这之前
call vundle#end()            " required
filetype plugin on    " required

" 常用命令
" :PluginList       - 查看已经安装的插件
" :PluginInstall    - 安装插件
" :PluginUpdate     - 更新插件
" :PluginSearch     - 搜索插件，例如 :PluginSearch xml就能搜到xml相关的插件
" :PluginClean      - 删除插件，把安装插件对应行删除，然后执行这个命令即可

" h: vundle         - 获取帮助

" vundle的配置到此结束，下面是你自己的配置

```
#### 目录插件：nerdtree
### vim一般配置
