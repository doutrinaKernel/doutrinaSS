#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# LODARK TEAM FREE - ADVANCED ROOT & WALLHACK DETECTION SYSTEM
# MODELO DE DETECTAÇÃO SSCI: VERMELHO (CRÍTICO), AMARELO (ALERTA), CINZA (INFO)

import subprocess
import sys
import re
from datetime import datetime
from textual.app import App, ComposeResult
from textual.containers import Container
from textual.widgets import Header, Footer, Label, RichLog, ProgressBar
from rich.text import Text

# ========= ADB FUNCTIONS =========
def adb(cmd):
    try:
        result = subprocess.run(f"adb shell '{cmd}' 2>/dev/null", shell=True, capture_output=True, text=True, timeout=5)
        return result.stdout.strip()
    except:
        return ""

def adb_test(cmd):
    try:
        result = subprocess.run(f"adb shell '{cmd}' 2>/dev/null", shell=True)
        return result.returncode == 0
    except:
        return False

def adb_list(cmd):
    try:
        result = subprocess.run(f"adb shell '{cmd}' 2>/dev/null", shell=True, capture_output=True, text=True)
        return [l for l in result.stdout.strip().split('\n') if l]
    except:
        return []

def check_adb():
    result = subprocess.run("adb devices", shell=True, capture_output=True, text=True)
    lines = result.stdout.strip().split('\n')[1:]
    for line in lines:
        if 'device' in line and 'unauthorized' not in line:
            return line.split('\t')[0]
    return None

# ========= NEW DETECTIONS ACCORDING TO SPECS =========

def check_wallhack_kernel():
    detections = []
    agora = datetime.now().strftime("%H:%M:%S")
    
    paths_to_stat = [
        "/sys/kernel/Shaders",
        "/sys/kernel/optional",
        "/proc/sys/kernel/shaders",
        "/data/local/tmp/Shaders",
        "/data/local/tmp/optional",
        "/dev/shm/shaders"
    ]
    
    for path in paths_to_stat:
        if adb_test(f"[ -e '{path}' ]"):
            stat_out = adb(f"stat '{path}' 2>/dev/null")
            if "5555" in stat_out or "9999" in stat_out:
                detections.append(("WALLHACK DETECTADO !", f"Inconsistência em {path} (ID/Nulo 5555/9999 encontrado no stat)", agora))
    
    cmdline = adb("cat /proc/cmdline 2>/dev/null")
    if "5555" in cmdline or "9999" in cmdline:
        if "shader" in cmdline.lower() or "optional" in cmdline.lower():
            detections.append(("WALLHACK DETECTADO !", "Parâmetro suspeito no cmdline do Kernel", agora))
            
    return detections

def detect_connection_logs():
    detections = []
    keywords = [
        "adbdebugging", "USBdebugging", "MtpServer", "mtp transfer", 
        "usb_config", "mtp_open", "mtp_release", "Functionfs", "adb_open"
    ]
    
    logcat_lines = adb("logcat -d -v time *:V | grep -iE 'mtp|usb|adb' | tail -n 30")
    
    if logcat_lines:
        for line in logcat_lines.split('\n'):
            line = line.strip()
            if any(kw.lower() in line.lower() for kw in keywords):
                match_time = re.match(r'^\d{2}-\d{2}\s+(\d{2}:\d{2}:\d{2})', line)
                hora_log = match_time.group(1) if match_time else datetime.now().strftime("%H:%M:%S")
                detections.append(("CONEXÃO DETECTADA", line, hora_log))
                
    usb_state = adb("getprop sys.usb.state")
    usb_config = adb("getprop sys.usb.config")
    if "mtp" in usb_state or "adb" in usb_state:
        agora = datetime.now().strftime("%H:%M:%S")
        detections.append(("STATUS USB ATIVO", f"Estado: {usb_state} | Config: {usb_config}", agora))
        
    return detections

# ========= BASE ROOT DETECTION (FROM TEMPLATE) =========
def analise_processos():
    resultados = []

    # CORREÇÃO 1: Em vez de usar \[, usamos -Fv '[' para procurar diretamente o bracket fixo, evitando o SyntaxWarning
    cmd = "ps -A -o pid,user,args 2>/dev/null | grep '^root' | grep -Fv '[' | head -15"
    root_procs = adb(cmd)
    if root_procs:
        for line in root_procs.split('\n'):
            if line.strip():
                resultados.append(("ROOT", line.strip()))

    cmd = "ps -A 2>/dev/null | grep -i 'busybox'"
    busybox = adb(cmd)
    if busybox:
        for line in busybox.split('\n'):
            if line.strip():
                resultados.append(("BUSYBOX", line.strip()))

    cmd = "ps -A 2>/dev/null | grep -i 'zn-zygisk-companion64'"
    zn_proc = adb(cmd)
    if zn_proc:
        resultados.append(("ZYGISK", zn_proc))

    cmd = "ps -A 2>/dev/null | grep -i 'playintegrityfix'"
    pif_proc = adb(cmd)
    if pif_proc:
        resultados.append(("PIF", pif_proc))

    return resultados

def detect_pm_uninstall():
    detections = []
    agora = datetime.now().strftime("%H:%M:%S")

    uninstalled_packages = adb("pm list packages -u 2>/dev/null | sed 's/package://g'")
    installed_packages = adb("pm list packages 2>/dev/null | sed 's/package://g'")

    installed_set = set(installed_packages.split('\n'))
    system_prefixes = ["com.sec.", "com.samsung.", "com.android.", "android.", "com.google.android.", "com.qualcomm."]

    for pkg in uninstalled_packages.split('\n'):
        pkg = pkg.strip()
        if not pkg:
            continue

        is_system = any(pkg.startswith(prefix) for prefix in system_prefixes)
        if pkg not in installed_set and not is_system and len(pkg) > 5:
            detections.append(("PM UNINSTALL", pkg, agora))

    return detections

def collect_detections():
    detections = []
    agora = datetime.now().strftime("%H:%M:%S")

    wallhack_dets = check_wallhack_kernel()
    for tipo, detalhe, hora in wallhack_dets:
        detections.append((tipo, detalhe, hora))

    if adb_test("lsmod | grep -q 'susfs4ksu'"):
        detections.append(("Kernel SUSFS", "MODULO 'susfs4ksu' encontrado", agora))

    cmdline = adb("cat /proc/cmdline 2>/dev/null")
    if "ksu_susfs" in cmdline:
        detections.append(("Kernel SUSFS", "PARAMETRO 'ksu_susfs' detectado", agora))

    if adb_test("[ -e '/dev/ksu_susfs' ]"):
        detections.append(("Kernel SUSFS", "DISPOSITIVO '/dev/ksu_susfs' encontrado", agora))

    if adb_test("[ -f '/data/adb/magisk.log' ]"):
        detections.append(("Magisk Logs", "magisk.log encontrado", agora))

    if adb_test("logcat -d -s KernelSU 2>/dev/null | head -1 | grep -q '.'"):
        detections.append(("KernelSU Logs", "logs encontrados", agora))

    pm_detections = detect_pm_uninstall()
    for tipo, pkg, hora in pm_detections:
        detections.append((tipo, pkg, hora))

    if adb_test("ps -A 2>/dev/null | grep -qi 'zn-zygisk-companion64'"):
        detections.append(("Play Integrity Fix", "ATIVO", agora))

    proc_version = adb("cat /proc/version")
    if "Wild" in proc_version:
        detections.append(("Kernel", "WILD modificado", agora))
    if "1970" in proc_version:
        detections.append(("Kernel Timestamp", "ZERADO - 1970", agora))

    wb_snap = adb("getprop ro.boot.wb.snapQB")
    if wb_snap and wb_snap != "0":
        detections.append(("Firmware Samsung", f"CUSTOM - {wb_snap}", agora))

    # CORREÇÃO: Utilizar -qiE em vez de -qi para usar grep regex estendido e não precisar dar escape em \|
    if adb_test("logcat -d 2>/dev/null | grep -qiE 'zygisk|zygote'"):
        detections.append(("Zygisk", "INJECAO ATIVA", agora))

    if adb_test("pm list packages 2>/dev/null | grep -qiE 'lsposed|xposed'"):
        detections.append(("LSPosed/Xposed", "FRAMEWORK PRESENTE", agora))

    if adb_test("ps -A 2>/dev/null | grep -qiE 'ksud|magiskd'"):
        detections.append(("Hidden Process", "ksud/magiskd rodando", agora))

    if adb_test("[ -d '/data/adb/modules/brene' ] || [ -d '/data/adb/modules/BRENE' ]"):
        detections.append(("BRENE Module", "INSTALADO", agora))
    if adb_test("[ -d '/data/adb/modules/susfs4ksu' ]"):
        detections.append(("SUSFS4KSU", "INSTALADO", agora))
    if adb_test("[ -d '/data/adb/modules/playintegrityfix' ]"):
        detections.append(("PIF Module", "INSTALADO", agora))

    if adb_test("cat /proc/self/maps 2>/dev/null | grep -q 'rwxp'"):
        detections.append(("Inline Hook", "rwxp detectado", agora))

    ns_pid = adb("readlink /proc/self/ns/mnt")
    ns_init = adb("readlink /proc/1/ns/mnt")
    if ns_pid and ns_init and ns_pid != ns_init:
        detections.append(("Isolated Namespace", "MOMO DETECTION", agora))

    # CORREÇÃO: Transformar a string em um raw string (r"...") para o Python não reclamar do \.sh
    scripts = adb_list(r"ls /data/local/tmp/ 2>/dev/null | grep -i '\.sh$'")
    for script in scripts:
        detections.append((f"Suspicious Script", script, agora))

    if adb("getprop ro.boot.flash.locked") == "0":
        detections.append(("Bootloader", "DESTRAVADO", agora))

    return detections

# ========= TEXTUAL APPLICATION INITIALIZATION =========
class LodarkTeamScannerApp(App):
    CSS = """
    Screen { background: #0d0d0d; }
    #main-container { layout: vertical; padding: 1; background: #0d0d0d; }
    .info-box { border: solid #555555; background: #141414; margin: 1; padding: 1; }
    .threat-box { border: solid #ff0000; background: #141414; margin: 1; padding: 1; }
    .status-box { border: solid #888888; background: #141414; margin: 1; padding: 1; }
    .processos-box { border: solid #ffff00; background: #141414; margin: 1; padding: 1; }
    .detections-box { border: solid #ff0000; background: #141414; margin: 1; padding: 1; }
    .resultado-box { background: #0d0d0d; margin: 1; padding: 1; }
    RichLog { background: #111111; }
    .label-title { color: #888888; text-style: bold; }
    .label-value { color: #ffffff; }
    .threat-text { color: #ff0000; text-style: bold; }
    .warning-text { color: #ffff00; text-style: bold; }
    .info-text { color: #888888; }
    ProgressBar { color: #ff0000; }
    """

    def compose(self) -> ComposeResult:
        yield Header(show_clock=True)

        with Container(id="main-container"):
            with Container(classes="info-box"):
                yield Label("LODARK TEAM FREE - INFORMAÇÕES DO SISTEMA", classes="label-title")
                yield Label("")
                yield Label("DISPOSITIVO: ", classes="label-title")
                yield Label("", id="target-val", classes="label-value")
                yield Label("ANDROID: ", classes="label-title")
                yield Label("", id="android-val", classes="label-value")
                yield Label("KERNEL: ", classes="label-title")
                yield Label("", id="kernel-val", classes="label-value")
                yield Label("TEMPO ATIVO: ", classes="label-title")
                yield Label("", id="uptime-val", classes="label-value")

            with Container(classes="threat-box"):
                yield Label("MÉTRICAS DE AMEAÇA SYSTEM", classes="threat-text")
                yield Label("")
                yield Label("TOTAL DE ANOMALIAS: ", classes="label-title")
                yield Label("", id="threat-count", classes="threat-text")
                yield Label("NÍVEL GERAL SSCI: ", classes="label-title")
                yield Label("", id="threat-level", classes="info-text")
                yield Label("SINAL ADB: ", classes="label-title")
                yield Label("", id="signal-val", classes="warning-text")

            with Container(classes="status-box"):
                yield Label("MONITORAMENTO DE CONEXÕES & TRANFERÊNCIAS MTP/USB", classes="label-title")
                yield Label("")
                yield ProgressBar(id="energy-bar", total=100, show_percentage=True)
                yield RichLog(id="connection-log", wrap=True, markup=True)

            with Container(classes="processos-box"):
                yield Label("ANÁLISE DE PROCESSOS E DAEMONS", classes="warning-text")
                yield Label("")
                yield RichLog(id="processos-log", wrap=True, markup=True)

            with Container(classes="detections-box"):
                yield Label("LOGS DE AUDITORIA CRÍTICA & INTEGRIDADE", classes="threat-text")
                yield Label("")
                yield RichLog(id="detections-log", wrap=True, markup=True)

            with Container(classes="resultado-box"):
                yield RichLog(id="resultado-log", wrap=True, markup=True)

        yield Footer()

    def on_mount(self):
        self.run_scan()

    def run_scan(self):
        device = check_adb()

        model = adb("getprop ro.product.model")
        android = adb("getprop ro.build.version.release")
        kernel = adb("uname -r")
        
        # CORREÇÃO 2: As aspas triplas ( """ ) protegem todas as aspas duplas e simples dentro do comando contra o erro SyntaxError apontado
        uptime_cmd = """cat /proc/uptime | awk '{print int($1/3600)"h "int(($1%3600)/60)"m"}'"""
        uptime = adb(uptime_cmd)

        self.query_one("#target-val").update(model or "DESCONHECIDO")
        self.query_one("#android-val").update(android or "DESCONHECIDO")
        self.query_one("#kernel-val").update(kernel or "DESCONHECIDO")
        self.query_one("#uptime-val").update(uptime or "DESCONHECIDO")
        self.query_one("#signal-val").update("ONLINE" if device else "OFFLINE")

        conn_logs = detect_connection_logs()
        log_conn = self.query_one("#connection-log")
        log_conn.clear()
        
        if conn_logs:
            for tipo, log_msg, hora in conn_logs[:15]: 
                log_conn.write(Text(f"[{hora}] [{tipo}] -> {log_msg}", style="yellow"))
        else:
            log_conn.write(Text("[+] Nenhuma transferência ou log suspeito de MTP/USB no momento.", style="grey50"))

        processos = analise_processos()
        log_proc = self.query_one("#processos-log")
        log_proc.clear()

        tem_processo_suspeito = False
        if processos:
            for tipo, linha in processos:
                if tipo in ["ROOT", "ZYGISK", "PIF"]:
                    log_proc.write(Text(f"[!] DETECTADO -> {tipo}: {linha}", style="red"))
                    tem_processo_suspeito = True
                elif tipo == "BUSYBOX":
                    log_proc.write(Text(f"[!] ALERTA -> BUSYBOX: {linha}", style="yellow"))
                    tem_processo_suspeito = True
        else:
            log_proc.write(Text("[+] Nenhum processo ou binário invasivo em execução.", style="grey50"))

        detections = collect_detections()
        count = len(detections)
        self.query_one("#threat-count").update(str(count))

        has_wallhack = any(d == "WALLHACK DETECTADO !" for d, _, _ in detections)
        
        if count == 0 and not tem_processo_suspeito:
            self.query_one("#threat-level").update("LIMPO (CINZA)")
        elif has_wallhack or tem_processo_suspeito:
            self.query_one("#threat-level").update("PERIGO MÁXIMO (VERMELHO)")
        else:
            self.query_one("#threat-level").update("SUSPEITO (AMARELO)")

        energy = min(100, count * 8 if count > 0 else 0)
        self.query_one("#energy-bar").progress = energy

        log_det = self.query_one("#detections-log")
        log_det.clear()

        tem_root_or_cheat = False

        if detections:
            for d, detail, hora in detections:
                if d == "WALLHACK DETECTADO !":
                    log_det.write(Text(f"[*] [{hora}] {d} - {detail}", style="bold red"))
                    tem_root_or_cheat = True
                else:
                    log_det.write(Text(f"[!] [{hora}] {d}: {detail}", style="red"))
                    tem_root_or_cheat = True
        else:
            log_det.write(Text("[+] Sistema íntegro nas validações de kernel.", style="grey50"))

        if tem_processo_suspeito:
            tem_root_or_cheat = True

        resultado_log = self.query_one("#resultado-log")
        resultado_log.clear()

        if has_wallhack:
            resultado_log.write(Text("═" * 70, style="red"))
            resultado_log.write(Text("║" + " WALLHACK DETECTADO ! ".center(68) + "║", style="bold red"))
            resultado_log.write(Text("║" + "MODIFICAÇÃO DE SHADERS / KERNEL DETECTADA".center(68) + "║", style="red"))
            resultado_log.write(Text("═" * 70, style="red"))
        elif tem_root_or_cheat:
            resultado_log.write(Text("═" * 70, style="yellow"))
            resultado_log.write(Text("║" + " R O O T   D E T E C T A D O ".center(68) + "║", style="bold yellow"))
            resultado_log.write(Text("║" + "SISTEMA SE COMPORTOU DE MANEIRA INSEGURA".center(68) + "║", style="yellow"))
            resultado_log.write(Text("═" * 70, style="yellow"))
        else:
            resultado_log.write(Text("═" * 70, style="grey50"))
            resultado_log.write(Text("║" + " DISPOSITIVO SEGURO ".center(68) + "║", style="bold grey50"))
            resultado_log.write(Text("═" * 70, style="grey50"))

if __name__ == "__main__":
    LodarkTeamScannerApp().run()
