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

## Introdução

doutrinaSS é um scanner para dispositivos Android que tem o objetivo de reunir logs e arquivos suspeitos em questão de segundos de utilização.

**Por que usar o doutrinaSS?**

O projeto tem como principal função facilitar o trabalho dos analistas em suas telagens, que contém várias funções, como:

* **Automação:** O scanner faz todo o trabalho pesado por você, poupando seu tempo.
* **Logs suspeitas:** Reúne logs de todos os possíveis bypass para você automaticamente.
* **Facilidade:** O scanner roda utilizando `Termux`, e com alguns simples comandos você já vai estar rodando ele sem problemas.

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

Desenvolvido por **doutrinaking**.

<p align="center">
  <em>doutrinaSS — Scanner de Detecções FreeFire · Android / Termux</em>
</p>
