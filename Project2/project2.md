# Project 2

## Switch 基本設定

- 每台 switch 皆需要
- 將 Hostname 設爲標籤上的名稱
    
    ```
    # On Every Switch
    (config)# **hostname** *hostname*
    ```
    
- 增加本地帳號 (local account)
    - 帳號: `ccna`
    - 密碼: `ccna`
    - 以 **md5** 形式存在 configuration 裏
    - CS-Core console 操作需登入本地帳號
    
    ```
    # On Every Switch
    (config)# **username** ccna **secret** ccna
    
    # Only On CS-Core
    (config)# **line console** 0
    (config-line)# **login local**
    (config-line)# **exit**
    ```
    
- 設定 enable 密碼
    - 密碼: `project2`
    - 以 **md5** 形式存在 configuration 裏
    
    ```
    # On Every Switch
    (config)# **enable secret** project2
    ```
    
- 設定 ssh
    - 使用 `cs.nycu.edu.tw` 爲 domain name
    - modulus length 設為 `2048`
    - version 設為 `2`
    - 僅能使用 ssh 登入，關閉 telnet 登入 **（套用到所有 vtys）**
    
    ```
    # On Every Switch
    (config)# **ip domain-name**
     cs.nycu.edu.tw
    (config)# **crypto key generate rsa**
    *How many bits in the modulus [512]:* **2048**
    (config)# **ip ssh version 2**
    (config)# **line vty 0 15**
    (config-line)# **transport input ssh**
    (config-line)# **login local**
    (config-line)# **exit**
    ```
    
- 關閉所有沒有在使用的 interface（沒有接線的 interface）
    
    ```
    # **show interface | include down**
    # **configure terminal**
    (config)# **interface range** *type number* **-** *type number*
    (config-if-range)# **shutdown**
    (config-if-range)# **exit**
    or
    (config)# **interface** *type number*
    (config-if)# **shutdown**
    (config-if)# **exit**
    ```
    
- Switch 不允許向終端 (edge) 發送 CDP
    - CSCC 內的 Switch 除了往上行 NYCU IT 走之外都算終端設備 (edge)
    - **請勿將不發送設為預設，僅將某些下行介面關閉就好。**
    
    ```
    (config)# **interface** *type number*
    (config-if)# **no cdp enable**
    (config-if)# **exit**
    ```
    

## VLAN

- 命名為 `VLAN{number}`
- Lab1 使用 `VLAN101`
- Lab2-1 & Lab2-2 使用 `VLAN102`
- Management 使用 `VLAN30`
- 324 使用 `VLAN324`
- 316 使用 `VLAN316`
- 321 使用 `VLAN321`
    - 請注意他的前面有一台 Hub
- **系計中內的 switch 之間的線路都是 trunk mode**
    - **其他的線路都是 access mode**
- 對於每一條 trunk 只允許**必要的** VLAN
- 如果有調整 Native VLAN 的需求，請做**最小程度**的調整，其餘維持預設 (VLAN 1)

### `CSCC-Lab-Sw1`

```
CSCC-Lab-Sw1# configure terminal

CSCC-Lab-Sw1(config)# vlan 101
CSCC-Lab-Sw1(config-vlan)# name VLAN101
CSCC-Lab-Sw1(config-vlan)# exit

CSCC-Lab-Sw1(config)# vlan 102
CSCC-Lab-Sw1(config-vlan)# name VLAN102
CSCC-Lab-Sw1(config-vlan)# exit

CSCC-Lab-Sw1(config)# interface Fa0/1
CSCC-Lab-Sw1(config-if)# switchport mode access
CSCC-Lab-Sw1(config-if)# switchport access vlan 101
CSCC-Lab-Sw1(config-if)# exit

CSCC-Lab-Sw1(config)# interface Fa0/2
CSCC-Lab-Sw1(config-if)# switchport mode access
CSCC-Lab-Sw1(config-if)# switchport access vlan 102
CSCC-Lab-Sw1(config-if)# exit

CSCC-Lab-Sw1(config)# interface Gi0/1
CSCC-Lab-Sw1(config-if)# switchport trunk allowed vlan 30,101,102
CSCC-Lab-Sw1(config-if)# switchport mode trunk
CSCC-Lab-Sw1(config-if)# exit

CSCC-Lab-SW1(config)# exit
CSCC-Lab-Sw1# write memory
```

### `CSCC-Lab-Sw2`

```
CSCC-Lab-Sw2# configure terminal

CSCC-Lab-Sw2(config)# vlan 30
CSCC-Lab-Sw2(config-vlan)# name VLAN30
CSCC-Lab-Sw2(config-vlan)# exit

CSCC-Lab-Sw2(config)# vlan 102
CSCC-Lab-Sw2(config-vlan)# name VLAN102
CSCC-Lab-Sw2(config-vlan)# exit

CSCC-Lab-Sw2(config)# interface Fa0/1
CSCC-Lab-Sw2(config-if)# switchport mode access
CSCC-Lab-Sw2(config-if)# switchport access vlan 102
CSCC-Lab-Sw2(config-if)# exit

CSCC-Lab-Sw2(config)# interface range Gi0/1 - Gi0/2
CSCC-Lab-Sw2(config-if-range)# switchport trunk allowed vlan 30,102
CSCC-Lab-Sw2(config-if-range)# switchport mode trunk
CSCC-Lab-Sw12(config-if-range)# exit

CSCC-Lab-Sw2(config)# exit
CSCC-Lab-Sw2# write memory
```

### `CSCC-intranet-Sw`

```
CSCC-intranet-Sw# configure terminal

CSCC-intranet-Sw(config)# vlan 102
CSCC-intranet-Sw(config-vlan)# name VLAN102
CSCC-intranet-Sw(config-vlan)# exit

CSCC-intranet-Sw(config)# interface Fa0/1
CSCC-intranet-Sw(config-if)# switchport mode access
CSCC-intranet-Sw(config-if)# switchport access vlan 102
CSCC-intranet-Sw(config-if)# exit

CSCC-intranet-Sw(config)# interface Fa0/2
CSCC-intranet-Sw(config-if)# switchport mode access
CSCC-intranet-Sw(config-if)# switchport access vlan 30
CSCC-intranet-Sw(config-if)# exit

CSCC-intranet-Sw(config)# interface range Gi0/1 - Gi0/2
CSCC-intranet-Sw(config-if-range)# switchport trunk allowed vlan 30,102
CSCC-intranet-Sw(config-if-range)# switchport mode trunk
CSCC-intranet-Sw(config-if-range)# exit

CSCC-intranet-Sw(config)# exit
CSCC-intranet-Sw# write memory
```

### `CSCC-PCRoom-Sw`

```
CSCC-PCRoom-Sw# configure terminal

CSCC-PCRoom-Sw(config)# vlan 316
CSCC-PCRoom-Sw(config-vlan)# name VLAN316
CSCC-PCRoom-Sw(config-vlan)# exit

CSCC-PCRoom-Sw(config)# vlan 321
CSCC-PCRoom-Sw(config-vlan)# name VLAN321
CSCC-PCRoom-Sw(config-vlan)# exit

CSCC-PCRoom-Sw(config)# vlan 324
CSCC-PCRoom-Sw(config-vlan)# name VLAN324
CSCC-PCRoom-Sw(config-vlan)# exit

CSCC-PCRoom-Sw(config)# vlan 30
CSCC-PCRoom-Sw(config-vlan)# name VLAN30
CSCC-PCRoom-Sw(config-vlan)# exit

CSCC-PCRoom-Sw(config)# interface range Fa0/1 - Fa0/2
CSCC-PCRoom-Sw(config-if-range)# switchport mode access
CSCC-PCRoom-Sw(config-if-range)# switchport access vlan 324
CSCC-PCRoom-Sw(config-if-range)# exit

CSCC-PCRoom-Sw(config)# interface range Fa0/11 - Fa0/12
CSCC-PCRoom-Sw(config-if-range)# switchport mode access
CSCC-PCRoom-Sw(config-if-range)# switchport access vlan 316
CSCC-PCRoom-Sw(config-if-range)# exit

CSCC-PCRoom-Sw(config)# interface Fa0/20
CSCC-PCRoom-Sw(config-if)# switchport trunk allowed vlan 30,321
CSCC-PCRoom-Sw(config-if)# switchport trunk native vlan 321
CSCC-PCRoom-Sw(config-if)# switchport mode trunk
CSCC-PCRoom-Sw(config-if)# exit

CSCC-PCRoom-Sw(config)# interface Gi0/1
CSCC-PCRoom-Sw(config-if)# switchport trunk allowed vlan 30,316,321,324
CSCC-PCRoom-Sw(config-if)# switchport mode trunk
CSCC-PCRoom-Sw(config-if)# exit

CSCC-PCRoom-Sw(config)# exit
CSCC-PCRoom-Sw# write memory
```

### `CS-Core`

```
CS-Core# configure terminal

CS-Core(config)# vlan 30
CS-Core(config-vlan)# name VLAN30
CS-Core(config-vlan)# exit

CS-Core(config)# vlan 101
CS-Core(config-vlan)# name VLAN101
CS-Core(config-vlan)# exit

CS-Core(config)# vlan 102
CS-Core(config-vlan)# name VLAN102
CS-Core(config-vlan)# exit

CS-Core(config)# vlan 316
CS-Core(config-vlan)# name VLAN316
CS-Core(config-vlan)# exit

CS-Core(config)# vlan 321
CS-Core(config-vlan)# name VLAN321
CS-Core(config-vlan)# exit

CS-Core(config)# vlan 324
CS-Core(config-vlan)# name VLAN324
CS-Core(config-vlan)# exit

CS-Core(config)# interface Gi1/0/1
CS-Core(config-if)# switchport trunk allowed vlan 30,101,102
CS-Core(config-if)# switchport mode trunk
CS-Core(config-if)# exit

CS-Core(config)# interface Gi1/0/2
CS-Core(config-if)# switchport trunk allowed vlan 30,102
CS-Core(config-if)# switchport mode trunk
CS-Core(config-if)# exit

CS-Core(config)# interface Gi1/0/3
CS-Core(config-if)# switchport trunk allowed vlan 30,102
CS-Core(config-if)# switchport mode trunk
CS-Core(config-if)# exit

CS-Core(config)# interface Gi1/0/4
CS-Core(config-if)# switchport trunk allowed vlan 30,316,321,324
CS-Core(config-if)# switchport mode trunk
CS-Core(config-if)# exit

CS-Core(config)#exit
CS-Core# write memory
```

### `EC321-Sw`

```
EC321-Sw# configure terminal

EC321-Sw(config)# vlan 30
EC321-Sw(config-vlan)# name VLAN30
EC321-Sw(config-vlan)# exit

EC321-Sw(config)# vlan 321
EC321-Sw(config-vlan)# name VLAN321
EC321-Sw(config-vlan)# exit

EC321-Sw(config)# interface Fa0/1
EC321-Sw(config-if)# switchport trunk allowed vlan 30,102
EC321-Sw(config-if)# switchport trunk native vlan 321
EC321-Sw(config-if)# switchport mode trunk
EC321-Sw(config-if)# exit

EC321-Sw(config)# interface Fa0/2
EC321-Sw(config-if)# switchport mode access
EC321-Sw(config-if)# switchport access vlan 321
EC321-Sw(config-if)# exit

EC321-Sw(config)# exit
EC321-Sw# write memory
```

## Switch IP Address & Gateway

| Device | IP Address | Gateway |
| --- | --- | --- |
| CSCC-PCRoom | 140.113.10.10/24 | 140.113.10.254 |
| CSCC-intranet | 140.113.10.11/24 | 140.113.10.254 |
| CSCC-Lab1 | 140.113.10.12/24 | 140.113.10.254 |
| CSCC-Lab2 | 140.113.10.13/24 | 140.113.10.254 |
| EC321-Sw | 140.113.10.14/24 | 140.113.10.254 |
- Switch IP 請設置於 VLAN30
- CS-Core 會擔任每一個 VLAN 的 gateway：
    - VLAN 30: 140.113.10.254/24
    - VLAN 101: 140.113.20.1/27
    - VLAN 102: 140.113.20.33/27
    - VLAN 324: 140.113.24.254/24
    - VLAN 316: 140.113.16.254/24
    - VLAN 321: 140.113.21.254/24

```
# CSCC-PCRoom-Sw
CSCC-PCRoom-Sw# configure terminal
CSCC-PCRoom-Sw(config)# ip default-gateway 140.113.10.254
CSCC-PCRoom-Sw(config)# interface vlan 30
CSCC-PCRoom-Sw(config-if)# ip address 140.113.10.10 255.255.255.0

# CSCC-intranet-Sw
CSCC-intranet-Sw# configure terminal
CSCC-intranet-Sw(config)# ip default-gateway 140.113.10.254
CSCC-intranet-Sw(config)# interface vlan 30
CSCC-intranet-Sw(config-if)# ip address 140.113.10.11 255.255.255.0

# CSCC-Lab1-Sw
CSCC-Lab1-Sw# configure terminal
CSCC-Lab1-Sw(config)# ip default-gateway 140.113.10.254
CSCC-Lab1-Sw(config)# interface vlan 30
CSCC-Lab1-Sw(config-if)# ip address 140.113.10.12 255.255.255.0

# CSCC-Lab2-Sw
CSCC-Lab2-Sw# configure terminal
CSCC-Lab2-Sw(config)# ip default-gateway 140.113.10.254
CSCC-Lab2-Sw(config)# interface vlan 30
CSCC-Lab2-Sw(config-if)# ip address 140.113.10.13 255.255.255.0

# EC321-Sw
EC321-Sw# configure terminal
EC321-Sw(config)# ip default-gateway 140.113.10.254
EC321-Sw(config)# interface vlan 30
EC321-Sw(config-if)# ip address 140.113.10.14 255.255.255.0

# CS-Core
CS-Core# configure terminal

CS-Core(config)# interface vlan 30
CS-Core(config-if)# ip address 140.113.10.254 255.255.255.0
CS-Core(config-if)# exit

CS-Core(config)# interface vlan 101
CS-Core(config-if)# ip address 140.113.20.1 255.255.255.224
CS-Core(config-if)# exit

CS-Core(config)# interface vlan 102
CS-Core(config-if)# ip address 140.113.20.33 255.255.255.224
CS-Core(config-if)# exit

CS-Core(config)# interface vlan 316
CS-Core(config-if)# ip address 140.113.16.254 255.255.255.0
CS-Core(config-if)# exit

CS-Core(config)# interface vlan 321
CS-Core(config-if)# ip address 140.113.21.254 255.255.255.0
CS-Core(config-if)# exit

CS-Core(config)# interface vlan 324
CS-Core(config-if)# ip address 140.113.24.254 255.255.255.0
CS-Core(config-if)# exit
```

### 連通性

- 至此的設定應該使相同 LAN 機器 (PC & Switch) 可以互相 ping 通，而不同 LAN 則不行 (CS-Core routing 尚未啟用)
- 確保 CS-Core、CSCC-Lab2、CSCC-intranet 之間任何一條 link 斷線都不會影響連通性

## STP

- 對於每臺 Switch STP Mode 設定爲 `rapid-pvst`
- Switch 終端 (edge) 連線 Port 設定直接進到 Forwarding，使連線快速啟用，並防止 BPDU 封包進入這些界面，當偵測到，則 **Error Disable 掉該介面**
    - CSCC 內的 Switch 除了往上行 NYCU IT 走之外都算終端設備 (edge)
    - 不要將設定設爲 default
    - 注意，CSCC & 321 之間也需要，但也要確保該線路通暢
- 固定 CS-Core 爲所有 VLAN 的 spanning tree instance 的 root
    - 設爲 root primary
    - 包括 `VLAN 1,30,101,102,316,321,324`

### 連通性

- 可能會使部分連線 Error Disable

```
# On Other Switch
(config)# **spanning-tree mode** rapid-pvst
(config)# **interface** *interface*
(config-if)# **spanning-tree portfast**
(config-if)# **spanning-tree bpduguard enable**

# On CS-Core
(config)# **spanning-tree mode** rapid-pvst
****(config)# **spanning-tree vlan** 1,30,101,102,316,321,324 **root** primary
```

## CS-Core

- 啟用 Routing
- 設定 default route 將流量丟往 140.113.1.1
- 上行介面（Gig1/0/24）
    - L3 interface
    - IP: 140.113.1.2/30
    - Gateway: 140.113.1.1
    - 要發送與接收 LLDP 封包

```
# On CS-Core
CS-Core(config)# **ip routing**
CS-Core(config)# **ip route 0.0.0.0 0.0.0.0 140.113.1.1**
CS-Core(config)# **ip default-gateway 140.113.1.1**
CS-Core(config)# interface Gi 1/0/24
****CS-Core(config-if)# **no switchport**
CS-Core(config-if)# ip address 140.113.1.2 255.255.255.252
CS-Core(config-if)# lldp receive
CS-Core(config-if)# lldp transmit
```

### 連通性

- 除因 STP 部分 Error Disable 連線外，若暫時取消 STP 特定設定，則所有機器 (PC & Switch) 彼此應該都能 ping 通（包括 ping 到 NYCU-IT）