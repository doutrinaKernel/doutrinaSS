# doutrinaSS

<p align="center">
  <b>Scanner de detecções para FreeFire — Android / Termux</b><br>
  <i>Feito por e para a comunidade que fiscaliza. Rápido, direto e sem depender de acesso root.</i>
</p>

---

## O que é

O **doutrinaSS** é um scanner de terminal que varre o dispositivo Android e reúne, em poucos segundos, tudo o que costuma indicar bypass em FreeFire: vestígios de root, frameworks de hook, shaders/wallhack, modificações na OBB, proxy/VPN, depuração ativa e arquivos suspeitos deixados por mods.

Foi pensado para quem fiscaliza partidas e precisa de um resultado limpo e legível, **sem** precisar de um painel web — o retorno sai direto no terminal, em blocos categóricos e código de evento para cada achado.

### Por que usar o doutrinaSS?

- **Automático** — uma execução e a varredura inteira acontece sozinha.
- **Sem root** — roda no Termux puro. Quando algo exige permissão elevada, ele reporta isso de forma explícita, em vez de inventar resultado.
- **Legível** — saída categorizada (Root, Shaders, OBB, Proxy, Overlay, Localconfig, Logs) com nível de severidade e código de evento.
- **Leve** — binário único, compilado em Go e ofuscado, baixado com um comando.

---

## Como usar

Você só precisa do Termux. Depois de instalado, rode:

```bash
pkg update && pkg upgrade -y
pkg reinstall curl libcurl
pkg install android-tools -y

rm -f doutrinaSS
curl -L -o doutrinaSS https://raw.githubusercontent.com/doutrinaking/doutrinaSS/main/doutrinaSS
chmod +x doutrinaSS
./doutrinaSS
```

Pronto — em segundos o relatório completo aparece na tela.

### Roda sem root?

Sim. O doutrinaSS foi projetado para operar no ambiente padrão do Termux, **sem root**.

O único ponto que esbarra no sandbox do Android é o `Localconfig.json`, que mora dentro de `files/` do pacote do jogo (área privada do app). Sem root ele é marcado como `SEM PERMISSAO / requer root ou Shizuku`; com acesso elevado, o conteúdo é lido e exibido normalmente.

---

## Detecções

| Detecção | O que verifica |
| --- | --- |
| `Instalação do FreeFire` | Se o pacote do jogo está presente |
| `Reinicialização do dispositivo` | Se o boot ocorreu há menos de 60 minutos |
| `Versão do Android` | Versão, SDK e modelo do aparelho |
| `Root` | Binários, caminhos e propriedades de root |
| `Gestores de root` | Magisk, KernelSU, APatch, LSPosed, Xposed, Substrate |
| `Data e Hora` | Possível bypass de relógio / fuso |
| `MTP / ADB` | MTP, depuração USB e porta ADB TCP |
| `Shaders` | Bypass via wallhack / holograma |
| `OBB` | Alterações recentes na OBB do jogo |
| `Localconfig.json` | Presença e conteúdo do arquivo no pacote |
| `Proxy / VPN` | Túnels, SOCKS e portas comuns |
| `Overlay` | Janelas sobrepostas / alert window |
| `Logs` | Palavras-chave de bypass capturadas do logcat |

---

## Códigos de evento

Cada achado é etiquetado com um código, no padrão `E00–E20`, para facilitar a triagem:

`E00` inicialização · `E01` jogo ausente · `E02` boot recente · `E03` root · `E04` data/hora · `E05` replay · `E06` MTP · `E07` shaders · `E08` OBB · `E09` proxy/vpn · `E10` overlay · `E11` depuração · `E12` ambiente suspeito · `E13` gestor de root · `E14` framework de hook · `E15` arquivos suspeitos · `E16` Localconfig ausente · `E17` Localconfig localizado · `E18` Localconfig sem acesso · `E19` keywords nos logs · `E20` verificação concluída.

---

## Compilando você mesmo

Se preferir compilar em vez de baixar o binário:

```bash
pkg install golang -y
go install mvdan.cc/garble@latest

go mod init doutrinass
garble -literals -tiny build -o doutrinaSS doutrinaSS.go
chmod +x doutrinaSS && ./doutrinaSS
```

> O binário publicado já vem compilado e ofuscado. O fonte fica à parte, para quem quiser estudar ou contribuir.

---

## Contribuições

Contribuições são bem-vindas. Achou um bug, tem ideia para uma detecção nova ou quer sugerir um ajuste na saída? Abra uma *issue* ou um *pull request* — toda ajuda soma.

**Reportar um problema** · **Sugerir melhoria** · **Enviar um pull request**

---

## Créditos

- **doutrinaking** — idealização, desenvolvimento, manutenção e owner do projeto.

Este projeto nasceu do esforço de quem passou tempo demais olhando log e entendendo bypass, e é dedicado a quem fiscaliza partidas com paciência e honestidade. Se ele te poupar tempo, ele já cumpriu o papel.

---

## Licença

Copyright © doutrinaking, 2026.

Uso livre para fins de fiscalização e estudo. Não autorizado para redistribuição como produto pago.
