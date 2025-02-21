# Citrix

## 部署环境

1. 域控服务器-Windows Server  
    - [配置域及域控](#配置域)
    - 安装License Server  

2. Citrix控制机
    - [安装Citrix Hypervisor/XenServer](#虚拟机部署citrix-hypervisor)  

3. Citrix管理机-Windows Server
    - 安装XenCenter  
        Citrix Hypervisor的Windows GUI远程管理程序  

    - 安装DeliveryController套件  
        不能安装在域控服务器上  
        通过Citrix Studio创建Virtual Machine Catalog和Delivery Group并发布桌面或应用
4. 共享机
    - 安装Agent  
        将本机资源通过管理机的DeliveryController套件共享  

        该机可为物理机或任意虚拟机  
        关闭本机防火墙
5. 客户机  
    - 安装Workspace app  
        通过该应用远程连接至被共享机

## 配置域

### 设置网络

1. 将域控服务器及域内主机设置为同一网段
2. 服务器地址设置为静态IP
3. 安装DNS服务器

### 配置域控

1. 添加角色和功能=>Active Directory域服务
2. 将服务器提升为域控  
3. 配置AD域OU和用户

### 加入域

1. 设置DNS服务器为域内服务器(域控服务器)
2. ping域名检查域名是否与外网域重名，若重名修改域名
3. 「System Properties」=>「Computer Name」=>「Change」，使用域内用户加入域
4. (非必须)「Computer Management」=>「Local Users and Groups」=>「Groups」=>「Administrators」，添加域内用户
5. 切换系统用户至域用户

## 虚拟机部署Citrix Hypervisor

### 硬件需求

官方文件给出的最低需求，实际可能大于该需求

1. CPU 64bit, >=1.5GHz
2. RAM >= 2GB
3. 外存 >= 46GB
4. Network >= 100MB/s

### 安装步骤

1. 下载ISO镜像  
2. 按[硬件需求](#硬件需求)配置虚拟机
    > 若不暴露物理机的CPU虚拟化，稍后在安装时提示Citrix功能会受限  
    > 对于VMware，在「Settings」=>「Processors/CPU」=>「Virtualize Intel VT-x/EPT or AMD-V/RVI」，选项名依据VMware版本会有不同  
3. 从CD/DVD引导，选择下载的ISO
4. 按照安装引导进行操作
    - 格式化硬盘
    - 选择键盘布局
    - 同意EULA协议
    - 选择用作虚拟机存储的硬盘
    - 配置网络连接(DHCP/manual)
    - 是否验证完整性
    - 设置root密码
    - 指定主机(即安装Citrix系统的机器)名和DNS
    - 选择时区
    - 选择确定本地时间的方式(NTP/manual)
    - 确认安装XenServer
    - 选择是否安装补充包
    - 选择确认以重启
5. 重启后，将显示xsconsole，Alt+F3以切换到本地shell，Alt+F1以切换到xsconsole
