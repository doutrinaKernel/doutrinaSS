#!/bin/bash

clear

GREEN="\033[1;32m"
RED="\033[1;31m"
YELLOW="\033[1;33m"
RESET="\033[0m"

score=0

check(){
printf "%-50s" "$1"
sleep 0.2
}

det(){
echo -e "${RED}DETECTADO${RESET}"
score=$((score+1))
}

ok(){
echo -e "${GREEN}OK${RESET}"
}

echo -e "${GREEN}"
echo "======================================================"
echo "        ANDROID ROOT / BOOT / BUGREPORT SCANNER"
echo "======================================================"
echo -e "${RESET}"

sleep 1

# -------- SYSTEM CHECKS --------

check "Checking su binary"
command -v su >/dev/null && det || ok

check "Checking busybox"
command -v busybox >/dev/null && det || ok

check "Searching Magisk files"
[ -d /data/adb ] || [ -f /sbin/magiskinit ] && det || ok

check "KernelSU module"
ls /sys/module 2>/dev/null | grep -i ksu >/dev/null && det || ok

check "OverlayFS mount"
mount | grep overlay >/dev/null && det || ok

check "Frida process"
ps -A | grep frida | grep -v grep >/dev/null && det || ok

check "Xposed / LSPosed modules"
[ -d /data/adb/modules ] && ls /data/adb/modules | grep -iE "xposed|lsposed|riru" >/dev/null && det || ok

check "Kernel symbols"
cat /proc/kallsyms 2>/dev/null | grep -iE "ksu|magisk" >/dev/null && det || ok

check "Kernel modules"
cat /proc/modules 2>/dev/null | grep -iE "ksu|magisk" >/dev/null && det || ok

# -------- BOOT IMAGE CHECK --------

BOOTIMG="boot.img"

if [ -f "$BOOTIMG" ]; then

echo ""
echo "---------- BOOT IMAGE ANALYSIS ----------"

check "Scanning boot.img for Magisk"
strings "$BOOTIMG" | grep -i magisk >/dev/null && det || ok

check "Scanning boot.img for Zygisk"
strings "$BOOTIMG" | grep -i zygisk >/dev/null && det || ok

check "Scanning boot.img for KernelSU"
strings "$BOOTIMG" | grep -i ksu >/dev/null && det || ok

check "Scanning boot.img for Frida"
strings "$BOOTIMG" | grep -i frida >/dev/null && det || ok

check "Scanning boot.img for Xposed"
strings "$BOOTIMG" | grep -i xposed >/dev/null && det || ok

fi

# -------- BUGREPORT CHECK --------

BUG="bugreport.txt"

if [ -f "$BUG" ]; then

echo ""
echo "---------- BUGREPORT ANALYSIS ----------"

check "Bugreport: magisk traces"
grep -i magisk "$BUG" >/dev/null && det || ok

check "Bugreport: frida traces"
grep -i frida "$BUG" >/dev/null && det || ok

check "Bugreport: xposed traces"
grep -i xposed "$BUG" >/dev/null && det || ok

check "Bugreport: su execution"
grep -i " su " "$BUG" >/dev/null && det || ok

check "Bugreport: overlayfs"
grep -i overlay "$BUG" >/dev/null && det || ok

fi

echo ""
echo "======================================================"

if [ "$score" -ge 6 ]; then
echo -e "${RED}ROOT VERY LIKELY ($score indicators)${RESET}"
elif [ "$score" -ge 3 ]; then
echo -e "${YELLOW}POSSIBLE ROOT ($score indicators)${RESET}"
else
echo -e "${GREEN}LOW ROOT INDICATION ($score indicators)${RESET}"
fi

echo "======================================================"
echo ""

# -------- FINAL SIGNATURE --------

echo -e "${GREEN}"
echo "██████╗  ██████╗ ██╗   ██╗████████╗██████╗ ██╗███╗   ██╗ █████╗ "
echo "██╔══██╗██╔═══██╗██║   ██║╚══██╔══╝██╔══██╗██║████╗  ██║██╔══██╗"
echo "██║  ██║██║   ██║██║   ██║   ██║   ██████╔╝██║██╔██╗ ██║███████║"
echo "██║  ██║██║   ██║██║   ██║   ██║   ██╔══██╗██║██║╚██╗██║██╔══██║"
echo "██████╔╝╚██████╔╝╚██████╔╝   ██║   ██║  ██║██║██║ ╚████║██║  ██║"
echo "╚═════╝  ╚═════╝  ╚═════╝    ╚═╝   ╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝"
echo ""
echo "        BY: DOUTRINA SCREEN SHARE"
echo -e "${RESET}"
