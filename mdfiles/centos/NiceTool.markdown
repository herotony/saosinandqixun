### tmux

* 该工具最大的好处就是在服务器端保持session，而不用担心断开终端连接相应的工作就被迫中断，这是其最大价值。
    * 比如，我连接到QueueConsumeService定义一个经常用来查询日志的session，该session同时打开查询business,default,error以及查看cpu使用率的窗口，那么建立好tmux的session后，以后都能随时进入该session而无需每次各种cd,切换目录等麻烦了，当然更大的价值在于持续长时间的项目编译或者监控等事宜。
    * 在centos6.8以后版本直接**yum install tmux**/mac则直接**brew install tmux**即可完成安装。
        * 安装好后，tmux new -s your-job-name,建立your-job-name的新session
        * tmux ls,列举当前所有有效的tmux session
        * tmux a -t your-job-name,重新建立用户与tmux session的联系
        * 所有涉及tmux本身的操作，都是先点击ctrl+b来触发，随后再按相应功能键来完成相应功能，清单如下：
            * c ： 创建新窗口
            * ，： 重命名当前窗口
            * o :  切换当前同一窗口内的pane
            * p :  切换当前同一session下的窗口
            * d :  **关键好处哦，断开当前用户和tmux session间的联系，但tmux sesssion中的任何前台任务还是有效维持着**
            * % ： 垂直划分当前窗口中的当前pane
            * " :  水平划分当前窗口中的当前pane
            * x :  关闭当前pane

暂时用到的就这些，其他好处待以后有体会再补充。
