# 人头检测模型部署

目标是转换为rknn文件部署到瑞芯微的板卡上使用

## pt-onnx

要转换为rknn文件首先要从pt转换为onnx

onnx是一个开放的交换格式，方便各种框架之家互相转换 大部分转换流程的中间过程都会有一个onnx

这边就从头开始安装环境吧

![f6a75b1e5c728e975866ca096babe24](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\f6a75b1e5c728e975866ca096babe24.png)

![8a80e39236112395357b5d2028feeb8](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\8a80e39236112395357b5d2028feeb8.png)

如果下载速度很慢的话可以使用清华源

![0a6b76c1275b605f6e7890c3fcf1489](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\0a6b76c1275b605f6e7890c3fcf1489.png)

安装了yolov5的库后 直接安装requirements.txt文件

![eaa633eef307508840179b7919d3e70](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\eaa633eef307508840179b7919d3e70.png)

### 遇到的第一个问题

![0863775eb6803025e01bca4b6616903](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\0863775eb6803025e01bca4b6616903.png)

一开始看不懂就先问了一下chatgpt 他告诉我是版本不对 但是我觉得不应该 因为提示我C3没有 但是5.0版本了肯定会有C3 难道是我下错了？ 然后去检查这个文件果然没有 在网页中的experimental.py这个文件找C3这个函数 复制过来 问题解决  

为了避免类似的问题 直接把仓库中model文件的代码全复制到yolo5源代码上 然后再执行

![a120560e8c3c452945d00048a081997](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\a120560e8c3c452945d00048a081997.png)

又出现一个问题 这个问题一看就知道是我yaml文件没设置好 在代码仓库中找到对应的yaml文件 复制过去 再次执行

![714a08cfc6a7142d6dbc596fd003ec6](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\714a08cfc6a7142d6dbc596fd003ec6.png)

转换为onnx成功

## onnx到rknn

这个问题因为有以前做过的经验 所以碰到的意外比较少

首先使用RKNN-Toolkit2工具配置好环境 进入packages文件夹下面  安装对应的环境文件

![10463985e25b9ad040e8767ad8887b1](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\10463985e25b9ad040e8767ad8887b1.png)

然后下载rknn_model_zoo 完成模型转换

进入到对应的yolo5文件夹下  先修改coco_80_labels_list.txt的内容

![81667787f944c6452937d53d23f83d0](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\81667787f944c6452937d53d23f83d0.png)

改成两个类别 然后把之前准备的onnx文件放在这个文件夹下

然后使用他提供的python文件convert.py完成模型转换

![8f1484e711c83117cf15c8deb6068b1](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\8f1484e711c83117cf15c8deb6068b1.png)

![5b48af3f2d62685e929ade3de637b60](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\5b48af3f2d62685e929ade3de637b60.png)

## 部署到rk3588

首先去GitHub上下载rknpu

注意版本 和之前转换用的RKNN-Toolkit2版本一样

然后部署交叉编译链 方便在本机上先编译好 再转到板卡上运行

```
sudo apt-get install gcc-9-aarch64-linux-gnu g++-9-aarch64-linux-gnu
```

下载交叉编译链

git clone https://gitlab.com/firefly-linux/prebuilts/gcc/linux-x86/aarch64/gcc-buildroot-9.3.0-2020.03-x86_64_aarch64-rockchip-linux-gnu.git

然后修改rknpu中的.sh脚本

```
1. export TOOL_CHAIN=<your-proj-path>/gcc-buildroot-9.3.0-2020.03-x86_64_aarch64-rockchip-linux-gnu
2. export CC=${GCC_COMPILER}-gcc-9
3. export CXX=${GCC_COMPILER}-g++-9
```

然后把自己的rknn文件 对应的.txt文件替换掉他原来的 就可以开始编译

![db95c8255b8002b25d546e7068969d9](D:\Documents\WeChat Files\wxid_0wqw918isk9x12\FileStorage\Temp\db95c8255b8002b25d546e7068969d9.png)

编译成功 就可以打demo文件发送到板卡

用ssh远程连接既可发送到板卡上

在板卡上执行

```
./rknn_yolov5_demo ./yolov5.rknn ./1.jpg
```

### 出现问题

![out](C:\Users\Administrator\Desktop\out.jpg)

看到这种情况 我第一反应是感觉我前面的转换步骤哪里出了问题，又看了一下结果输出 发现置信度＞100% 

首先怀疑量化出了问题 但是我有点无从下手不知道哪里改

那就先去看看后逻辑部分

