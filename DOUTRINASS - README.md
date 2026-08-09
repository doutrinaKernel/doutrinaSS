#include <iostream>
#include <string>
#include <vector>
#include <fstream>
#include <filesystem>
#include <thread>
#include <chrono>
#include <cstdio>
#include <memory>
#include <array>

namespace fs = std::filesystem;

// ===== PALETA DE CORES STRICT (PRETO E VERMELHO) =====
const std::string RESET = "\033[0m";
const std::string BG_BLACK = "\033[40m";
const std::string BOLD_RED = "\033[1;31m";
const std::string DARK_RED = "\033[0;31m";
const std::string WHITE = "\033[1;37m";
const std::string DARK_GRAY = "\033[1;30m";

// ===== FUNÇÃO PARA EXECUTAR COMANDOS SHELL/ADB =====
std::string execCommand(const char* cmd) {
    std::array<char, 128> buffer;
    std::string result;
    std::unique_ptr<FILE, decltype(&pclose)> pipe(popen(cmd, "r"), pclose);
    if (!pipe) {
        return "";
    }
    while (fgets(buffer.data(), buffer.size(), pipe.get()) != nullptr) {
        result += buffer.data();
    }
    return result;
}

// ===== LEITURA NATIVA C++ (ANTI-HOOK) =====
std::string readNativeFile(const std::string& path) {
    std::ifstream file(path);
    if (!file.is_open()) return "";
    std::string content((std::istreambuf_iterator<char>(file)), std::istreambuf_iterator<char>());
    return content;
}

// ===== ENGINE DE RENDERIZAÇÃO (UI) =====
void printBanner() {
    std::system("clear || cls");
    std::cout << BG_BLACK << BOLD_RED
              << "▄█          ▄██████▄  ████████▄     ▄████████    ▄████████    ▄█   ▄█▄ \n"
              << "███         ███    ███ ███   ▀███   ███    ███   ███    ███   ███ ▄███▀ \n"
              << "███         ███    ███ ███    ███   ███    ███   ███    ███   ███▐██▀   \n"
              << "███         ███    ███ ███    ███   ███    ███  ▄███▄▄▄▄██▀  ▄█████▀    \n"
              << "███         ███    ███ ███    ███ ▀███████████ ▀▀███▀▀▀▀▀   ▀▀█████▄    \n"
              << "███         ███    ███ ███    ███   ███    ███ ▀███████████   ███▐██▄   \n"
              << "███▌    ▄   ███    ███ ███   ▄███   ███    ███   ███    ███   ███ ▀███▄ \n"
              << "█████▄▄██   ▀██████▀  ████████▀    ███    █▀    ███    ███   ███   ▀█▀ \n"
              << "▀                                               ███    ███   ▀         \n"
              << "          [ A D V A N C E D   R O O T   S C A N N E R ]                \n"
              << RESET << "\n";
    std::cout << BG_BLACK << DARK_GRAY << "[!] INICIALIZANDO NÚCLEO DE DETECÇÃO...\n" << RESET;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::system("clear || cls");
}

void drawBlockHeader(const std::string& title) {
    std::cout << BG_BLACK << DARK_RED << "████████████████████████████████████████████████████████████████████████████\n";
    std::cout << BG_BLACK << BOLD_RED << "█ [ " << WHITE << title << BOLD_RED << " ]\n";
    std::cout << BG_BLACK << DARK_RED << "████████████████████████████████████████████████████████████████████████████\n" << RESET;
}

void drawItem(const std::string& label, const std::string& value) {
    std::cout << BG_BLACK << BOLD_RED << "[-] " << DARK_GRAY << label << ": " << WHITE << value << "\n" << RESET;
}

void drawAlert(const std::string& alert) {
    std::cout << BG_BLACK << BOLD_RED << "[!] " << alert << "\n" << RESET;
}

void drawClean(const std::string& msg) {
    std::cout << BG_BLACK << DARK_GRAY << "[+] " << msg << "\n" << RESET;
}

void drawSpacing() {
    std::cout << BG_BLACK << " \n" << RESET;
}

int main() {
    printBanner();

    int threat_count = 0;
    std::vector<std::string> process_logs;
    std::vector<std::string> detection_logs;

    // --- 1. COLETA DE INFORMAÇÕES DE HARDWARE ---
    std::string model = execCommand("getprop ro.product.model");
    std::string android_ver = execCommand("getprop ro.build.version.release");
    std::string kernel_ver = execCommand("uname -r");
    
    if(!model.empty()) model.pop_back();
    if(!android_ver.empty()) android_ver.pop_back();
    if(!kernel_ver.empty()) kernel_ver.pop_back();

    drawBlockHeader("HARDWARE & FIRMWARE INTELLIGENCE");
    drawItem("TARGET DEVICE ", model.empty() ? "DESCONHECIDO" : model);
    drawItem("OS VERSION    ", android_ver.empty() ? "DESCONHECIDO" : android_ver);
    drawItem("KERNEL TARGET ", kernel_ver.empty() ? "DESCONHECIDO" : kernel_ver);
    drawSpacing();

    // --- 2. ANÁLISE DE DAEMONS E PROCESSOS ---
    std::string ps_output = execCommand("ps -A 2>/dev/null");
    
    std::vector<std::pair<std::string, std::string>> signatures = {
        {"magiskd", "MAGISK DAEMON"},
        {"ksud", "KERNELSU DAEMON"},
        {"ksu_next", "KERNELSU NEXT DAEMON"},
        {"apatchd", "APATCH DAEMON"},
        {"sukisu", "SUKISU DAEMON (FORK)"},
        {"suki_", "SUKISU AGENT"},
        {"zn-zygisk", "ZYGISK NEXT ENGINE"},
        {"shamiko", "SHAMIKO HIDER PROCESS"},
        {"trickystore", "TRICKYSTORE INTEGRITY BYPASS"},
        {"kpm_daemon", "KPM BACKGROUND DAEMON"}
    };

    for (const auto& sig : signatures) {
        if (ps_output.find(sig.first) != std::string::npos) {
            process_logs.push_back("DAEMON ATIVO: " + sig.second + " [" + sig.first + "]");
            threat_count++;
        }
    }

    drawBlockHeader("PROCESSOS COMPROMETIDOS & DAEMONS");
    if (process_logs.empty()) {
        drawClean("NENHUM PROCESSO DE ROOT OU DAEMON INJETADO EM EXECUÇÃO.");
    } else {
        for (const auto& log : process_logs) {
            drawAlert(log);
        }
    }
    drawSpacing();

    // --- 3. DETECÇÕES NUCLEARES (KERNEL, SUSFS, KPM, KPIM, .sh) ---
    std::string filesystems = readNativeFile("/proc/filesystems");
    if (filesystems.find("susfs") != std::string::npos) {
        detection_logs.push_back("KERNEL INFILTRATION (SUSFS): Modulo 'susfs' exposto em /proc/filesystems");
        threat_count++;
    }

    std::string cmdline = readNativeFile("/proc/cmdline");
    if (cmdline.find("ksu_susfs") != std::string::npos || cmdline.find("susfs_path") != std::string::npos) {
        detection_logs.push_back("KERNEL BOOT INJECTION: Parâmetros SUSFS detectados na linha de boot");
        threat_count++;
    }
    
    std::vector<std::string> check_cmds = {
        "ls -d /sys/interface/susfs 2>/dev/null",
        "ls -d /sys/fs/susfs 2>/dev/null",
        "ls /dev/ksu_susfs 2>/dev/null",
        "adb shell 'ls -la /data/adb/apatch/kpm/ 2>/dev/null | grep -E \"\\.kpm|\\.kpim\"'",
        "adb shell 'ls -la /data/adb/kpm/ 2>/dev/null'",
        "adb shell 'find /data/adb/ -name \"*.kpm\" -o -name \"*.kpim\" 2>/dev/null'",
        "adb shell 'find /data/adb/modules/ /data/adb/apatch/ /data/adb/ksu/ -name \"*.sh\" 2>/dev/null | grep -iE \"post-fs-data|service\"'"
    };

    for (const auto& cmd : check_cmds) {
        std::string result = execCommand(cmd.c_str());
        if (!result.empty() && result.find("No such file") == std::string::npos) {
            if(cmd.find("*.kpm") != std::string::npos || cmd.find("*.kpim") != std::string::npos) {
                detection_logs.push_back("MÓDULO KPM/KPIM IDENTIFICADO -> " + result.substr(0, result.find('\n')));
                threat_count++;
            } else if(cmd.find("*.sh") != std::string::npos) {
                detection_logs.push_back("SCRIPT .SH INJETADO -> " + result.substr(0, result.find('\n')));
                threat_count++;
            } else {
                detection_logs.push_back("ARTEFATO DE KERNEL SUSPEITO LOCALIZADO.");
                threat_count++;
            }
        }
    }

    drawBlockHeader("MÓDULOS DE KERNEL (KPM/SUSFS) & SCRIPTS INJETADOS");
    if (detection_logs.empty()) {
        drawClean("INTEGRIDADE MÁXIMA: NENHUM VETOR DE KERNEL OU HOOK LOCALIZADO.");
    } else {
        for (const auto& log : detection_logs) {
            drawAlert(log);
        }
    }
    drawSpacing();

    // --- 4. RESULTADO GLOBAL E VEREDITO ---
    drawBlockHeader("VEREDITO DO SISTEMA");
    if (threat_count > 0) {
        std::cout << BG_BLACK << BOLD_RED
                  << "█ STATUS : SISTEMA COMPROMETIDO (ROOT DETECTADO)\n"
                  << "█ THREATS: " << threat_count << " ANOMALIAS CRÍTICAS ENCONTRADAS\n" << RESET;
    } else {
        std::cout << BG_BLACK << DARK_GRAY
                  << "█ STATUS : LIMPO (SISTEMA 100% SEGURO)\n"
                  << "█ THREATS: 0 ANOMALIAS\n" << RESET;
    }
    drawSpacing();

    // --- CRÉDITOS ---
    std::cout << BG_BLACK << DARK_GRAY << "Copyright 2026-2027\n";
    std::cout << BG_BLACK << WHITE << "developers: doutrina king & chapolin\n";
    std::cout << BG_BLACK << BOLD_RED << "by: Lodark Team\n" << RESET;

    return 0;
}
