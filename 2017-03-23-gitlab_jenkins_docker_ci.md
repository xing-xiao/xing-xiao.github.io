# gitlab + jenkins + kubernetes自动化集成

## docker

jenkins运行docker命令需要将jenkins加入docker用户组，

`gpasswd -a jenkins root`

`gpasswd -a jenkins docker`

记得要把jenkins重启


jenkins运行sudo命令，在`/etc/sudoers`文件中加入如下内容

```
## Same thing without a password
%wheel  ALL=(ALL)       NOPASSWD: ALL
jenkins ALL=(ALL)       NOPASSWD: ALL
```