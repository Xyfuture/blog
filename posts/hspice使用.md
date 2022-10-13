---
title: "HSPICE简单使用"
date: 2022-10-13
draft: false
---

# 启动配置

我们首先需要配置一下license,目前有两种配置方案,针对旧版本的hspice,我们可以直接设置`LM_LICENSE_FILE`环境变量,其值为license文件地址.针对新版本的hspice需要借助scl软件完成认证,而scl仍旧是借助flexlm进行验证的,不过我们需要起起一个flexlm server.在lmtools.exe中设置一下license文件,并启动server,之后在环境变量中添加环境变量`SNPSLMD_LICENSE_FILE`,其值为`port@computer_name`,之后scl会使用该端口连接flexlm server进行验证.

hspice提供了一个简单的ui:`hspui.exe`,我们可以设置一下.

Configuration选项中有个option,点进去后设置一下hspice的地址和wave view的地址即可,不过wave view没找到可用的版本,只能用awaves代替了,但是awaves其实有点小bug,直接用hspui打开会报个错,需要手动打开文件

![image-20221013214937260](https://s2.loli.net/2022/10/13/nkbdypY1eiL7Ma9.png)

设置完了一定要保存配置,在File选项卡中有一个save,点一下就行了.

![image-20221013215341636](https://s2.loli.net/2022/10/13/D4FjpzvUcawdlN7.png)

# 语法部分

![image-20221013221121062](https://s2.loli.net/2022/10/13/ot8KMWhq5E6I9z1.png)

这个图还漏了一点,后面还可以加.dc .ac等电压扫描语句.measure .print等输出语句. 注意最后的 **.end**一定不能漏,否则不能仿真

具体用法看berkeley的教程吧 [hspice tutorial](https://inst.eecs.berkeley.edu/~ee105/sp11/tutorials/HSPICE_Tutorial.pdf)

具体的语法有点奇怪,大致主要是在写网表,端口之间应该如何连接

老师给的例子也可以看看

# 波形查看

由于找不到最新的Custom Wave View,只能使用古老的Avan Waves了,总之是比较难用

如果新版本的hspice输出的结果不能看,在sp文件中添加一句`.opt post = 2`

正常直接打开sp文件,就会自动搜索相关的结果,然后显示,如果不想打开sp文件,直接打开波形的话,可以设置filter,选择其他的文件

![image-20221013221931493](https://s2.loli.net/2022/10/13/rPZuab1KISUXF3w.png)

打开后自己选择想要的信号,可以自定义x轴和y轴,不同的扫描会对应在不同sw中(sw0,sw1)

![image-20221013222039512](https://s2.loli.net/2022/10/13/CiWtpF85nGedvPY.png)

打开后就能看到波形了,右键坐标轴就能设置log显示,上面的几个按键能够提供缩放,查看曲线中具体点的值的功能.

波形不能直接转换为图片,但是可以曲线救国,ctrl+p打印,选择输出到文件,然后选择eps格式的文件,保存的时候要手动填上后缀名eps,最后找个在线的eps转jpg的网站就可以获得图片了,或者有PS可以直接打开eps格式的文件.

