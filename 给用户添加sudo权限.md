 # Linux给用户添加sudo权限 
 
 
    分类： LINUX
    linux给用户添加sudo权限： 
    有时候，linux下面运行sudo命令，会提示类似： 
    xxxis not in the sudoers file.  This incident will be reported. 
    这里，xxx是用户名称，然后导致无法执行sudo命令，这时候，如下解决：
    进入超级用户模式。也就是输入"su -",系统会让你输入超级用户密码，输入密码后就进入了超级用户模式。（当然，你也可以直接用root用）
    添加文件的写权限。也就是输入命令"chmod u+w /etc/sudoers"。 
    编辑/etc/sudoers文件。也就是输入命令"vim /etc/sudoers",进入编辑模式，找到这一 行："root ALL=(ALL) ALL"在起下面添加"xxx ALL=(ALL) ALL"(这里的xxx是你的用户名)，然后保存退出。
    撤销文件的写权限。也就是输入命令"chmod u-w /etc/sudoers"。 
    然后就行了。
