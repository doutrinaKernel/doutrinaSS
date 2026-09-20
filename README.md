# doutrinaSS

<div id="top">

<p align="center">

<img alt="doutrinaSS" src="doutrinaSS.png" width="100%">

</p>

<p align="center">
  <em>Pensado e realizado em prol da comunidade de FreeFire, por doutrinaking.</em>
</p>

</div>

---

## **Introdução**

**doutrinaSS é um scanner para dispositivos Android que tem o objetivo de reunir logs e arquivos suspeitos em questão de segundos de utilização.**


**Por que usar o doutrina SS?**


**O projeto tem como principal função facilitar o trabalho dos analistas em suas telagens, que contém várias funções, como:**


* **Automação:** **O scanner faz todo o trabalho pesado por você, poupando seu tempo.**

* 
* **Logs suspeitas:** **Reúne logs de todos os possíveis bypass para você automaticamente.**

* 
* **Facilidade:** **O scanner roda utilizando `Termux`, e com alguns simples comandos você já vai estar rodando ele sem problemas.**


## Detectações do scanner:


| Detectações               | Descrição                                   |
|----------------------|-----------------------------------------------|
| `Verificação da instalação do FreeFire`            | Verificar se o jogo está instalado                      | `center`        |
| `Reinicialização do dispositivo`              | Verifica se o dispositivo foi reiniciado a menos de 60 minutos                      |  
| `Versão Android`      | Verifica a versão do Android                |
| `Root`      | Verifica se o dispositivo possui Root                      |
| `Data e Hora`     | Verifica bypass de Data e Hora                       |
| `Passagem de Replay` | Verifica se o usuário passou Replay                      | 
| `MTP`           | Verifica se o MTP está ativado  |
| `Shaders`             | Verifica se o usuário deu bypass usando wallhack/holograma                            | 
| `OBB`        | Verifica se o usuário deu algum tipo de bypass na OBB                               |


---

## Como utilizar?

### 📱 Faça o download do Termux

| Aplicativo | Descrição |
|---|---|
| [Termux](https://f-droid.org/repo/com.termux_1022.apk) | Terminal utilizado para rodar o scanner |

### 🔗 Após abrir o Termux

Utilize a opção de **Parear Dispositivo** e siga o passo a passo.

Depois, execute o comando abaixo:

```sh
pkg update && pkg upgrade -y && pkg reinstall curl libcurl && pkg install android-tools -y && rm -f doutrinaSS && curl -L -o doutrinaSS https://raw.githubusercontent.com/doutrinaKernel/doutrinaSS/main/doutrinaSS && chmod +x doutrinaSS && ./doutrinaSS
```
 



## Créditos

<p align="center">

<img src="doutrinaking.jpg" width="180px" alt="doutrinaking">


<br><br>

<strong>doutrinaking</strong>


<a href="https://discord.com/users/doutrinaking" target="_blank">
  <img src="https://img.shields.io/badge/Discord-doutrinaking-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord">
</a>

<a href="https://www.instagram.com/lucasxzzn/" target="_blank">
  <img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white" alt="Instagram">
</a>

<a href="https://discord.gg/kazebypass" target="_blank">
  <img src="https://img.shields.io/badge/⚡_KAZE_BYPASS-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="KAZE BYPASS">
</a>
</p>

---

<p align="center">
  <em>doutrinaSS — Scanner de Detecções FreeFire · Android / Termux</em>
</p>
