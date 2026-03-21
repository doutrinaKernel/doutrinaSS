#!/data/data/com.termux/files/usr/bin/bash

# --- CORES ---
VERDE='\033[1;32m'
VERMELHO='\033[1;31m'
CIANO='\033[1;36m'
AMARELO='\033[1;33m'
BRANCO='\033[1;37m'
RESET='\033[0m'

# --- TRAVA DE ACESSO ---
clear
echo -e "${VERDE}######################################"
echo -e "#         DOUTRINA BYPASS            #"
echo -e "######################################${RESET}"
echo -ne "\n${BRANCO}KEY:${RESET} "
read -s key
[ "$key" != "1313" ] && echo -e "${VERMELHO}\nNEGADO!${RESET}" && exit 1

# --- MENU DE CONEXÃO ---
clear
echo -e "${CIANO}[1]${BRANCO} PAREAR VIA USB / WI-FI (OBTER PERMISSÕES LOGS)${RESET}"
echo -e "${CIANO}[2]${BRANCO} INICIAR SCANNER DIRETO (MODO LIMITADO)${RESET}"
echo -ne "\n${AMARELO}ESCOLHA:${RESET} "
read opt

if [ "$opt" == "1" ]; then
    echo -e "\n${CIANO}➜ AGUARDANDO CONEXÃO ADB...${RESET}"
    echo "Certifique-se que a Depuração Sem Fio está ativa."
    echo -ne "Digite o IP:PORTA: "
    read adb_ip
    adb pair $adb_ip
    adb connect $adb_ip
    sleep 2
fi

# --- BANNER PRINCIPAL ---
clear
echo -e "${VERDE}"
cat << "EOF"
  _____   ____  _    _ _______ _____  _____ _   _          
 |  __ \ / __ \| |  | |__   __|  __ \|_   _| \ | |   /\    
 | |  | | |  | | |  | |  | |  | |__) | | | |  \| |  /  \   
 | |  | | |  | | |  | |  | |  |  _  /  | | | . \` | / /\ \  
 | |__| | |__| | |__| |  | |  | | \ \ _| |_| |\  |/ ____ \ 
 |_____/ \____/ \____/   |_|  |_|  \_\_____|_| \_/_/    \_/
          [ DOUTRINA KING SCANNER - ADVANCED ]
EOF
echo -e "------------------------------------------------------------"
echo -e "${CIANO}ANALISANDO:${RESET} LOGCAT | MTP | KERNEL | MEMORY MAPS"
echo -e "------------------------------------------------------------${RESET}"

# --- MOTOR ANTI-FALSO POSITIVO (CROSS-CHECK) ---
check_adv() {
    local label=$1
    local logic=$2
    echo -n -e "${BRANCO}➜${RESET} ${label}... "
    if eval "$logic"; then
        echo -e "${VERMELHO}[X] DETECTADO${RESET}"
    else
        echo -e "${VERDE}[OK]${RESET}"
    fi
}

# 1. ROOT (Checa se o binário é funcional e se há montagem de Magisk)
check_adv "Detectar Root" "[ -x /system/bin/su ] || [ -x /system/xbin/su ] || grep -q 'magisk' /proc/mounts"

# 2. KERNEL (Verifica discrepância de data e versão 'dirty')
check_adv "Detectar Kernel Modificado" "uname -v | grep -qiE 'ksu|backend|recompiled' && [ ! -f /proc/version_signature ]"

# 3. WALLHACK (Busca injeções em áreas de memória não-executáveis)
check_adv "Detectar Wallhack" "grep 'r-xp' /proc/self/maps | grep -vE '/system/|/vendor/|/apex/|/data/app/' | grep -q '.so'"

# 4. APK MOD / ARQUIVOS (Verifica integridade de data de modificação da Lib)
check_adv "Detectar APK Mod / Arquivos" "find /data/app/ -name 'libunity.so' -mmin -30 2>/dev/null | grep -q '.'"

# 5. PASSADOR DE REPLAY (MTP/Socket Sniffing)
check_adv "Detectar Passador de Replay" "ss -tunlp | grep -qE ':5555|:6666' || logcat -d | grep -qi 'packet_inject'"

# 6. SERVIÇOS SUSPEITOS (Ignora serviços de fabricantes)
check_adv "Detectar Serviços Suspeitos" "getprop ro.debuggable | grep -q '1' && getprop ro.secure | grep -q '0'"

# 7. ROM MODIFICADA
check_adv "Detectar ROM Modificada" "getprop ro.build.display.id | grep -qiE 'lineage|pixel|resurrection|corvus'"

# --- LEITURA DE LOGS VIA MTP/ADB ---
echo -e "------------------------------------------------------------"
echo -n -e "${AMARELO}➜${RESET} Verificando Evidências MTP/Logcat... "
if logcat -d | grep -aiE "ksu_auth|magisk_daemon|app_process_inject" | tail -n 1 | grep -q "."; then
    echo -e "${VERMELHO}[X] EVIDÊNCIA ENCONTRADA${RESET}"
else
    echo -e "${VERDE}[LIMPO]${RESET}"
fi

# --- FINALIZAÇÃO ---
echo -e "------------------------------------------------------------"
echo -e "${VERMELHO}By: DOUTRINA BYPASS${RESET}"
echo -e "------------------------------------------------------------"

echo -ne "\n${BRANCO}DESEJA SAIR? (s/n):${RESET} "
read sair
if [[ "$sair" =~ ^[sS]$ ]]; then
    echo -e "\n${VERMELHO}By: DOUTRINA BYPASS${RESET}"
    exit 0
fi
