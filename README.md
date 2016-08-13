**目录**
- [简介](#简介)
- [方案](#方案)
- [源码](#源码)
- [参考](#参考)

###简介
关于急速打包
之前有过一篇文章[《Gradle编译打包小结》](http://www.jianshu.com/p/d7279528e661)
笔者平时也是用这种常规方式打包编译的，速度比较慢，尤其是在渠道比较多的的时候
笔者整理了一些网络上的资料，大致有两类方案，作个小结。
###方案
1.使用美团的方案
  美团使用的是python打包，脚本也并不复杂。
  大致思路是，可以将我们的apk包看成是一个zip压缩包，app在初始化运行的时候，读取压缩包里META-INF下的配置文件名区分渠道，然后写入自己的配置文件中，下次直接从配置文件里读取(当然，这里先不考虑不同渠道包覆盖安装的问题哈...)
  所以使用python的脚本，可以直接向我们事先生成好的apk包里的META-INF目录添加配置文件即可，实现难度不大，但是最先想到这种方案的确实高明。
  笔者略改动的脚本
```
#!/usr/bin/python
# coding=utf-8
import zipfile
import shutil
import os

# 空文件 便于写入此空文件到apk包中作为channel文件
src_empty_file = 'info/temp.txt'
# 创建一个空文件（不存在则创建）
f = open(src_empty_file, 'w') 
f.close()

# 获取目录中所有的apk源包
src_apks = []
# 目标目录
dir='apks'
for file in os.listdir(dir):
    filepath = os.path.join(dir,file)
    if os.path.isfile(filepath):
        extension = os.path.splitext(filepath)[1][1:]
        if extension in 'apk':
            src_apks.append(filepath)

# 获取渠道列表
channel_file = 'info/channel.txt'
f = open(channel_file)
lines = f.readlines()
f.close()

for src_apk in src_apks:
    # 含后缀名的文件名
    src_apk_file_name = os.path.basename(src_apk)
    # 分割文件名与后缀
    temp_list = os.path.splitext(src_apk_file_name)
    # name without extension
    src_apk_name = temp_list[0]
    # 后缀名，包含.   例如: ".apk "
    src_apk_extension = temp_list[1]
    # 创建生成目录,与文件名相关
    output_dir = 'output_' + src_apk_name + '/'
    # 目录不存在则创建
    if not os.path.exists(output_dir):
        os.mkdir(output_dir)
        
    # 遍历渠道号并创建对应渠道号的apk文件
    for line in lines:
        # 获取当前渠道号，因为从渠道文件中获得带有\n,所有strip一下
        target_channel = line.strip()
        # 拼接对应渠道号的apk
        target_apk = output_dir + src_apk_name + "-" + target_channel + src_apk_extension  
        # 拷贝建立新apk
        shutil.copy(src_apk,  target_apk)
        # zip获取新建立的apk文件
        zipped = zipfile.ZipFile(target_apk, 'a', zipfile.ZIP_DEFLATED)
        # 初始化渠道信息
        empty_channel_file = "META-INF/CHANNEL_{channel}".format(channel = target_channel)
        # 写入渠道信息
        zipped.write(src_empty_file, empty_channel_file)
        # 关闭zip流
        zipped.close()
```

笔者的工作系统是Linux系统，Shell脚本也同样方便简单，所以翻译成Shell脚本的形式
```
#!/bin/bash
 

apkdir=apks
infodir=info
outdir=outputs
metadir=META-INF

# 创建配置目录
if [ ! -d "$metadir" ]; then
  mkdir $metadir
fi

# 创建输出目录
if [ ! -d "$outdir" ]; then
  mkdir $outdir
fi

touch $infodir/temp.txt

#添加渠道标识
function addChannel(){
    while read channel
    do
         resultApk=$(basename $selectApk .apk)_$channel.apk
         echo $resultApk
         cp $apkdir/$selectApk $outdir/$resultApk && cp $infodir/temp.txt $metadir/CHANNEL_$channel
         zip -r $outdir/$resultApk $metadir/CHANNEL_$channel
    done < $infodir/channel.txt
}

# 遍历目标目录APK文件
for selectApk in `ls $apkdir`
do 
   if [ "${selectApk##*.}" = "apk" ]; then
       echo $selectApk
       addChannel
   fi
done
```
原理都一样，如果是Windows系统，也可以弄个bat批处理也行，不过笔者目前没有windows系统，所以没写。

![](http://upload-images.jianshu.io/upload_images/2006464-20d2f78eba4eee9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

执行相应的脚本命令

![](http://upload-images.jianshu.io/upload_images/2006464-90a4924d9cb52df1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###源码
https://github.com/byhook/PackageRun
###参考
http://tech.meituan.com/mt-apk-packaging.html
