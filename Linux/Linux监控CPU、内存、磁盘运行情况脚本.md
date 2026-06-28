## Linux监控CPU、内存、磁盘运行情况脚本 ##

### 1、监控CPU负载信息 ###

```shell
#!/bin/bash
# check_system_load.sh - 监控CPU负载信息脚本

# --- 配置 ---
THRESHOLD=5  # 固定阈值，或者可以根据核心数动态计算

# --- 获取数据 ---
# 1. 获取 CPU 核心数
CORES=$(nproc)

# 2. 获取负载详情 (直接从 /proc/loadavg 读取，比解析 uptime 更快更准)
# 变量依次代表: 1分钟负载, 5分钟负载, 15分钟负载, 运行队列/总进程数
read LOAD_1 LOAD_5 LOAD_15 RUN_QUEUE TOTAL_PROCS < /proc/loadavg

# --- 输出状态 ---
echo "=== 系统负载监控 ==="
echo "CPU 核心数: $CORES"
echo "当前负载 (1/5/15分钟): $LOAD_1, $LOAD_5, $LOAD_15"

# --- 告警判断 ---
# 使用 bc 进行浮点数比较 (因为负载可能是 5.01 这种小数)
# 逻辑：如果 1分钟负载 > 阈值
if (( $(echo "$LOAD_1 > $THRESHOLD" | bc -l) )); then
    echo "⚠️ $(date '+%Y-%m-%d %H:%M:%S') 警告: 系统负载过高! 当前值: $LOAD_1 (阈值: $THRESHOLD)"
    # 可以添加更多操作，如发送邮件、记录日志等
else
    echo "✅ $(date '+%Y-%m-%d %H:%M:%S') 当前系统负载正常"
fi
```

### 2、监控内存使用情况 ###

```shell
#!/bin/bash
# check_memory.sh - 监控内存使用情况脚本

# --- 1. 获取数据 ---
# 使用 free 命令获取 Mem 行数据
# 变量对应: 总内存, 已用, 空闲, 共享, buff/cache, 可用
read -r _ TOTAL USED FREE SHARED BUFF_CACHE AVAILABLE < <(free -m | awk '/^Mem:/')

# 计算使用率 (保留2位小数)
MEM_PERCENT=$(awk "BEGIN {printf \"%.2f\", $USED/$TOTAL*100}")

# 获取 Swap 使用率 (防止除以0)
SWAP_TOTAL=$(free -m | awk '/^Swap:/ {print $2}')
if [ "$SWAP_TOTAL" -gt 0 ]; then
    SWAP_PERCENT=$(awk "BEGIN {printf \"%.2f\", $(free -m | awk '/^Swap:/ {print $3}')/$SWAP_TOTAL*100}")
else
    SWAP_PERCENT="0.00"
fi

# --- 2. 输出状态 ---
echo "=== 内存监控报告 ==="
echo "✅ $(date '+%Y-%m-%d %H:%M:%S') 物理内存: 总 ${TOTAL}MB / 已用 ${USED}MB / 使用率 ${MEM_PERCENT}%"
echo "✅ $(date '+%Y-%m-%d %H:%M:%S') 交换分区: 使用率 ${SWAP_PERCENT}%"

# --- 3. 告警判断 ---
THRESHOLD=80

# 检查物理内存
if (( $(echo "$MEM_PERCENT > $THRESHOLD" | bc -l) )); then
    echo "⚠️ $(date '+%Y-%m-%d %H:%M:%S')  严重警告: 物理内存使用率过高! (当前: ${MEM_PERCENT}%)"
    # 可以在这里添加邮件发送代码
fi

# 检查 Swap (通常 Swap 高了也需要关注)
if (( $(echo "$SWAP_PERCENT > $THRESHOLD" | bc -l) )); then
    echo "⚠️ $(date '+%Y-%m-%d %H:%M:%S')  警告: Swap 分区使用率过高! (当前: ${SWAP_PERCENT}%)"
fi

```

### 3、监控磁盘使用情况 ###

```shell
#!/bin/bash
# check_disk.sh - 监控磁盘使用情况脚本

echo "=== 检查根分区磁盘使用情况 ==="

# 检查根分区磁盘使用情况
# df -h /|grep -vE '^Filesystem|tmpfs|cdrom'|awk '{print$5 "" $1}'

# 如果使用超过80%，则输出警告
threshold=80

# 使用更健壮的 df 输出格式，避免列位置依赖
used=$(df -h / | awk 'NR>1 && $1 !~ /tmpfs|cdrom/ {gsub(/%/,"",$5); print $5}')


# 检查是否超过阈值
if [ "$used" -gt "$threshold" ]; then
    echo "⚠️ $(date '+%Y-%m-%d %H:%M:%S') 警告: 根分区使用超过 ${threshold}%! 当前使用率: ${used}%"
    # 可以添加更多操作，如发送邮件、记录日志等
else
	echo "✅ $(date '+%Y-%m-%d %H:%M:%S') 当前使用率: ${used}%"
fi
```

### 4、监控I/O情况 ###

```shell
#!/bin/bash
# check_io.sh - 监控I/O情况脚本

# --- 1. 自动检测磁盘名称 ---
# 尝试获取第一个物理磁盘 (sda, vda, nvme0n1 等)
DEVICE=$(lsblk -nd -o NAME | grep -E '^sd|^vd|^nvme' | head -n 1)

if [ -z "$DEVICE" ]; then
    echo "❌ 错误: 未找到磁盘设备"
    exit 1
fi

echo "=== 磁盘 I/O 监控 ($DEVICE) ==="

# --- 2. 定义获取数据的函数 ---
get_disk_stats() {
    local dev=$1
    # /proc/diskstats 字段说明 (内核 2.6+):
    # $3: 设备名
    # $4: 读取完成次数 (Reads completed)
    # $5: 读取扇区数 (Sectors read)
    # $8: 写入完成次数 (Writes completed)
    # $9: 写入扇区数 (Sectors written)
    # 注意：扇区通常是 512 字节，即 0.5KB
    grep " $dev " /proc/diskstats | awk '{print $4, $5, $8, $9}'
}

# --- 3. 第一次采样 (基准值) ---
read START_VALS < <(get_disk_stats "$DEVICE")
r1=$(echo $START_VALS | awk '{print $1}')
s1=$(echo $START_VALS | awk '{print $2}')
w1=$(echo $START_VALS | awk '{print $3}')
sw1=$(echo $START_VALS | awk '{print $4}')

# 等待 1 秒
sleep 1

# --- 4. 第二次采样 (当前值) ---
read END_VALS < <(get_disk_stats "$DEVICE")
r2=$(echo $END_VALS | awk '{print $1}')
s2=$(echo $END_VALS | awk '{print $2}')
w2=$(echo $END_VALS | awk '{print $3}')
sw2=$(echo $END_VALS | awk '{print $4}')

# --- 5. 计算差值 (每秒速率) ---
# 计算 IOPS (次数差)
read_iops=$((r2 - r1))
write_iops=$((w2 - w1))

# 计算吞吐量 (扇区差 * 512字节 / 1024 = KB)
# 这里的 512 是标准扇区大小
read_kbs=$(( (s2 - s1) * 512 / 1024 ))
write_kbs=$(( (sw2 - sw1) * 512 / 1024 ))

# --- 6. 输出结果 ---
echo "----------------------------------------"
echo "读取速度: ${read_kbs} KB/s  (IOPS: $read_iops)"
echo "写入速度: ${write_kbs} KB/s  (IOPS: $write_iops)"
echo "----------------------------------------"

# --- 7. 简单的阈值告警 ---
# 这里设定如果 读写IOPS 总和超过 500 则告警 (你可以根据实际情况修改)
TOTAL_IOPS=$((read_iops + write_iops))
if [ $TOTAL_IOPS -gt 500 ]; then
    echo "⚠️ $(date '+%Y-%m-%d %H:%M:%S') 警告: 磁盘负载较高! (总 IOPS: $TOTAL_IOPS)"
else
    echo "✅ $(date '+%Y-%m-%d %H:%M:%S') 磁盘负载正常"
fi

```

### 5、服务器端口情况 ###

> 服务器端口安全是防御的第一道门，定期自查比事后补救成本低100倍

```shell
#!/bin/bash
# check_port.sh - 端口巡检脚本

set -euo pipefail

# 颜色定义
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m'

# 白名单端口（按需增减）
WHITELIST="22|80|443|3306"

# 定义列名和宽度（便于统一调整）
COL1="ADDRESS"
COL2="PROCESS"
COL3="PID"
WIDTH1=21
WIDTH2=15
WIDTH3=8

# 打印表头（带颜色）
print_header() {
    printf "${YELLOW}%-${WIDTH1}s %-${WIDTH2}s %-${WIDTH3}s${NC}\n" "$COL1" "$COL2" "$COL3"
}

echo -e "${GREEN}=== 端口巡检 $(date '+%Y-%m-%d %H:%M:%S') ===${NC}\n"

# 1. 显示所有监听端口
echo -e "${YELLOW}【监听端口】${NC}"
print_header
ss -tlnp | awk 'NR>1 {
    addr = $4
    if (match($6, /users:\(\("([^"]+)",pid=([0-9]+)/, arr)) {
        name = arr[1]
        pid = arr[2]
    } else {
        name = "unknown"
        pid = "-"
    }
    printf "%-21s %-15s %-8s\n", addr, name, pid
}' | sort -t: -k2 -n

echo

# 2. 异常端口检查（排除白名单）
echo -e "${YELLOW}【异常端口检查】${NC}"
result=$(ss -tlnp | awk -v wl=":$WHITELIST$" 'NR>1 && $4 !~ wl {
    addr = $4
    if (match($6, /users:\(\("([^"]+)",pid=([0-9]+)/, arr)) {
        name = arr[1]
        pid = arr[2]
    } else {
        name = "unknown"
        pid = "-"
    }
    printf "%-21s %-15s %-8s\n", addr, name, pid
}' | sort -t: -k2 -n)

if [ -z "$result" ]; then
    echo -e "${GREEN}未发现非标准端口监听。${NC}"
else
    print_header
    echo "$result"
fi

echo -e "\n${GREEN}巡检完成。${NC}"

```

> 发现可疑端口后的标准动作：

```shell
# 1. 确认进程归属
systemctl status <进程名>

# 2. 如果确认不需要，停掉服务
systemctl stop <服务名>
systemctl disable <服务名>

# 3. 防火墙层面封堵（双保险）
iptables -A INPUT -p tcp --dport <端口号> -j DROP

# 4. 如果是必须对外开放的服务，限制IP白名单
iptables -A INPUT -p tcp --dport <端口号> -s <信任IP> -j ACCEPT
iptables -A INPUT -p tcp --dport <端口号> -j DROP
```