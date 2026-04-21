<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>JAILBREAK - Advanced Diagnostic Pro</title>

<style>
    :root {  
        --bg: #050505; --card-bg: #0f0f10; --card-border: #1a1a1b;  
        --text-primary: #ffffff; --text-secondary: #808084;  
        --accent: #0a84ff; --danger: #ff453a; --success: #32d74b;  
        --warning: #ffd60a; --purple: #bf5af2;  
        --sev-critical: #ff2d55; --sev-high: #ff9f0a; --sev-med: #5e5ce6;  
    }  

    body { font-family: -apple-system, system-ui, sans-serif; background: var(--bg); color: var(--text-primary); margin: 0; padding: 20px; line-height: 1.4; }  
    .container { max-width: 950px; margin: 0 auto; }  

    /* 🔐 LOGIN OVERLAY */
    #loginScreen {
        position: fixed;
        inset: 0;
        background: rgba(0,0,0,0.95);
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        z-index: 9999;
    }

    #loginScreen input {
        padding: 12px;
        width: 250px;
        border-radius: 10px;
        border: none;
        margin-top: 10px;
        text-align: center;
    }

    #loginScreen button {
        margin-top: 10px;
        padding: 12px 20px;
        border: none;
        border-radius: 10px;
        cursor: pointer;
        background: var(--accent);
        color: white;
        font-weight: 800;
    }

    #loginMsg {
        margin-top: 10px;
        font-size: 12px;
        color: var(--danger);
    }

    .hidden { display: none; }
</style>

</head>

<body>

<!-- 🔐 TELA DE KEY (ADICIONADA) -->
<div id="loginScreen">
    <h2>🔐 ACESSO RESTRITO</h2>
    <p>Digite a Key para acessar o Scanner</p>
    <input type="password" id="keyInput" placeholder="KEY">
    <button onclick="checkKey()">ENTRAR</button>
    <div id="loginMsg"></div>
</div>

<!-- 🔓 SEU SISTEMA ORIGINAL (INTACTO) -->
<div id="app" class="hidden">

<div class="container">

<header>  
    <div class="brand">  
        <h1>JAILBREAK DIAGNOSTICS</h1>  
        <p>ADVANCED SECURITY ENGINE • BRISAXX ELITE</p>  
    </div>  
</header>  

<div class="device-panel">  
    <div><span class="info-label">System Version</span><span id="hw-ios" class="info-val">---</span></div>  
    <div><span class="info-label">Build ID</span><span id="hw-build" class="info-val">---</span></div>  
    <div style="grid-column: 1 / -1;"><span class="info-label">Hardware Identifiers (IMEI/UDID)</span><span id="hw-id" class="info-val">---</span></div>  
</div>  

<div class="upload-zone" onclick="document.getElementById('fileInput').click()">  
    <span id="fileLabel">📁 Carregar Sysdiagnose (.log / .txt)</span>  
    <input type="file" id="fileInput" class="hidden">  
</div>  

<button id="analyzeBtn" class="btn-main" disabled>INICIAR SCANNER DE ALTA PRECISÃO</button>  

<div id="resultsArea" class="hidden">  
    <div id="sessionContainer"></div>  
</div>

</div>

</div>

<script>
/* 🔐 KEY FIXA */
const VALID_KEY = "DOUTRINASS";

/* 🔐 VERIFICAÇÃO DE KEY */
function checkKey() {
    const input = document.getElementById("keyInput").value;

    if (input === VALID_KEY) {
        document.getElementById("loginScreen").style.display = "none";
        document.getElementById("app").classList.remove("hidden");
    } else {
        document.getElementById("loginMsg").innerText = "❌ KEY INVÁLIDA";
    }
}
</script>

<!-- 🔻 SEU SCRIPT ORIGINAL INTACTO ABAIXO -->
<script>
    const REGEX = {  
        PROXY: /(https?:\/\/|connection|request).*(freefireproxy\.xyz|dashproxy\.xyz|khoindv\.online|esign\.me)(:\d+)?/i,  
        JB_PATH: /\/var\/jb\/(bin|lib|Library|usr)/i,  
        UUID: /[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}/,  
        TS_IOS: /(\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2})|(\d{2}\/\d{2}\/\d{4},\s\d{2}:\d{2}:\d{2})/  
    };  

    const fileInput = document.getElementById('fileInput');  
    const analyzeBtn = document.getElementById('analyzeBtn');  

    fileInput.addEventListener('change', () => {  
        if(fileInput.files[0]) {  
            document.getElementById('fileLabel').innerText = fileInput.files[0].name;  
            analyzeBtn.disabled = false;  
        }  
    });  

    analyzeBtn.addEventListener('click', async () => {  
        const text = await fileInput.files[0].text();  
        processLogs(text.split('\n'));  
        document.getElementById('resultsArea').classList.remove('hidden');  
    });  

    function processLogs(lines) {
        let hardware = { ios: "---", build: "---", ids: new Set() };  
        let rawEvents = [];  

        lines.forEach(line => {
            const tsMatch = line.match(REGEX.TS_IOS);
            if(!tsMatch) return;

            if(line.includes('ProductVersion:')) hardware.ios = line.split(':')[1].trim();
            if(line.includes('ProductBuildVersion:')) hardware.build = line.split(':')[1].trim();

            const idMatch = line.match(/\b([a-f0-9]{40}|[a-f0-9]{24}|\d{15})\b/i);
            if(idMatch) hardware.ids.add(idMatch[0]);
        });

        document.getElementById('hw-ios').innerText = hardware.ios;
        document.getElementById('hw-build').innerText = hardware.build;
        document.getElementById('hw-id').innerText = Array.from(hardware.ids).join(' | ') || "Not Found";
    }
</script>

</body>
</html>
