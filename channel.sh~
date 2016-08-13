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




