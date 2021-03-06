# mySuperPoint
数据集生成方法是在[Superpoint](https://github.com/rpautrat/SuperPoint)中Synthetic dataset的生成方法

## refactor
* 预估epoch时间
* 打印数据集信息
* 保存最好的model
* ACC
* optional truncate ellipse 0.3 strides 0.2 gaussian noise 0.1
* homo photo augmentation

## tf版实现笔记
* 网络结构与pytorch版一样

训练
```shell
python experiment.py train configs/magic-point_shapes.yaml magic-point_synth
```
第一个参数是指定进行训练，第二个参数是配置文件目录

第一次训练时，会自动生成数据集

experience.py(line 150) - > _cli_train(line 81) 
-> train(line 19) -> _init_graph(line63)
-> get_dataset -> datasets/__init__.py 返回SyntheticShapes(**config['data'])，
SyntheticShape的__init__方法会合并类自带的default_config和config['data']这两个字典作为
最后的config，然后调用_init_dataset和_get_data生成数据集,其中_init_dataset负责划分文件目录，
然后用dump_primitive_data来预处理图片

* bn层竟然放在激活函数的后面，但是网上说放在激活层后面效果也不错？
* tf实现版本有batch normalization，在relu之后

## TODO
* 浏览数据集，记下笔记 done
* 测试数据集生成方法 done
* 找到原数据集生成的代码 done
* 根据实际场景，结合Opencv函数，找到扩展数据集的方法
* 需要有label生成前后的对比 done
* 加上TensorboardX可视化训练过程
* 正负样本差太多导致网络倾向全部判定为无关键点
* 即便可能关键点的激活值很小，也可以保留topk个作为参考
* pixel shuffle in pytorch
* pretrained model的置信度的门限值竟然是0.015,tensroflow版的置信度门限是0.001,或许门限设低一点可以解决问题？
* 并不行，检测出的图片兴趣点分布在边框上？发现pretrained model有一个去除边框上一定范围内的点的操作，为什么？
* 借鉴tf实现中的使用argmax来做label生成？
* tf版本
* 调查pretrained model的输出scale的范围
* 每轮的输出加上准确率和召回率
* nms是干啥的
* 加上在relu后的batch normalization
* 迅速的过拟合是否表明需要给网络做正则化？
* 奇怪的现象，即便test_loss变大了，但是precision和recall竟然也变大
* 记录multiprocess的用法
* 理顺precision和recall在边界上的计算方式
* 计算pr曲线的时候要不要加上nms

## 坑
直接将(H,W)reshape成(Hc,Wc,64)和先将(H,W)reshape成(Hc,8,Wc,8)然后transpose成(Hc,Wc,8,8)再reshape成(Hc,Wc,64)是不一样的结果

对比训练过程发现，我的实现永远在Loss降到0.28左右的时候不再下降，可能是因为？
    * 学习率
    * bn
    * 奇怪的nms
    * homogra????
    
预测出的x,y是反的

论文上的各项都给的很明确了，由于没有仔细看复现的时候踩了不少坑，其中包括
* 使用Adam及其默认参数
* MagicPoint的数据集是on the fly生成的，每个生成的图片会被homographic wrap来进行augmentation，保证没有样本出现过两次
* 提到了使用BatchNormalization
* 给了参数Nh=100
* 包括随机高斯噪声、motion blur，brightness level changes来提高模型的robustness
* 训练的一些参数

## log
更改point2label写法，提高了一倍的速度

加上了batch normalization，loss可以降低到0.16左右

加上learning_rate decay，loss可以降到0.13左右

换上Adam，learning rate = 0.001，loss可以降到0.0几了，预测也正确了

## 未解决
我没有使用homographic wrap来进行训练

## Homography在opencv的格式
[0,0],[1,0],[1,1],[0,1]

## Reference
* [Superpoint](https://github.com/rpautrat/SuperPoint)，基于tensorflow的SuperPoint
* [SuperpointPretrainedNetwork](https://github.com/MagicLeapResearch/SuperPointPretrainedNetwork)，
作者预训练的SuperPoint模型，可以调用到图片、视频和摄像头上，但作者明确说明不会开放训练代码、合成数据集
* [demoasd](https://www.youtube.com/watch?v=gtzxuET74Mk) ,youtube上的视频Demo，在高分辨率的情况下似乎有很大的问题
* [关于损失函数](https://zhuanlan.zhihu.com/p/54969632)
* [map计算方法](https://www.jianshu.com/p/82be426f776e)
* [pytorch geometry](https://torchgeometry.readthedocs.io/en/latest/warp_perspective.html)