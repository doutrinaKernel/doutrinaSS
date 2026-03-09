#!/system/bin/sh

clear

BLUE="\033[1;34m"
GREEN="\033[1;32m"
CYAN="\033[1;36m"
NC="\033[0m"

score=0

line(){
printf "${BLUE}════════════════════════════════════════════════════${NC}\n"
}

check(){
printf "${CYAN}%-40s${NC}" "$1"
sleep 0.3
}

ok(){
printf "${GREEN} ✔ NORMAL${NC}\n"
}

det(){
printf "${BLUE} ✘ ANORMAL${NC}\n"
score=$((score+1))
}

scan(){
for i in 10 20 30 40 50 60 70 80 90 100
do
printf "\r${CYAN}Scanning system modules [%s%%]${NC}" "$i"
sleep 0.07
done
printf "\n"
}

# HEADER
line
printf "${GREEN}        ANDROID ROOT / BOOT / BUGREPORT SCANNER\n"
line

scan

line

# SU BINARY
check "Checking su binary"
if [ -f /system/bin/su ] || [ -f /system/xbin/su ]; then
det
else
ok
fi

# BUSYBOX
check "Checking busybox"
command -v busybox >/dev/null 2>&1 && det || ok

# MAGISK
check "Searching Magisk files"
[ -d /data/adb ] && det || ok

# KERNELSU
check "KernelSU module"
ls /sys/module 2>/dev/null | grep -i ksu >/dev/null && det || ok

# OVERLAYFS
check "OverlayFS mount"
mount | grep overlay >/dev/null && det || ok

# FRIDA
check "Frida process"
ps -A 2>/dev/null | grep frida >/dev/null && det || ok

# XPOSED / LSPOSED
check "Xposed / LSPosed modules"
ps -A 2>/dev/null | grep -iE "xposed|lsposed" >/dev/null && det || ok

# KERNEL SYMBOLS
check "Kernel symbols"
cat /proc/kallsyms 2>/dev/null | grep -i su >/dev/null && det || ok

# KERNEL MODULES
check "Kernel modules"
cat /proc/modules 2>/dev/null | grep -i su >/dev/null && det || ok

line

# RESULTADO
if [ "$score" -ge 5 ]; then
printf "${BLUE}HIGH ROOT INDICATION (%s indicators)${NC}\n" "$score"
elif [ "$score" -ge 2 ]; then
printf "${CYAN}MEDIUM ROOT INDICATION (%s indicators)${NC}\n" "$score"
else
printf "${GREEN}LOW ROOT INDICATION (%s indicators)${NC}\n" "$score"
fi

line

# ASCII DOUTRINA
printf "${GREEN}\n"
printf "██████╗  ██████╗ ██╗   ██╗████████╗██████╗ ██╗███╗   ██╗ █████╗\n"
printf "██╔══██╗██╔═══██╗██║   ██║╚══██╔══╝██╔══██╗██║████╗  ██║██╔══██╗\n"
printf "██║  ██║██║   ██║██║   ██║   ██║   ██████╔╝██║██╔██╗ ██║███████║\n"
printf "██║  ██║██║   ██║██║   ██║   ██║   ██╔══██╗██║██║╚██╗██║██╔══██║\n"
printf "██████╔╝╚██████╔╝╚██████╔╝   ██║   ██║  ██║██║██║ ╚████║██║  ██║\n"
printf "╚═════╝  ╚═════╝  ╚═════╝    ╚═╝   ╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝\n"
printf "\n        SCREEN SHARE TOOL\n"
printf "${NC}"

line
