### 公司服务器
* 在mdpaygate-常规环境(10.9.28.109)中搭建自己的vim练习环境，后续转到公司笔记本，最后，迁移到家里的mac pro上。九月完成。
* vundle + github管理vim的云配置，将会实现极其容易打理自己的基于vim的开发环境。即简单同步更新相应的.vimrc文件就能通过vundle插件更新所有的插件，从而立即可用vim进行相应的开发。本次将先搭建基于python的开发环境，后续追加c/c++的。逐步记录每个插件的安装，配置和使用的记录。
    * [参考网文：如何建立vim云配置](http://jintongyao.github.io/2014/how-to-use-vundle/)
* 在windows下，建议使用gitbash.exe来模拟linux命令行环境，cd ~ -> ls -a -> install vundle...算啦，各种不正常，windows环境下不瞎折腾了.
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
* [经典介绍](http://www.zlovezl.cn/articles/vim-plugins-cannot-live-without/)
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
    * 或者命令行模式下输入"vim +PluginInstall +qall"
##### 下载及安装
    * 下载： git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
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
##### 安装
* 非常简单，在上述的.vimrc文件中添加<font color=Teal>Plugin 'scrooloose/nerdtree'</font>,直接在vim中运行PluginUpdate即可。
##### 如何使用
* 一种很简单，打开vim，命令行模式输入**:NERDTree 回车即可**
* 另外一种，更常用，在vimrc中配置启动和相应的快捷键映射进行开关。说明如下：
    * 在vimrc中，添加**map <F3> :NERDTreeMirror<CR>** 和 **map <F3> :NERDTreeToggle<CR>**即可实现在vim中随时用F3键打开和关闭NERDTree目录。
##### 使用指南
* 进入当前目录的树形界面，通过小键盘"上下"键，能移动选中的目录或文件。目录前面有"+"号，按Enter会展开目录，文件前面是"-"号，按Enter会在右侧窗口展现该文件的内容，并光标的焦点focus右侧。"ctr+w+h"光标focus左侧树形目录，"ctrl+w+l"光标focus右侧文件显示窗口。多次按"ctrl+w+w"，光标自动在左右侧窗口切换。光标focus左侧树形窗口，按"？"弹出NERDTree的帮助，再次按"？"关闭帮助显示。输入":q"回车，关闭光标所在窗口。

NERDTree提供了丰富的键盘操作方式来浏览和打开文件，介绍一些常用的快捷键：

    * 和编辑文件一样，通过h j k l移动光标定位
    * 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
    * go 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
    * i和s可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
    * t 在标签页中打开
    * T 在后台标签页中打开
    * p 到上层目录
    * P 到根目录
    * K 到同目录第一个节点
    * J 到同目录最后一个节点
    * m 显示文件系统菜单（添加、删除、移动操作）
    * ? 帮助
    * q 关闭

* 标签页可用**gt**直接切换，或者**{N}gt**指定要跳转的标签页，**gT**则反方向跳转
* **gi/go/gs/**则光标还停在nerdtree目录内。
* Bookmark在很多层级目录，很多文件时特别有用，输入:Book..tab键会显示相关完整Bookmark指令。大写B会显示书签，再按B则关闭书签显示目录。
#### 搜索插件ctrlp
* ctrl+p键的组合简称？一款极其高效的搜索插件，对于多大数百文件的工程项目带来极大的效率上的提升。
* 只能根据文件名进行搜索，但能在很多级目录中迅速搜索，很快！
##### 安装
* 同样非常简单，在上述的.vimrc文件中添加<font color=Teal>Plugin 'kien/ctrlp.vim'</font>,直接在vim中(最好重启一个新的vim)运行PluginUpdate即可。
##### 配置
* 需要在.vimrc文件中添加如下配置，其中'c-p'键冲突了，用'c-f'来启动即可
```
let g:ctrlp_map = '<c-f>'
let g:ctrlp_cmd = 'CtrlP'
let g:ctrlp_match_window = 'bottom,order:btt,min:1,max:10,results:20'
let g:NERDTreeChDirMode = 2
let g:ctrlp_working_path_mode = 'rw'
```
##### 使用指南
* [CtrlP插件配置参数说明](http://blog.codepiano.com/pages/ctrlp-cn.light.html#ctrlp_working_path_mode)
* 在<c-v>,<c-x>,<c-t>分别对应将搜素结果垂直，水平分割显示或新标签页显示选中文件
* 而<c-k>,<c-j>是在搜索结果中上下浏览选中搜索结果。
### vim一般配置
