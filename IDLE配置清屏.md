[IDLE配置清屏](https://blog.csdn.net/weixin_35674012/article/details/78195834)
-------------
在Python IDLE中的清屏快捷键默认是没有的，需要进行一些操作 

一、将下面的代码复制保存到新建的ClearWindow.py （这个文件放在Python X\Lib\idlelib目录下）中
```python
"""

Clear Window Extension
Version: 0.2

Author: Roger D. Serwy
        roger.serwy@gmail.com

Date: 2009-06-14

It provides "Clear Shell Window" under "Options"
with ability to undo.

Add these lines to config-extensions.def

[ClearWindow]
enable=1
enable_editor=0
enable_shell=1
[ClearWindow_cfgBindings]
clear-window=<Control-Key-l>


"""

class ClearWindow:

    menudefs = [
        ('options', [None,
               ('Clear Shell Window', '<<clear-window>>'),
       ]),]

    def __init__(self, editwin):
        self.editwin = editwin
        self.text = self.editwin.text
        self.text.bind("<<clear-window>>", self.clear_window2)

        self.text.bind("<<undo>>", self.undo_event)  # add="+" doesn't work

    def undo_event(self, event):
        text = self.text

        text.mark_set("iomark2", "iomark")
        text.mark_set("insert2", "insert")
        self.editwin.undo.undo_event(event)

        # fix iomark and insert
        text.mark_set("iomark", "iomark2")
        text.mark_set("insert", "insert2")
        text.mark_unset("iomark2")
        text.mark_unset("insert2")


    def clear_window2(self, event): # Alternative method
        # work around the ModifiedUndoDelegator
        text = self.text
        text.undo_block_start()
        text.mark_set("iomark2", "iomark")
        text.mark_set("iomark", 1.0)
        text.delete(1.0, "iomark2 linestart")
        text.mark_set("iomark", "iomark2")
        text.mark_unset("iomark2")
        text.undo_block_stop()
        if self.text.compare('insert', '<', 'iomark'):
            self.text.mark_set('insert', 'end-1c')
        self.editwin.set_line_and_column()

    def clear_window(self, event):
        # remove undo delegator
        undo = self.editwin.undo
        self.editwin.per.removefilter(undo)

        # clear the window, but preserve current command
        self.text.delete(1.0, "iomark linestart")
        if self.text.compare('insert', '<', 'iomark'):
            self.text.mark_set('insert', 'end-1c')
        self.editwin.set_line_and_column()

        # restore undo delegator
        self.editwin.per.insertfilter(undo)
 ```
二:然后在这个目录下找到config-extensions.def这个文件（idle扩展的配置文件），打开config-extensions.def 后在句末加上如下代码
```def
[ClearWindow]
enable=1
enable_editor=0
enable_shell=1
[ClearWindow_cfgBindings]
clear-window=<Control-Key-l>
```
三:打开Python的IDLE，options选项中就可以看到增加了Clear shell window ctrl+;
* 说明:  新建的ClearWindow.py文件名一定要与在config-extensions.def中添加的[ClearWindow]内容中的名字一致，要不然找不到配置文件的，注意大小写！！！
