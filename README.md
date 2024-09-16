# Padavan
代码基于vb1980和keke1023，感谢v大持续为老旧的系统更新软件版本。
我做了一些整合优化。更新github  action云编译脚本和教程。
老设备padavan 3.4内核最终版！！

注意事项：
如果是硬改过32M内存的机器，如k2p 记得修改下编译脚本增加 CONFIG_32M_REBOOT_FIXUP=y  否则会有软重启失灵问题！！

编译了openvpn后，配置页面在网络地图-网络状态-更多设置的菜单里、不会出现在主页面的菜单中！

frps界面开关需要的自己去把那部分注释删掉就可以了！ https://github.com/fightroad/Padavan-KVR/commit/01dedd47494696e7a2c4fde2204e69a17f3f8942

测试ssp 绕过模式、全局、回国模式有问题，界面只保留了gfw模式！

# 版本特点：

1、mac系统打开页面响应慢的问题，适配手机页面，精简部分页面显示内容。

2、frp、zerotier等穿透插件版本更新。

3、ss /xray/trojan 版本更新，都正常使用。

4、iptv使用 msd_lite 大大降低资源占用.

5、定时重启失效、32M内存机器重启问题。

6、k2p mt7615 wifi芯片使用5.0.4.0驱动，支持打开kvr漫游。

7、5.0驱动理论支持7603/7615/7915的kvr，自己测试。编译的时候自己检查下配置的是否5.0驱动！
非这些wifi芯片的型号设置里也会有设置项，但是不起作用的！！

zerotier 使用技巧：

1、ap模式下如果想要其他zerotier端可以访问ap网段
#在padavan开机脚本里开启ap模式下的ip转发功能，虚拟网段改成你自己实际的。
```
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -s 10.11.12.0/24 -j MASQUERADE
```
2、解决当wan重拨号或其他原因，导致zerotier防火墙规则丢失，zerotier网络无法正常使用。可以在padavan设置  WAN 上行/下行启动后执行  加入下面的脚本。
切记不要放防火墙重启后里，测试有重启进不去系统的可能！！
```
if [ $1 == "up" ] ; then
SleepTime=30
logger -t "WAN状态改变" "【延时$SleepTime秒检测ZeroTier状态】"
sleep $SleepTime
KEYWORD="zte"
RULE_EXIST=$(iptables -L -n -v | grep "$KEYWORD")
NVRAM_ZEROTIER_ENABLE=$(nvram get zerotier_enable)
if [ -z "$RULE_EXIST" ]; then
    if [ "$NVRAM_ZEROTIER_ENABLE" = "1" ]; then
        logger -t "检测结果" "【ZeroTier防火墙规则不存在，但服务已启用，需要重启服务！】"
        zerotier.sh stop && zerotier.sh start
    else
        logger -t "检测结果" "【ZeroTier防火墙规则不存在，但服务未启用，不需要重启服务！】"
    fi
else
    logger -t "检测结果" "【ZeroTier防火墙规则存在，不需要重启服务！】"    
fi
fi
```
固件默认wifi名称
 - 2.4G：机器名_mac地址最后四位，如K2P_9981
 - 5G：机器名_5G_mac地址最后四位，如K2P_5G_9981

wifi密码
 - 1234567890

管理地址
 - 192.168.2.1

管理账号密码
 - admin
 - admin

# 一、ACTION一键自编译方法

1、fork我的代码，从已经有的workflow里复制一个模版。或者直接找一个模版进行修改。

![1](https://github.com/fightroad/Padavan-KVR/assets/39027157/611c13d7-0b32-4eb1-80d8-6bebaaf6913c)

2、新建一个action脚本

![2](https://github.com/fightroad/Padavan-KVR/assets/39027157/42120ab1-e245-4f04-a984-27f381e82565)

![3](https://github.com/fightroad/Padavan-KVR/assets/39027157/75cefd7d-fe60-40c8-87b1-1c250c98e425)

![4](https://github.com/fightroad/Padavan-KVR/assets/39027157/a5c28213-3bbb-4ad5-872e-a3639acb6496)


3、自定义自己的脚本，想要编译进去的软件直接按照编译脚本的模式增减。也可以直接修改template里面对应机型的配置。
脚本里面按照提示，先写上删除配置项的语句，再写上要增加的插件的语句！！#为注释符号
注意：action一键编译不会理会build_firmware_modify 脚本里面的配置。template里配置了和脚本里一样的配置会直接使用脚本里的！！！

![配置1](https://github.com/fightroad/Padavan-KVR/assets/39027157/4bc31b0d-a1c8-4ed9-8ff7-f6babb060ba5)


# 下图说的删除是指在那部分内容中增加删除配置的命令，不是把原来有的命令内容删掉！！！

![image](https://github.com/fightroad/Padavan-KVR/assets/39027157/aed2259d-125d-4f71-bf4c-9e4fef141656)


4、开始编译，获取编译后的固件

![5](https://github.com/fightroad/Padavan-KVR/assets/39027157/15c6aed1-41b9-41e2-b13b-a5fd6063cbd6)

![6](https://github.com/fightroad/Padavan-KVR/assets/39027157/9ab4089e-0eab-4728-8cdf-3960e503f640)


# 二、本地编译参照chongshengb的代码编译。增减插件修改 build_firmware_modify 脚本或直接修改template里面的机型配置。
![image](https://github.com/fightroad/Padavan-KVR/assets/39027157/94e9076a-965c-4632-8a59-896b7dadbd09)


# 固件简介
基于hanwckf,chongshengB以及padavanonly的源码整合而来，支持7603/7615/7915的kvr  
编译方法同其他Padavan源码，主要特点如下：  
1.采用padavanonly源码的5.0.4.0无线驱动，支持kvr  
2.添加了chongshengB源码的所有插件  
3.其他部分等同于hanwckf的源码，有少量优化来自immortalwrt的padavan源码  
4.添加了MSG1500的7615版本config  

以下附上他们四位的源码地址供参考 

https://github.com/hanwckf/rt-n56u

https://github.com/chongshengB/rt-n56u

https://github.com/padavanonly/rt-n56u

https://github.com/immortalwrt/padavan

