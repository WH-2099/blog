---
description: 脚本写着爽，DEBUG 火葬场
---

# Bash DEBUG 一行搞定

```bash
set -Eeuxo pipefail
```

* `-E` 让 `trap` 能够捕获函数内信号。
* `-e` 非零返回值终止执行。
* `-u` 读取不存在的变量会报错并停止执行。
* `-x` 执行前打印指令内容。
* `-o pipefail` 子命令失败将使管道返回非零并终止执行。
