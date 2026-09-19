# doutrinaSS

<div id="top">

<p align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/banner.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/banner.png">
  <img alt="doutrinaSS" src="assets/banner.png" width="60%">
</picture>

</p>

<p align="center">
  <em>Pensado e realizado em prol da comunidade de FreeFire, por doutrinaking.</em>
</p>

</div>

<img src="assets/divider.png" alt="line break" width="100%" height="3px">


## Introdução

doutrinaSS é um scanner para dispositivos Android que tem o objetivo de reunir logs e arquivos suspeitos em questão de segundos de utilização.

**Por que usar o doutrinaSS?**

O projeto tem como principal função facilitar o trabalho dos analistas em suas telagens, que contém várias funções, como:

* **Automação:** O scanner faz todo o trabalho pesado por você, poupando seu tempo.
* **Logs suspeitas:** Reúne logs de todos os possíveis bypass para você automaticamente.
* **Facilidade:** O scanner roda utilizando `Termux`, e com alguns simples comandos você já vai estar rodando ele sem problemas.


## Como utilizar?


#### <img width="2%" src="https://simpleicons.org/icons/diagramsdotnet.svg">&emsp13; Faça o download do Termux:


| Aplicativo                  | Descrição                |
|----------------------------|---------------------------|
| [Termux](https://f-droid.org/repo/com.termux_1022.apk) | Terminal utilizado para rodar o scanner   |


#### <img width="2%" src="https://simpleicons.org/icons/termius.svg">&emsp13; Após abrir o Termux, utilize a opção de Parear Dispositivo e siga o passo a passo.

```sh
pkg update && pkg upgrade -y && pkg reinstall curl libcurl && pkg install android-tools -y && rm -f doutrinaSS && curl -L -o doutrinaSS https://raw.githubusercontent.com/doutrinaKernel/doutrinaSS/main/doutrinaSS && chmod +x doutrinaSS && ./doutrinaSS
