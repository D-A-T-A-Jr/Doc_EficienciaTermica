# Doc_EficienciaTermica
# ❄️ Controle Adaptativo e Eficiência Térmica em Salas de Aula com IoT

Este repositório contém a documentação técnica e os fontes em LaTeX do projeto de automação e eficiência energética para climatização em ambientes educacionais, desenvolvido com **ESP32**, atuador **IR** e integração via **ThingsBoard**.

---

## 📌 Visão Geral do Problema

Em salas de aula climatizadas por ar-condicionado convencional, é comum fixar o termostato em 23 °C (ou abaixá-lo para 17 °C em dias quentes). No entanto:
- **Reduzir o setpoint não acelera o resfriamento:** o compressor opera em 100% de capacidade assim que detecta o diferencial térmico;
- **Desperdício e Sobrecarga:** em dias de calor severo somados à alta lotação, a carga térmica da sala supera a capacidade nominal do aparelho, gerando consumo elétrico elevado, desgaste mecânico e ruído excessivo sem atingir a temperatura desejada;
- **Ruído operacional via IoT:** disparos frequentes de comandos infravermelho (IR) ou histereses inadequadas causam bips sonoros e oscilações desconfortáveis durante as aulas.

---

## ⚙️ Arquitetura e Soluções Adotadas

O sistema integra dados de uma **estação meteorológica externa** e de um **sensor DHT11 interno** processados pelo ESP32 e pela *Rule Chain* do ThingsBoard.

### 1. Setpoint Dinâmico (ASHRAE 55)
Em vez de um valor fixo, o ponto de ajuste varia de forma adaptativa com base na temperatura externa ($T_{\text{ext}}$):
$$T_{\text{comf}} = 17{,}8 + 0{,}31 \cdot T_{\text{ext}}$$

Para garantir conforto cognitivo sem sobrecarga, o setpoint é limitado (*clamping*) entre **22 °C e 25 °C**:
$$T_{\text{setpoint}} = \text{round}\Big( \min\big(25{,}0\,;\; \max(22{,}0\,;\; 17{,}8 + 0{,}31 \cdot T_{\text{ext}})\big) \Big)$$

### 2. Estabilidade de Controle e Histerese
- **Banda morta:** histerese simétrica de $\pm 1{,}0\text{ °C}$ para evitar oscilações bruscas (histereses maiores geravam até 5 °C de oscilação real percebida);
- **Trava de tempo (*Hold Time*):** intervalo mínimo de 15 minutos entre comandos IR para respeitar a inércia térmica do ambiente;
- **Média móvel:** filtro de 5 amostras nas leituras do sensor interno para amortecer ruídos e rajadas momentâneas de portas abertas.

### 3. Automação de Desligamento Passivo
- Se $T_{\text{ext}} \le 21{,}0\text{ °C}$ ou $T_{\text{int}} \le 20{,}5\text{ °C}$, o ciclo ativo é desnecessário. O sistema comuta automaticamente para **OFF** (ventilação natural passiva).

---

## 📊 Matriz Operacional e Coeficiente de Performance (COP)

Referência: Sistema *Split Inverter* 18.000 BTU/h (Classe A).

| $T_{\text{ext}}$ (°C) | Estado do AC | $T_{\text{setpoint}}$ (°C) | Carga Estimada | COP Estimado | Regime de Operação |
| :---: | :---: | :---: | :---: | :---: | :--- |
| $\le 21{,}0$ | **Desligado** | — | Baixa ($< 1{,}8\text{ kW}$) | — | Ventilação Natural |
| $22{,}0$ | Ligado | $22{,}0$ | Leve ($\approx 2{,}4\text{ kW}$) | $4{,}15$ | Inverter em rotação mínima |
| $24{,}0$ | Ligado | $23{,}0$ | Moderada ($\approx 3{,}1\text{ kW}$) | $3{,}80$ | Faixa de máxima eficiência |
| $26{,}0$ | Ligado | $23{,}0$ | Padrão ($\approx 3{,}8\text{ kW}$) | $3{,}52$ | Operação nominal contínua |
| $28{,}0$ | Ligado | $24{,}0$ | Alta ($\approx 4{,}4\text{ kW}$) | $3{,}21$ | Modulação intermediária |
| $30{,}0$ | Ligado | $24{,}0$ | Severa ($\approx 5{,}0\text{ kW}$) | $2{,}95$ | Alta pressão na linha de descarga |
| $32{,}0$ | Ligado | $25{,}0$ | Crítica ($\approx 5{,}4\text{ kW}$) | $2{,}70$ | Compressor próximo a 100% |
| $\ge 35{,}0$ | Ligado | $25{,}0$ | Sobrecarga ($> 5{,}8\text{ kW}$) | $2{,}35$ | Capacidade saturada / Gargalo |

---

## 📂 Estrutura de Arquivos

```text
├── main.tex           # Documento principal em LaTeX pronto para compilação (Overleaf)
├── referencias.bib    # Base bibliográfica (ASHRAE 55, ISO 7730, Çengel, INMETRO)
└── README.md          # Visão geral e resumo do projeto
