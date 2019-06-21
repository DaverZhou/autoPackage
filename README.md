# 脚本自动化打包
iOS 使用脚本自动化打包主要是包含两种方式：
- `exportArchive`
- `PackageApplication`
别的方式上大同小异，这边就简单介绍下。本文就以相同部分、不同部分来区分。因为个人觉得一般项目都是用git来管理、方便团队开发，所以源码的路径直接就是git仓库了。
### git仓库
`Git仓库`不管是用GitHub、gitee、gitlab都是一样的，因为直接用的是用脚本为了方便请在打包机上配置好,如果需要请参考另一篇[Git配置SSH](https://www.jianshu.com/p/27a595cc8d2b)。
脚本：
```
# pull source  && clone git and pull source
if [ -d $SOURCEPATH ]; then
    cd $SOURCEPATH
    git clean
    git pull origin master
    cd 工程路径
else
    mkdir $SOURCEPATH
    cd $SOURCEPATH
    git clone 远程仓库url.git
    git pull origin master
    cd 工程路径
fi
```
### 打包
#### 方式一：exportArchive
使用这个方式打包，可以制作成测试包或者生产包，当然还需要附带上两个文件`DEVELOPExportOptionsPlist.plist`、`AppStoreExportOptionsPlist.plist`，这两个文件在相应对应的ipa包中是可以找到的。这两个文件不需要更改固定的就行了，需要的话可以去[github仓库](https://github.com/DaverZhou/autoPackage/tree/master)下载。
##### 测试包
```
# 构建archive
xcodebuild -workspace $workspace_Name.xcworkspace -scheme $target_Name -configuration $Configuration -archivePath 存放xcarchive文件的路径/$target_Name-develop.xcarchive clean archive build
# 生成ipa
xcodebuild  -exportArchive -archivePath 存放xcarchive文件的路径/$target_Name-develop.xcarchive -exportOptionsPlist ${DEVELOPExportOptionsPlist} -exportPath 导出ipa包的文件路径/develop-$ipa_Time
```
`$workspace_Name`：这边为工程名，脚本内直接定义了变量方便更改。
`$target_Name`：target的名字，可以直接打开自己的工程查看，也是个变量名
`${DEVELOPExportOptionsPlist}`：用于区别测试包的文件，也是个变量
`$ipa_Time`：日期变量，用于区别不同时间段打的包
这边因为打的是测试包，后面在打包完成后可以将包上传到fir、或者蒲公英上，下面就以fir为例：
```
fir login fir的登录token
fir publish 导出ipa包的文件路径/develop-$ipa_Time/LoveChat.ipa
```
##### 生产包
跟打测试包大同小异，区别在于`DEVELOPExportOptionsPlist`这个文件，具体如下：
```
#构建archive
xcodebuild -workspace $workspace_Name.xcworkspace -scheme $target_Name -configuration $Configuration -archivePath 存放xcarchive文件的路径/$target_Name-dis.xcarchive clean archive build
#生成ipa
xcodebuild  -exportArchive -archivePath 存放xcarchive文件的路径/$target_Name-dis.xcarchive -exportOptionsPlist ${AppStoreExportOptionsPlist} -exportPath 导出ipa包的文件路径/distribution-$ipa_Time
```
这种方式的打包就讲到这边，下面将另外一种打包的方式。
##### 完整的脚本
``` sh
#!/bin/sh
# package time
archive_Time=`date "+%Y-%m-%d %H:%M:%S"`
echo "======================================="
echo "开始打包，时间："$archive_Time
echo "======================================="

# ipa time
ipa_Time=`date "+%Y-%m-%d"`
# project name
target_Name="工程名"

# workspace name
workspace_Name="工程名"

# 配置环境:Release or Debug,default Release
Configuration="Release"

# 加载各个版本的plist文件，这两个版本的无需修改任何内容
DEVELOPExportOptionsPlist=`PWD`/DEVELOPExportOptionsPlist.plist
AppStoreExportOptionsPlist=`PWD`/AppStoreExportOptionsPlist.plist

DEVELOPExportOptionsPlist=${DEVELOPExportOptionsPlist}
AppStoreExportOptionsPlist=${AppStoreExportOptionsPlist}

# project path
MAINPATH=`PWD`
BUILDPATH=`PWD`/build
SOURCEPATH=`PWD`/source

# file
if [ ! -d $BUILDPATH ]; then
mkdir $BUILDPATH
fi

# pull source  && clone git and pull source
if [ -d $SOURCEPATH ]; then
cd $SOURCEPATH
git clean
git pull origin master
cd LoveChat/LoveChat
else
mkdir $SOURCEPATH
cd $SOURCEPATH
git clone 远程仓库源码路径.git
git pull origin master
cd LoveChat/LoveChat
fi

# 选择打包类型
echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "        1 Develop"
echo "        2 AppStore"

# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]; then

if [ "$method" = "1" ]; then # develop
echo "~~~~~~~~~~~进入develop~~~~~~~~~~~"
# 构建archive
xcodebuild -workspace $workspace_Name.xcworkspace -scheme $target_Name -configuration $Configuration -archivePath $BUILDPATH/develop-$ipa_Time/$target_Name-develop.xcarchive clean archive build
# 生成ipa
xcodebuild  -exportArchive -archivePath $BUILDPATH/develop-$ipa_Time/$target_Name-develop.xcarchive -exportOptionsPlist ${DEVELOPExportOptionsPlist} -exportPath $BUILDPATH/develop-$ipa_Time

#upload to fir
fir login fir的token
fir publish ${BUILDPATH}/develop-$ipa_Time/$workspace_Name.ipa

elif [ "$method" = "2" ]; then # distribution
echo "~~~~~~~~~~~进入distribution~~~~~~~~~~~"
#构建archive
xcodebuild -workspace $workspace_Name.xcworkspace -scheme $target_Name -configuration $Configuration -archivePath $BUILDPATH/distribution-$ipa_Time/$target_Name-dis.xcarchive clean archive build
#生成ipa
xcodebuild  -exportArchive -archivePath $BUILDPATH/distribution-$ipa_Time/$target_Name-dis.xcarchive -exportOptionsPlist ${AppStoreExportOptionsPlist} -exportPath $BUILDPATH/distribution-$ipa_Time
fi

else
echo "参数无效...."
exit 1
fi
```
#### 方式二：PackageApplication
xcode升级到8.3后，`PackageApplication`这个工具就没了，但素可以将[旧版本的工具](https://github.com/DaverZhou/autoPackage/tree/master)中导入，下载后放到下面的路径：
```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/
```
执行如下命令授权:
```
$sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/
#chmod +x /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/PackageApplication
```
如果没有上面的处理，用命令进行打包提示下面这个错误：
```
xcrun: error: unable to find utility "PackageApplication", not a developer tool or in PATH
```
##### 打包
```
xcodebuild -workspace 工程名.xcworkspace -scheme 工程名 -configuration Release -derivedDataPath 导出路径
# package convert .app to .ipa
xcrun -sdk iphoneos PackageApplication -v 导出路径/Build/Products/Release-iphoneos/工程名.app -o 导出ipa包路径/LoveChat.ipa
```
这边如果是比较大的项目，那肯定是需要考虑打包的速度，测试包嘛那就需要快点，那是不是可以只打`arm64`的包：
```
xcodebuild -workspace 工程名.xcworkspace -scheme 工程名 -configuration Release -arch arm64 -derivedDataPath $BUILDPATH
```
觉得速度还不够的话，是不是可以考虑增加上增量编译？？？？？这边就先提供思路，具体实现em...暂时么有。
##### 完整的脚本
```
#!/bin/sh
# package time
archive_Time=`date "+%Y-%m-%d %H:%M:%S"`
echo "======================================="
echo "开始打包，时间："$archive_Time
echo "======================================="

# project path
MAINPATH=`PWD`
BUILDPATH=`PWD`/build
SOURCEPATH=`PWD`/source
# file
if [ ! -d $BUILDPATH ]; then
    mkdir $BUILDPATH
fi

# pull source  && clone git and pull source
if [ -d $SOURCEPATH ]; then
    cd $SOURCEPATH
    git clean
    git pull origin master
    cd 工程路径
else
    mkdir $SOURCEPATH
    cd $SOURCEPATH
    git clone 远程仓库.git
    git pull origin master
    cd 工程路径
fi

# build
# only package arm64
xcodebuild -workspace LoveChat.xcworkspace -scheme LoveChat -configuration Release -derivedDataPath $BUILDPATH
# package convert .app to .ipa
xcrun -sdk iphoneos PackageApplication -v ${BUILDPATH}/Build/Products/Release-iphoneos/LoveChat.app -o ${MAINPATH}/App名.ipa

#upload to fir
fir login fir的token
fir publish ${MAINPATH}/App名.ipa
```
### 权限问题
在执行脚本的时候可能会错误，提示权限不足，那就给脚本授权：
```
chmod a+x 脚本.sh
```
### 调用
脚本写好后就是调用了，打开终端cd到对应的文件夹路径
```
$./脚本.sh
```
### 结语
有了脚本，手动打包是不可能的了。都是一句命令的事为啥非要手动打包，em...
