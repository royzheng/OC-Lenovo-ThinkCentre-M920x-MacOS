# M920x-Hackintosh-EFI
- Hackintosh Opencore EFIs for M920x
**请自行将OC/config.plist.example重命名为config.plist并补充三码信息**

# 机器配置

* Lenovo M920x
* CPU: i9-9900T
* Motherboard: Q370
* BIOS: [M1UKT77A](https://support.lenovo.com/us/zc/downloads/ds503907-flash-bios-update-thinkcentre-m720t-m720s-m720q-m920t-m920s-m920q-m920x-thinkstation-p330-tiny)
* iGPU：Intel UHD Graphics 630
* Mem: Lenovo 32G * 2
* NVMe: SN550
* Wifi/Bluetooth: BCM94352Z

# 系统信息

* 仿冒机型：iMac19,1（SN已去除，需自行补充）
* 系统版本：Sonoma 14.2.1
* DP和HDMI接口都正常，支持双屏输出（三屏没试过）
* 内置扬声器、前置耳机、麦克风输出正常。
* WiFi正常
* 蓝牙正常
* USB端口正常，端口属性正确修正
* 支持开机音效

# 存在问题
* 接力不行
* sequoia不行

# BIOS设置(关闭CFG LOCK等)
### ***请谨慎操作***
这部分BIOS参数在bios里没有界面可以操作，所以需要你自己通过提取参数及值进行设置，具体步骤如下
1. 准备一台windows。
2. 确定好你bios的固件版本，去官网下载rom，以我现在用的 [M1UKT77A](https://support.lenovo.com/us/zc/downloads/ds503907-flash-bios-update-thinkcentre-m720t-m720s-m720q-m920t-m920s-m920q-m920x-thinkstation-p330-tiny)为例，我下载的是BIOS Update (USB Drive Package) m1ujt77usa.zip的文件,解压后可见一个imageM1U.ROM的文件。
3. 以管理员身份打开[BIOS提取工具.EXE](./DOCS/Tools/BIOS提取工具.EXE),先读取步骤2提取的rom，后备份。
4. 打开[UEFITool.exe](./DOCS/Tools/UEFITool.exe)，将步骤3备份出来的rom文件拖到窗口，然后ctrl+f调出搜索，切换到GUID搜索框，输入899407D799FE43D89A2179EC328CAC21后搜索，双击检索结果会定位到具体的位置，然后右键位置选择菜单里Extract as is选项，保存为FFS文件。
5. 打开[IRFExtractor.exe](./DOCS/Tools/IRFExtractor.exe),然后选择步骤4提取出来的FFS文件，点Extarct导出成txt文件。
6. 打开步骤5提取出来的txt文件，然后搜索CFG Lock，比如在我的固件版本里找到的东西是这样
```
0x549E2 		One Of: CFG Lock, VarStoreInfo (VarOffset/VarName): 0x721, VarStore: 0x1, QuestionId: 0xB31, Size: 1, Min: 0x0, Max 0x1, Step: 0x0 {05 91 CE 03 CF 03 31 0B 01 00 21 07 10 10 00 01 00}
0x549F3 			One Of Option: Disabled, Value (8 bit): 0x0 {09 07 04 00 00 00 00}
0x549FA 			One Of Option: Enabled, Value (8 bit): 0x1 (default) {09 07 03 00 30 00 01}
0x54A01 		End One Of {29 02}
```
从上面这段文件可以看出CFG Lock的选项值是0x721,参数值是0x0(Disabled)或0x1(Enabled)

7. 准备个U盘，格式化成FAT32，然后新建目录 EFI, 在EFI下新建目录BOOT,在BOOT下放入[bootx64.efi](./DOCS/Tools/bootx64.efi)文件，然后重启从U盘引导启动。
8. U盘启动后进入终端环境，我们可以输入对应的命令来进行设置(根据步骤6提取的信息)
```
# 查看CFG Lock当前的值
setup_var 0x721
# 关闭CFG Lock
setup_var 0x721 0x0
```
9. 除了CFG LOCK之外，如果你要使用核显，请设置以下值
```
# DVMT Pre-Allocated设置成64MB
setup_var 0xA44 0x2
# DVMT Total Gfx Mem设置成max
setup_var 0xA45 0x3
```
10. 跟我bios版本一样的可以直接从步骤7弄起来，不需要去做提取，其他bios版本则自己去提取一下搜索相应的选项值和参数值。
# BIOS设置
- 恢复出厂设置：Exit->OS Optimized Defaults [Enabled]->Load Optimal Defaults
- 关闭安全启动：Security->Secure Boot->Secure Boot [Disabled]
- 关闭VD-T

# 安装后需要做的事
```
# 解决可能存在的睡死问题
sudo pmset -a sleep 0;
sudo pmset -a hibernatemode 0;
sudo pmset -a disablesleep 1;

sudo spctl --master-disable
```
* 下载最新OCLP：https://github.com/dortania/OpenCore-Legacy-Patcher/releases 中pkg格式的文件
* 安装下载的文件
* 运行OCLP，选择Post-Install Root Patch
* OCLP会请求root权限，输入密码后，等待进度完成并重启系统
