# novelai_amdgfx_deployguide

本教程（伪）主要介绍在amd显卡平台上部署novelai（SD、SDWebui）的一些要点。

# Windows平台部署SD

A卡用户在windows系统上部署SD的主要参考教程为这一篇：https://rentry.org/ayymd-stable-diffustion-v1_4-guide。

大致内容如下：

1. 安装Python和git，对windows平台而言，只需要到对应官方网站下载适合的安装包即可。python如无法通过命令行执行，可能需要手动添加环境变量。如果执行python时唤醒应用商店的，请在windows设置中找到对应选项关闭。

2. 注册huggingface.co，这是因为后续镜像部署需要从该网站上取得与用户对应的密钥，在部署端上输入后才能正常进行。

3. 创建一个文件夹，依次执行前述教程所提到的的命令。教程的第一步是部署github项目上的文件；第二步是安装onnx主程序，该软件是在windows上执行ai计算的关键；第三步是从huggingface.co上将数据库等文件拉下，此时需要使用注册后在账户界面处可生成的密钥，在电脑上正确填写后，方可开始部署流程。

4. 在执行“python ./save_onnx.py”这步时，如执行快速返回结果，请排查python主程序可能的问题，如环境变量是否未添加，以及python唤醒应用商店的相关控制面板开关是否未关闭等。

5. 整个部署时间可能需要10~20分钟，因此耐心等候是关键

6. 如果能够正常执行"python ./dml_onnx.py"，并生成一个在月球上骑马的宇航员的小图片的，说明部署成功。

   在Windows平台上部署SD的优势在于使用较为方便，且支持核芯显卡生成图片（只要内存足够）。其缺点在于操作麻烦，缺乏可视化和交互能力，没有webui或gui提供，且上述教程默认带的数据库更偏向日常生活，无法对二次元内容做到精确表达，二者的数据库在类型上也不通用。因此，如果希望获得良好的二次元图像生成体验，建议在linux平台上尝试部署SDWebui。

#Liunx平台部署SDWebui

在linux平台上部署SDWebui（novelai）主要参考这一篇教程https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-AMD-GPUs。

具体步骤如下：

1. 安装python3，git，pip等一系列依赖软件；

2. 按照教程第1部分的前三个步骤逐步执行；

3. 在执行第四步（即下载torch、torchvision并执行launch.py安装启动主程序）前，请注意显卡驱动中rocm的版本应高于torch中的版本，否则可能导致执行报错：“return torch._C._cuda_getDeviceCount() > 0 False“;。查看rocm版本（主要是HIP版本）的命令为：

// hipconfig

笔者使用rocm5.2版本搭配1.12版本torch，liunx版本为ubuntu 20.04，使用正常。

4. 在执行launch.py安装主程序的过程中，可能出现的问题包括但不限于:
  
    (1) 提示cuda核心错误（该错误有两种类型，前文提到的rocm版本不对应是其中的一种），此时可以按照报错提示，手动在launch.py的指定段落中添加跳过cuda认证的相关内容，先行完成安装。但如果报错信息为：”ErrorNoBinaryForGpu: Unable to find code object for all current devices!"Aborted (core dumped)”。说明显卡可以被torch识别，rocm运行成功，此时建议在执行python3 ./launch.py --xxx --xxx的命令之前，添加如下内容：HSA_OVERRIDE_GFX_VERSION=10.3.0，该命令的目的在于将rdna2架构的独显伪装成代号为gfx1030的专业显卡。
   
   （2）网络错误（如提示ssl认证不安全，或提示TLS握手失败等），一般都是由于网络不稳定所致，不是程序错误。
   
   （3）torch、gfpgan安装速度慢，一般都是由于网速慢，文件大所导致，需要格外耐心等待。
   
   （4）其他问题，如提示socks出现问题（使用socks代理加速时会出现），缺乏数据库（一般需要放在module文件夹下的规定位置），在安装过程中可能需要重复多次执行launch.py程序。使用socks加速时，无法启动主程序，如果使用的是export命令构架的proxy，请使用unset命令清除。
  
    额外需要注意的是：由于教程里将所有相关依赖都存放在了venv文件夹中，因此请勿在cmd命令提示符首端没有venv标识时执行launch.py，否则相关依赖与数据库将会被重新下载到默认的lib文件夹中，占用两次磁盘空间。

5. 如果顺利的话，执行到最后所可能碰到的问题会有三个，其一是前文所提到的因电脑rocm版本与torch适配的rocm版本不对应所导致的torch无法识别显卡问题；其二是因安装rocm时没有将执行用户分类进用户组（group）所导致的torch无法识别显卡问题（虽然我也不清楚原理是什么）；其三则是amd显卡未给予官方支持所导致的问题。
  
   （1）第一种情况的解决取决于rocm版本与torch版本的匹配，在此不再赘述；
    
   （2）第二种情况的解决需要将用户分类进video用户组（UBuntu20.04版本还需要将用户分类进render），具体命令为：sudo usermod -a -G video【或者render】$LOGNAME；
   
   （3）第三种情况的具体提示内容为：“"hipErrorNoBinaryForGpu: Unable to find code object for all current devices!"Aborted (core dumped)”。此时应在执行python3 ./launch.py --xxx --xxx的命令之前，添加如下内容：HSA_OVERRIDE_GFX_VERSION=10.3.0，该命令的目的在于将rdna2架构的独显伪装成代号为gfx1030的专业显卡。

    在linux上的部署虽然稍显复杂，但体验相对而言也会好上许多。webui所提供各项微调、定制以及训练模型等功能也非colab版本可比的。但缺点在于其图像生成质量在我看来不及colab版本，且受到电脑性能影响较大。笔者使用笔记本6600M渲染多张照片，只要分辨率高于512*512，就会有爆掉8GB显存的风险。对于新手而言并不友好，各项指标都有较大的优化空间。

# 简单总结

1. 对于多数人来说，使用colab一键部署不仅方便快捷，而且生成图片速度快，生成图像质量高，是最适合新手尝鲜的选择；

2. Windows部署的优势在于可以通过onnx的广泛兼容性使各类AMD显卡都可进行AI图像生成，但缺点在于其暂时没有好用的可视化客户端，对A卡而言价值不大。

3. Linux部署的优势在于可以最大限度的自由度，且拥有多种增效工具（如MIOPEN等），可玩性极高。但缺点在于部署门槛高，问题解决耗时耗力。
