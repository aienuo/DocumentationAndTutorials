## Linux监控CPU、内存、磁盘运行情况脚本 ##

### 1、监控CPU负载信息 ###

```shell
#!/bin/bash

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