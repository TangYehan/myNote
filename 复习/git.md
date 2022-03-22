参考资料

+ [你真的懂git rebase吗？](https://www.jianshu.com/p/6960811ac89c)
+ 

### 多次本地commit合并为一次

+ 首先通过`git log`查看一下信息

+ `git rebase -i HEAD~X`

  将以head为起点的三个commit 合并

  将进入编辑界面

  ![](https://upload-images.jianshu.io/upload_images/2147642-03d48aa767efb307.png?imageMogr2/auto-orient/strip|imageView2/2/w/647/format/webp)

> pick：保留该commit（缩写:p）
>
> reword：保留该commit，但我需要修改该commit的注释（缩写:r）
>
> edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
>
> squash：将该commit和前一个commit合并（缩写:s）
>
> fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
>
> exec：执行shell命令（缩写:x）
>
> drop：我要丢弃该commit（缩写:d）

+ 一般来说，将后面几个pick改为s

+ 然后ctrl + c回到命令输入，再按`:wq`退出编辑

+ 此时会弹出commit编写界面

  ![](https://upload-images.jianshu.io/upload_images/2147642-44bbd784dcadfb31.png?imageMogr2/auto-orient/strip|imageView2/2/w/801/format/webp)

编辑完保存即可完成commit合并



+ 如果合并了不该合并的，通过`git reflog`查看本地记录
+ 通过`git reset --hard 02a3260`来回退到指定版本



### 新建分支

+ git checkout -b dev
+ git push origin dev（把本地dev分支提交到origin dev分支）
+ git branch 查看分支情况

