# Property Scanner

## Introduction

该脚本用于企业资产网络访问权限检查。脚本基于预先设定的各区域访问控制权限要求及运行主机所处网络区域来判断网络权限开放是否合规。

## Runtime Environment

- Support for Windows & Linux & MacOS

- Python 3

- Requirements : xlwt, xlrd, xlutils

  ```shell
  $ pip install xlwt, xlrd, xlutils
  ```

- Masscan
  - Windows : 需要自行安装 [winpcap](https://www.winpcap.org/install/default.htm) 而不需要安装 Masscan
  - Linux : 需要自行安装 Masscan

> Untested on Python 2 because I'm lazy  : )

## Configuration

目录结构如下：

```bash
PropertyScanner
├── README.md
├── propertyscan.py
├── requirements.txt
├── property.json
└── zone.json
```

### property.json

使用前我们需要先配置 property.json，该文件记录所有资产的端口及允许开放区域

以下为一个最基础的企业资产网络访问权限记录单元，每个单元以 IP 为主键，每个 IP 可以下挂多个系统。每个系统单元记录系统名称「system」、系统负责人「owner」以及管理权限要求「auth」。

```json
{
    "192.168.1.1": [
        {
            "system":"堡垒机",
            "owner":"Fmelon",
            "auth":[
                {
                    "ports":[22,443,3389],
                    "openzone":["IT_Department"]
                }
            ]
        }
    ]
}
```

因为文件的核心是记录资产每个敏感端口允许开放的区域，所以每个管理权限要求「auth」可以下挂多个端口准许区域，像下面这样：

```python
# 此处示例省略了父级信息
"auth":[
    {
        "ports":[22,443,3389],
        "openzone":["IT_Department"]
    },{
        "ports":[80],
        "openzone":["office"]
    }
]
```

其中，每个 「ports」 和 「openzone」都可以在列表中填写多个对应值。具体可以参考项目中同名示例文件。

### zone.json

该文件的作用主要为确定当前主机 IP 所处的网络区域，由于网络复杂性，该 IP 所具有的网络区域标签可能有多个。

一个基础的网络区域记录单元的定义如下：

```python
{
    "office": {
        "name":"办公区",
        "authIP":[
            "192.168.1.1/24"
        ]
    }
}
```

同理，一个区域的所属 IP 段「authIP」也可以有多个，比如：

```python
# 此处示例省略了父级信息
"authIP":[
		"192.168.1.1/24",
  	"192.168.8.2"
]
```

具体可以参考项目中同名示例文件。

> 注意：IP 格式支持 `10.10.0.1/24`，`10.10.0.1`，暂不支持 IP 区间。

### 其他

默认设定，若脚本同级目录下存在 porperty.json 和 zone.json，则会优先读取文件中的数据。

为保证脚本程序简洁性，支持将 porperty.json 和 zone.json 内容直接写入脚本中，只需替换脚本文件「main」函数中的「iplist」以及「zoneDict」值，并删除脚本同级目录下的 porperty.json 和 zone.json 文件即可。

## Usage

在配置好 porperty.json 和 zone.json 后，执行该脚本即可

```shell
$ python porpertyscan.py
```

执行后同级目录下会生成名为 tmpfile 的文件夹以及名称为日期时间的 Excel 表格，表格内为扫描结果，状态标记为异常则可能存在网络访问权限控制不严的问题。

> 注：tmpfile 文件夹内保存了 masscan 扫描的结果，可以删除。同时如果脚本运行在windows 平台则会在该文件夹下生成 masscan.exe 用于扫描。该扫描工具杀毒软件可能会以恶意扫描工具报毒，请加白。另本人不对该软件（masscan）的安全性负责，如果有担心，也可以替换为 Masscan 官方项目编译的版本。
