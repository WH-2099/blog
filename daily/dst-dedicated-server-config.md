---
description: Let's Starve Together
---

# 饥荒专用服务器配置文件

## 配置文件总览

```
Cluster_1  # 以集群方式提供服务，地面和洞穴是两个独立的服务器进程
├── cluster.ini  # 集群配置
├── cluster_token.txt  # 集群认证码
├── adminlist.txt  # 管理员名单，克雷的 ID（KU_XXXXXXXX），一行一个
├── blocklist.txt  # 封禁名单，SteamID64，一行一个
├── whitelist.txt  # 特权名单，克雷的 ID（KU_XXXXXXXX），一行一个
├── Master  # 主服务器进程（地面）
│   ├── server.ini  # 服务器配置
│   ├── modoverrides.lua  # mod 的设置
│   ├── leveldataoverride.lua  # 世界（地形、生物等）的设置
│   └── save  # 存档
│       └── ...
├── Caves  # 洞穴服务器
│   ├── server.ini  # 服务器配置
│   ├── modoverrides.lua  # mod 的设置
│   ├── leveldataoverride.lua  # 世界（地形、生物等）的设置
│   └── save  # 存档
│       └── ...
└── mods  # mod 相关
    └── dedicated_server_mods_setup.lua  # mod 加载设定
```

{% hint style="info" %}
可以在本地先利用游戏的界面创建并配置游戏服务器，然后将对应生成的文件拷贝为专用服务器配置。
{% endhint %}

## 核心配置文件

### cluster.ini

{% code title="cluster.ini" %}
```
[MISC]
; 要保留的最大快照数量。
; 这些快照在每次保存时都会被创建，并在 "主机游戏 "屏幕的 "回滚 "标签中可用。
; max_snapshots = 6

; 允许在服务器运行的命令提示符或终端中输入 lua 命令。
; console_enabled = true


[SHARD]
; 启用服务器分片。
; 对于多级服务器，这必须被设置为 "true"。
; 对于单级服务器，它可以被省略。
; shard_enabled = false

; 这是主服务器将监听的网络地址，供其他分片服务器连接使用。
; 如果你的集群中的所有服务器都在同一台机器上，则将其设置为 127.0.0.1 ；
; 如果你的集群中的服务器在不同的机器上，则设置为 0.0.0.0 。
; 这只需要为主服务器设置，可以在 cluster.ini 或主服务器的 server.ini 中设置。
; 可在 server.ini 中重写
; bind_ip = 127.0.0.1

; 非主控分片在试图连接到主控分片时将使用这个 IP 地址。
; 如果集群中的所有服务器都在同一台机器上，将其设置为 127.0.0.1 。
; 可在 server.ini 中重写
; master_ip =


; 这是主服务器将监听的 UDP 端口，非主分片在试图连接到主分片时将使用这个端口。
; 这应该通过在 cluster.ini 中的条目为所有分片设置相同的值，或者完全省略以使用默认值。
; 这必须与运行在与主控分片相同机器上的任何分片上的 server_port 设置不同。
; 可在 server.ini 中重写
; master_port = 10888

; 这是一个用于验证从属服务器与主服务器的密码。
; 如果你在不同的机器上运行需要相互连接的服务器，这个值在每台机器上必须是相同的。
; 对于在同一台机器上运行的服务器，你可以只在 cluster.ini 中设置。
; 可在 server.ini 中重写
; cluster_key =


[STEAM]
; 当设置为 "true "时，服务器将只允许属于 steam_group_id 设置中指定的 steam 组的玩家连接。
; steam_group_only = false

; steam_group_only / steam_group_admins 设置相关的 steam 组 ID。
; steam_group_id = 0

; 当这个设置为 "true "时，在 steam_group_id 中指定的 steam 组的管理员也将在服务器上拥有管理员身份。
; steam_group_admins = false


[NETWORK]
; 创建一个离线集群。
; 该服务器不会被公开列出，只有本地网络上的玩家能够加入，任何与 steam 有关的功能都会失效。
; offline_cluster = false

; 这是服务器每秒钟向客户提供更新数据的次数。
; 增加这个次数可以提高精度，但会消耗更大的网络带宽。
; 建议将其保持在默认值 15 。
; 建议你只在局域网游戏中改变这个选项，并使用一个能被 60 除以的数字（15、20、30）。
; tick_rate = 15

; 为白名单上的玩家保留的空位数量。
; 要将一个玩家列入白名单，请将他们的 Klei UserId 添加到 whiteelist.txt 文件中（将此文件与 cluster.ini 放在同一个目录中）。
; 仅可用于主分片的 cluster.ini
; whitelist_slots = 0

; 这是玩家加入你的服务器时必须输入的密码。
; 留空或省略它表示没有密码。
; 仅可用于主分片的 cluster.ini
; cluster_password =

; 你的服务器集群的名称。
; 这是将显示在服务器列表中的名称。
; 仅可用于主分片的 cluster.ini
; cluster_name =

; 集群描述。
; 这将显示在 "浏览游戏 "界面中的服务器信息区域。
; 仅可用于主分片的 cluster.ini
; cluster_description =

; 当设置为 "true "时，服务器将只接受来自同一局域网内机器的连接。
; 仅可用于主分片的 cluster.ini
; lan_only_cluster = false

; 集群的游戏风格。
; 这个字段相当于 "创建游戏"界面中的 "服务器游戏风格 "字段。
; 仅可用于主分片的 cluster.ini
; 有效值如下（不包含括号及括号中内容）：
; social  （社交）
; cooperative  （合作）
; competitive  （竞争）
; madness  （疯狂）
; cluster_intention =

; 当这个选项被设置为 false 时，游戏将不再在每天结束时自动保存。
; 游戏仍然会在关机时保存，并且可以使用 c_save() 手动保存。
; autosaver_enabled = true

; 集群的语言
; 中文 zh
; cluster_language = en

[GAMEPLAY]
; 可以同时连接到集群的最大玩家数量。
; 仅可用于主分片的 cluster.ini
; max_players = 16

; 玩家间战斗（队友伤害）
; pvp = false

; 集群的游戏模式。
; 这个字段相当于 "创建游戏 "界面中的 "游戏模式 "字段。
; 有效值如下（不包含括号及括号中内容）：
; survival  （生存）
; endless  （无尽）
; wilderness  （荒野）
; game_mode = survival

; 当没有玩家连接时，暂停服务器。
; pause_when_empty = false

; 设置为 "true"，以启用投票功能。
; vote_enabled = true
```
{% endcode %}

### server.ini

{% code title="server.ini" %}
```
[SHARD]
; 设置一个分片为集群的主分片。
; 每个集群必须有一个主服务器。
; 在主服务器的 server.ini 中设置为 true ;
; 在其他所有的 server.ini 中设置为 false 。
; is_master =

; 这是将在日志文件中显示的分片名称。
; 它对于主服务器来说是被忽略的，主服务器的名字总是叫 [SHDMASTER] 。
; name =

; 这个字段是为非主服务器自动生成的，并在内部用来唯一地识别一个服务器。
; 如果任何人的角色目前处于在这个服务器所管理的世界中，改变这个字段或删除它可能会产生问题。
; id =


[STEAM]
; steam 使用的内部端口。
; 请确保你在同一台机器上运行的每台服务器都是不同的。
; authentication_port = 8766

; steam 使用的内部端口。
; 请确保你在同一台机器上运行的每台服务器都是不同的。
; master_server_port = 27016


[NETWORK]
; 该服务器将监听连接的 UDP 端口。
; 如果你正在运行一个多级集群，对同一台机器上的多个服务器，这个端口各不相同。
; 这个端口必须在 10998 和 11018 之间（含），以便同一局域网的玩家在他们的服务器列表中看到它。
; 在某些操作系统上，低于 1024 的端口限制为只能特权用户使用。
; server_port = 10999
```
{% endcode %}

## 参考源

[Dedicated Server Settings Guide](https://forums.kleientertainment.com/forums/topic/64552-dedicated-server-settings-guide/)
