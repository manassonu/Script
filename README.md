#!/bin/bash

# Function to calculate CPU usage percentage
get_cpu_usage() {
    # Get idle cpu percentage using top and calculate usage
    CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | awk '{print $8}')
    CPU_USAGE=$(echo "100 - $CPU_IDLE" | bc)
    # Truncate decimal
    printf "%.0f" "$CPU_USAGE"
}

# Function to get memory usage percentage
get_mem_usage() {
    MEM_TOTAL=$(free | awk '/Mem:/ { print $2 }')
    MEM_USED=$(free | awk '/Mem:/ { print $3 }')
    MEM_USAGE=$(echo "$MEM_USED * 100 / $MEM_TOTAL" | bc)
    printf "%.0f" "$MEM_USAGE"
}

# Function to get disk usage percentage for root filesystem
get_disk_usage() {
    DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    echo "$DISK_USAGE"
}

CPU_USAGE=$(get_cpu_usage)
MEM_USAGE=$(get_mem_usage)
DISK_USAGE=$(get_disk_usage)

HEALTHY=true
REASON=""

if [ "$CPU_USAGE" -gt 60 ] || [ "$MEM_USAGE" -gt 60 ] || [ "$DISK_USAGE" -gt 60 ]; then
    HEALTHY=false
fi

if [ "$HEALTHY" = true ]; then
    STATUS="Healthy"
else
    STATUS="Unhealthy"
fi

EXPLAIN=false
if [ "$1" = "explain" ] || [ "$1" = "--explain" ]; then
    EXPLAIN=true
fi

echo "VM Health Status: $STATUS"

if [ "$EXPLAIN" = true ]; then
    if [ "$HEALTHY" = false ]; then
        REASON="Reasons for Unhealthy status:"
        [ "$CPU_USAGE" -gt 60 ] && REASON="$REASON\n- CPU usage is high: $CPU_USAGE%"
        [ "$MEM_USAGE" -gt 60 ] && REASON="$REASON\n- Memory usage is high: $MEM_USAGE%"
        [ "$DISK_USAGE" -gt 60 ] && REASON="$REASON\n- Disk usage is high: $DISK_USAGE%"
    else
        REASON="All monitored resources are within healthy limits:\n- CPU usage: $CPU_USAGE%\n- Memory usage: $MEM_USAGE%\n- Disk usage: $DISK_USAGE%"
    fi
    echo -e "$REASON"
fi

exit 0
