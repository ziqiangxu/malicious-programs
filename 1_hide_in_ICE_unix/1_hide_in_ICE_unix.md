# Hide in ICE unix

最近在服务器上发现了这个恶意程序，它运行的时候， `CPU` 飙高，进程名伪装成 `bash` 或者 `systemd`，所以它有可能会伪装成其它的常用程序名，各位要擦亮眼睛了。

最初发现它的时候隐藏在 `/tmp/.ICE_unix` 目录下，它的路径为:
-  `/tmp/.ICE_unix/.init/`
- 或者 `/tmp/.ICE_unix/.bash`
- 或者其它的隐藏目录

## 如何查看这个进程的信息，找到它隐藏的地方

参考文章：[https://blog.csdn.net/jsh13417/article/details/11778675?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase](https://blog.csdn.net/jsh13417/article/details/11778675?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.nonecase)

``` bash
gdb # 进入gdb调试命令行
attach {pid}  # pid是进程号
info proc  # 查看进程信息，就是在这里发现 .ICE_unix目录的
```

## 行为研究

> 这个程序被放到服务器并运行了起来，说明你的服务器的可能被入侵了，还好这个程序是一个温和的程序，目前发现它仅仅是占用你的计算资源。所以请做好相关的调查，并加强防护。

- 该程序启动后会在当前用户创建 `crontab` 定时任务，用于定时启动它自己，所以杀掉这个进程之后它很快又自己开起来
> （可以在 `/var/spool/cron/` 目录查看所有用户的定时任务）
- 该恶意程序启动后使得 `CPU` 飙高，观察发现，如果在无网络的情况下启动它，它不会占用大量计算资源，一旦可以访问公网，它就开始占用 `CPU`，所以判断它可能是一个挖矿程序

> 
``` sh
#!/bin/bash
cd -- /tmp/.ICE-unix/.bash 
mkdir -- .bash  # 创建一个隐藏目录
cp -f -- x86_64 .bash/bash  # 生成自己的副本
./.bash/bash  -c  # 运行副本
rm -rf .bash  # 移除副本，增加找到它的难度
```

## 应对策略

1. 先关闭计划任务
2. 再杀掉这个进程
3. 做好防入侵工作，可参考[https://www.cnblogs.com/feixiablog/articles/9186848.html](https://www.cnblogs.com/feixiablog/articles/9186848.html)

这次事件的处理，虽然探究这个恶意程序是做了什么是必要的，但是也许更重要的是找到它被放进来的原因。

## 下载这个可恶的恶意程序

[https://github.com/ziqiangxu/rogue-programes/releases/tag/1](https://github.com/ziqiangxu/rogue-programes/releases/tag/1)
> 运行这个程序请务必做好安全措施，防止意外发生
