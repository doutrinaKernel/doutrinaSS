#!/system/bin/sh

echo "=============================="
echo " ANDROID ROOT FULL SCAN "
echo "=============================="
echo ""

ROOT_FOUND=0

echo "[1] Procurando binario SU..."
for p in /system/bin/su /system/xbin/su /vendor/bin/su /sbin/su /su/bin/su; do
    if [ -f "$p" ]; then
        echo "ROOT DETECTADO -> binario su em $p"
        ROOT_FOUND=1
    fi
done

echo ""
echo "[2] Procurando arquivos do Magisk..."
for p in /sbin/magisk /sbin/.magisk /data/adb/magisk /data/adb/modules /cache/magisk.log; do
    if [ -e "$p" ]; then
        echo "MAGISK DETECTADO -> $p"
        ROOT_FOUND=1
    fi
done

echo ""
echo "[3] Procurando KernelSU..."
for p in /data/adb/ksu /data/adb/ksud /data/adb/modules/ksu; do
    if [ -e "$p" ]; then
        echo "KERNELSU DETECTADO -> $p"
        ROOT_FOUND=1
    fi
done

echo ""
echo "[4] Procurando apps de root..."
pm list packages | grep -Ei "magisk|kernelsu|supersu|kingroot|superuser" && ROOT_FOUND=1

echo ""
echo "[5] Verificando processos ativos..."
ps -A | grep -Ei "magisk|ksud|su" && ROOT_FOUND=1

echo ""
echo "[6] Verificando overlay de root..."
mount | grep overlay && echo "Overlay suspeito detectado"

echo ""
echo "[7] Verificando propriedades de boot..."
echo "verifiedbootstate: $(getprop ro.boot.verifiedbootstate)"
echo "flash.locked: $(getprop ro.boot.flash.locked)"
echo "veritymode: $(getprop ro.boot.veritymode)"

echo ""
echo "[8] Checando partições BOOT..."
ls -l /dev/block/by-name/boot 2>/dev/null
ls -l /dev/block/by-name/init_boot 2>/dev/null

echo ""
echo "[9] Informações do kernel..."
uname -a
cat /proc/version

echo ""
if [ $ROOT_FOUND -eq 1 ]; then
    echo "=============================="
    echo " ROOT OU MODULO DE ROOT DETECTADO "
    echo "=============================="
else
    echo "=============================="
    echo " NENHUM ROOT DETECTADO "
    echo "=============================="
fi
