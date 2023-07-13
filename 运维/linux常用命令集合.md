### inux下使用tar命令
_____
#### 主选项：
    c 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。相当于打包。
    x 从档案文件中释放文件。相当于拆包。
    t 列出档案文件的内容，查看已经备份了哪些文件。
特别注意，在参数的下达中， c/x/t 仅能存在一个！不可同时存在！因为不可能同时压缩与解压缩。

#### 辅助选项：
    -z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩或解压？ 一般格式为xx.tar.gz或xx. tgz
    -j ：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩或解压？一般格式为xx.tar.bz2  
    -v ：压缩的过程中显示文件！这个常用
    -f ：使用档名，请留意，在 f 之后要立即接档名喔！不要再加其他参数
    -p ：使用原文件的原来属性（属性不会依据使用者而变）
### 范例：
    压缩
    tar –cvf 打包后的文件名称 打包源文件
    解压
    tar –xvf 名称 –C 指定目录
### linux下使用scp命令
___
    scp是有Security的文件copy，基于ssh登录。 
    命令基本格式： 
    scp [OPTIONS] file_source file_target 
    OPTIONS： 
    -v 和大多数 linux 命令中的 -v 意思一样 , 用来显示进度 . 可以用来查看连接、认证、 或是配置错误 
    -C 使能压缩选项 
    -P 选择端口 . 注意 -p 已经被 rcp 使用 
    从本地复制到远程 
    scp /home/daisy/full.tar.gz root@172.19.2.75:/home/root      （然后会提示你输入另外那台172.19.2.75主机的root用户的登录密码，接着就开始copy了），复制目录加参数 -r 即可 
    从远程复制到本地 
    scp root@/172.19.2.75:/home/root/full.tar.gz /home/daisy/full.tar.gz
### linux下文件权限
___
    dxxx xxx xxx 第一位文件夹or文件 2-4 文件拥有者权限 5-7 用户组 8-9 其他组 
    rwx 4 2 1  -- 7 = 4+2+1 
    ex. 655 777
    小记：
### linux 基础命令
    grep、find、ps -aux、losf (打开句柄。linux 资源 文件 端口等都分配了一个句柄)、fdisk、mount、umount、df -h 、top 、curl 、scp、awk 、