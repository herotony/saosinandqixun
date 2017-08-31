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
### vim一般配置
