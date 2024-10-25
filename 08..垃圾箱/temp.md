李凌秋 23270519
脚本运行结果
![[Pasted image 20241025081043.png]]

```
# monitor.sh
#!/bin/bash
LOG_DIR="/home/test/monlog"
DATE=$(date +%Y-%m-%d_%H:%M:%S)
IFTOP_LOG="$LOG_DIR/iftop_$DATE.log"
NMON_LOG="$LOG_DIR/nmon_$DATE.nmon"
REPORT_LOG="$LOG_DIR/report_$DATE.log"

mkdir -p $LOG_DIR

echo "Running iftop..."
sudo iftop -t -s 10 > $IFTOP_LOG 2>&1
echo -e "iftop log saved to $IFTOP_LOG\n"

echo "Running nmon..."
sudo nmon -f $NMON_LOG -s 10 -c 1
sleep 12
echo -e "nmon log saved to $NMON_LOG\n"

echo "Extracting key performance metrics..." # 提取关键性能指标

# 提取 iftop 的总量
TOTAL_SEND_RATE=$(grep "Total send rate" $IFTOP_LOG | awk '{print $4}')
TOTAL_RECV_RATE=$(grep "Total receive rate" $IFTOP_LOG | awk '{print $4}')

# 提取 nmon 的 CPU 使用率
CPU_USER=$(grep "CPU_ALL,T" $NMON_LOG | awk -F',' '{print $3}')
CPU_SYS=$(grep "CPU_ALL,T" $NMON_LOG | awk -F',' '{print $4}')
CPU_WAIT=$(grep "CPU_ALL,T" $NMON_LOG | awk -F',' '{print $5}')
CPU_IDLE=$(grep "CPU_ALL,T" $NMON_LOG | awk -F',' '{print $6}')

echo "------------------------------------------------------------" >> $REPORT_LOG
echo "Network Traffic:" >> $REPORT_LOG
echo "Total Send Rate: $TOTAL_SEND_RATE" >> $REPORT_LOG
echo "Total Receive Rate: $TOTAL_RECV_RATE" >> $REPORT_LOG
echo "" >> $REPORT_LOG
echo "CPU Usage:" >> $REPORT_LOG
echo "User: $CPU_USER%" >> $REPORT_LOG
echo "System: $CPU_SYS%" >> $REPORT_LOG
echo "Wait: $CPU_WAIT%" >> $REPORT_LOG
echo "Idle: $CPU_IDLE%" >> $REPORT_LOG

cat $REPORT_LOG
echo "Report saved to $REPORT_LOG"
echo "Performance monitoring completed at $(date)."
```