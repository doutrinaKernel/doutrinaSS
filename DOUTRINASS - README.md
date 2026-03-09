#!/system/bin/sh

clear

GREEN="\033[1;32m"
RED="\033[1;31m"
NC="\033[0m"

score=0
checks=0

line() {
  printf "${GREEN}══════════════════════════════════════════════════════════════${NC}\n"
}

check() {
  checks=$((checks+1))
  printf "${GREEN}%-50s${NC}" "$1"
}

ok() {
  printf "${GREEN}✔ LIMPO${NC}\n"
}

sus() {
  printf "${RED}✘ SUSPEITO${NC}\n"
  score=$((score+1))
}

crit() {
  printf "${RED}✘ CRÍTICO${NC}\n"
  score=$((score+3))
}

################################
# CABEÇALHO
################################

line
printf "${GREEN}D O U T R I N A   F O R E N S E\n${NC}"
printf "${GREEN}Scanner de Segurança Android\n${NC}"
line

################################
# BINÁRIOS
################################

for f in \
/system/bin/su /system/xbin/su /sbin/su \
/system/bin/magisk /system/xbin/magisk \
/system/bin/ksu /data/adb/ksu \
/system/bin/apatch /data/adb/apatch/apd \
/system/bin/daemonsu \
/system/app/Superuser.apk /system/app/SuperSU.apk
do
  check "Binário $f"
  [ -f "$f" ] && crit || ok
done

################################
# DIRETÓRIOS
################################

for d in \
/data/adb /data/adb/magisk /data/adb/modules \
/data/adb/service.d /data/adb/post-fs-data.d \
/data/adb/ksu /data/adb/susfs /data/adb/apatch \
/data/adb/zygisk /data/adb/lspd \
/sbin/.magisk /debug_ramdisk/.magisk
do
  check "Diretório $d"
  [ -d "$d" ] && sus || ok
done

################################
# PROCESSOS
################################

for p in magisk magiskd zygisk ksu ksud apatch daemonsu frida lsposed
do
  check "Processo $p"
  ps -A 2>/dev/null | grep -w "$p" | grep -v grep >/dev/null && sus || ok
done

################################
# LEAKS IMPORTANTES
################################

check "Socket Magisk (/dev/socket/magisk*)"
ls /dev/socket/magisk* >/dev/null 2>&1 && crit || ok

################################
# PROPRIEDADES
################################

check "Build test-keys"
getprop ro.build.tags | grep -q test-keys && crit || ok

check "Debuggable"
getprop ro.debuggable | grep -q 1 && sus || ok

check "ADB root"
getprop service.adb.root | grep -q 1 && sus || ok

################################
# BOOTLOADER E VERIFIED
################################

check "Bootloader"
getprop ro.boot.flash.locked | grep -q 0 && crit || ok

check "Verified Boot"
getprop ro.boot.verifiedbootstate | grep -q green && ok || sus

check "dm-verity"
getprop ro.boot.veritymode | grep -q enforcing && ok || sus

check "SELinux"
getenforce | grep -q Enforcing && ok || crit

################################
# MOUNTS
################################

check "/system RW"
mount | grep " /system " | grep rw >/dev/null && crit || ok

check "Magisk traces in mount"
mount | grep -qi magisk && sus || ok

################################
# FRIDA E EMULADOR
################################

check "Frida"
ps -A | grep -qi frida && crit || ok

check "Emulador"
getprop ro.kernel.qemu | grep -q 1 && sus || ok

################################
# ADB WIFI
################################

check "ADB WiFi 5555"
getprop service.adb.tcp.port | grep -q 5555 && sus || ok

################################
# RESULTADO
################################

line

if [ "$score" -gt 10 ]; then
  printf "${RED}DISPOSITIVO COMPROMETIDO (%s pontos)${NC}\n" "$score"
elif [ "$score" -gt 3 ]; then
  printf "${RED}ALTERAÇÕES DETECTADAS (%s)${NC}\n" "$score"
else
  printf "${GREEN}PARECE LIMPO (%s)${NC}\n" "$score"
fi

printf "${GREEN}Verificações: %s${NC}\n" "$checks"

line

printf "${GREEN}"
cat << "EOF"
██████╗  ██████╗ ██╗   ██╗████████╗██████╗ ██╗███╗   ██╗ █████╗
██╔══██╗██╔═══██╗██║   ██║╚══██╔══╝██╔══██╗██║████╗  ██║██╔══██╗
██║  ██║██║   ██║██║   ██║   ██║   ██████╔╝██║██╔██╗ ██║███████║
██║  ██║██║   ██║██║   ██║   ██║   ██╔══██╗██║██║╚██╗██║██╔══██║
██████╔╝╚██████╔╝╚██████╔╝   ██║   ██║  ██║██║██║ ╚████║██║  ██║
╚═════╝  ╚═════╝  ╚═════╝    ╚═╝   ╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝
EOF

printf "DOUTRINA FORENSE${NC}\n"
line
