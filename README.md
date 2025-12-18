# 💧 Home Assistant – Caixa d’Água em Litros (Tuya / Smart Life)

Projeto para Home Assistant com sensores, automações e card personalizado
para controle de nível, abastecimento e histórico da caixa d’água.

---

## 📸 Visual do Card

<img src="Card.jpg" width="400">

<img src="www/fundocaixa1.png" width="400">

---

## 📦 1️⃣ Conversão das MEDIDAS

✔ **Installation height**  
0,7 m = 70 cm  

✔ **Maximum liquid depth**  
0,1 m = 10 cm  
🔎 Observação importante sobre a medição:
Apesar das medidas físicas estarem em metros/centímetros, a medição final do nível NÃO é feita por altura, e sim por percentual de volume.
A caixa possui 500 litros (100%), onde 1% equivale a 5 litros. Dessa forma, o valor exibido representa diretamente o volume real em litros, reduzindo erros comuns da medição por altura.
Exemplo: 91% = 455 litros | 10% = 50 litros | 1% = 5 litros

## 📦 2️⃣ Sensores Tuya (referência)

```text
number.nivel_do_tanque_a_alarm_maximum
number.nivel_do_tanque_a_alarm_minimum
sensor.nivel_do_tanque_a_liquid_level
```


## 📦 3️⃣ Sensor de LITROS (base do projeto)

📄 **sensors.yaml**

```yaml
# ==============================
# Caixa d'água - Litros
# ==============================
- platform: template
  sensors:
    caixa_agua_litros:
      friendly_name: "Caixa d'água - Litros"
      unit_of_measurement: "L"
      device_class: water
      icon_template: mdi:water
      value_template: >
        {% set percentual = states('sensor.nivel_do_tanque_a_liquid_level') | float(0) %}
        {{ (percentual * 5) | round(0) }}

# ==============================
# Sensor de VARIAÇÃO da caixa (CHAVE DO SISTEMA)
# ==============================
- platform: template
  sensors:
    variacao_caixa_agua_litros:
      friendly_name: "Variação Caixa d'água"
      unit_of_measurement: "L"
      value_template: >
        {% set atual = states('sensor.caixa_agua_litros') | float(0) %}
        {% set anterior = states('input_number.caixa_agua_litros_anterior') | float(0) %}
        {{ (atual - anterior) | round(1) }}

# ==============================
# Água que entrou na caixa
# ==============================
- platform: template
  sensors:
    caixa_agua_entrou:
      friendly_name: "Água que entrou na caixa"
      unit_of_measurement: "L"
      device_class: water
      icon_template: mdi:water-plus
      value_template: >
        {% set v = states('sensor.variacao_caixa_agua_litros') | float(0) %}
        {{ v if v > 0 else 0 }}

# ==============================
# Água que saiu da caixa (CONSUMO)
# ==============================
- platform: template
  sensors:
    caixa_agua_saiu:
      friendly_name: "Água que saiu da caixa"
      unit_of_measurement: "L"
      device_class: water
      icon_template: mdi:water-minus
      value_template: >
        {% set v = states('sensor.variacao_caixa_agua_litros') | float(0) %}
        {{ (v * -1) if v < 0 else 0 }}


# ==============================
# Consumo acumulado da Caixa (referência)
# ==============================
- platform: template
  sensors:
    consumo_caixa_litros:
      friendly_name: "Consumo da Caixa (Total)"
      unit_of_measurement: "L"
      device_class: water
      value_template: >
        {{ states('sensor.caixa_agua_litros') | float(0) }}

# ==============================
# Custos da Caixa d'água
# ==============================
- platform: template
  sensors:
    custo_caixa_agua_atual:
      friendly_name: "Custo Atual da Caixa"
      unit_of_measurement: "R$"
      icon_template: mdi:currency-brl
      value_template: >
        {{ (states('sensor.caixa_agua_litros') | float(0) * 0.00795) | round(2) }}

    custo_caixa_agua_diario:
      friendly_name: "Custo Diário da Caixa"
      unit_of_measurement: "R$"
      icon_template: mdi:currency-brl
      value_template: >
        {{ (states('sensor.consumo_caixa_diario') | float(0) * 0.00795) | round(2) }}

    custo_caixa_agua_mensal:
      friendly_name: "Custo Mensal da Caixa"
      unit_of_measurement: "R$"
      icon_template: mdi:currency-brl
      value_template: >
        {{ (states('sensor.consumo_caixa_mensal') | float(0) * 0.00795) | round(2) }}

# ==============================
# Litros do último abastecimento
# ==============================
- platform: template
  sensors:
    litros_ultimo_abastecimento_caixa:
      friendly_name: "Litros do Último Abastecimento"
      unit_of_measurement: "L"
      icon_template: mdi:water-plus
      value_template: >
        {% set ini = states('input_number.litros_inicio_abastecimento') | float(0) %}
        {% set fim = states('input_number.litros_fim_abastecimento') | float(0) %}
        {{ (fim - ini) | max(0) | round(0) }}


# ==============================
# Formatação do último abastecimento
# ==============================
- platform: template
  sensors:
    ultimo_abastecimento_caixa_formatado:
      friendly_name: "Último Abastecimento Caixa"
      value_template: >
        {% set dt = states('input_datetime.ultimo_abastecimento_caixa') %}
        {% if dt not in ['unknown', 'unavailable', ''] %}
          {{ strptime(dt, '%Y-%m-%d %H:%M:%S').strftime('%d/%m/%y - %H:%M') }}
        {% else %}
          --
        {% endif %}
        
# ==============================
# ACUMULADOR DE ENTRADA
# ==============================
- platform: template
  sensors:
    caixa_agua_entrada_total:
      friendly_name: "Entrada Total Caixa"
      unit_of_measurement: "L"
      device_class: water
      value_template: >
        {% set atual = states('sensor.caixa_agua_litros') | float(0) %}
        {% set anterior = states('input_number.caixa_agua_litros_anterior') | float(0) %}
        {% set total = states('sensor.caixa_agua_entrada_total') | float(0) %}
        {% if atual > anterior %}
          {{ (total + (atual - anterior)) | round(1) }}
        {% else %}
          {{ total | round(1) }}
        {% endif %}

# ==============================
# ACUMULADOR DE SAÍDA (CONSUMO)
# ==============================
- platform: template
  sensors:
    caixa_agua_saida_total:
      friendly_name: "Saída Total Caixa"
      unit_of_measurement: "L"
      device_class: water
      value_template: >
        {% set atual = states('sensor.caixa_agua_litros') | float(0) %}
        {% set anterior = states('input_number.caixa_agua_litros_anterior') | float(0) %}
        {% set total = states('sensor.caixa_agua_saida_total') | float(0) %}
        {% if atual < anterior %}
          {{ (total + (anterior - atual)) | round(1) }}
        {% else %}
          {{ total | round(1) }}
        {% endif %}
        

#Alera led verde vermelho
- platform: template
  sensors:
    status_caixa_agua:
      friendly_name: "Status Caixa d'Água"
      value_template: >
        {% set nivel = states('sensor.nivel_do_tanque_a_liquid_level') | float(0) %}
        {% set min = states('number.nivel_do_tanque_a_alarm_minimum') | float(0) %}
        {% set max = states('number.nivel_do_tanque_a_alarm_maximum') | float(100) %}
        {% if nivel < min %}
          baixo
        {% elif nivel < ((min + max) / 2) %}
          medio
        {% else %}
          ok
        {% endif %}

```

**Caixa de 500 L → cada 1% = 5 L**


## 📦 4️⃣ Sensor ACUMULADOR (ENTRADA)

📄 **sensors.yaml**

```yaml
# ==============================
# Entrada total de água na caixa
# ==============================
- platform: template
  sensors:
    caixa_agua_entrada_total:
      friendly_name: "Entrada Total Caixa"
      unit_of_measurement: "L"
      device_class: water
      value_template: >
        {% set atual = states('sensor.caixa_agua_litros') | float(0) %}
        {% set anterior = states('input_number.caixa_agua_litros_anterior') | float(0) %}
        {% if atual > anterior %}
          {{ (states('sensor.caixa_agua_entrada_total') | float(0)) + (atual - anterior) }}
        {% else %}
          {{ states('sensor.caixa_agua_entrada_total') | float(0) }}
        {% endif %}
```


## 📦 5️⃣ Sensor ACUMULADOR (SAÍDA / CONSUMO)

📄 **sensors.yaml**

```yaml
# ==============================
# Saída total de água da caixa
# ==============================
- platform: template
  sensors:
    caixa_agua_saida_total:
      friendly_name: "Saída Total Caixa"
      unit_of_measurement: "L"
      device_class: water
      value_template: >
        {% set atual = states('sensor.caixa_agua_litros') | float(0) %}
        {% set anterior = states('input_number.caixa_agua_litros_anterior') | float(0) %}
        {% if atual < anterior %}
          {{ (states('sensor.caixa_agua_saida_total') | float(0)) + (anterior - atual) }}
        {% else %}
          {{ states('sensor.caixa_agua_saida_total') | float(0) }}
        {% endif %}
```


## 📦 6️⃣ Utility Meter

📄 **utility_meter.yaml**

```yaml
consumo_caixa_diario:
  source: sensor.caixa_agua_saida_total
  cycle: daily

consumo_caixa_mensal:
  source: sensor.caixa_agua_saida_total
  cycle: monthly

abastecimento_caixa_mensal:
  source: sensor.caixa_agua_entrada_total
  cycle: monthly
```


## 📦 7️⃣ Inputs (configuration.yaml)

```yaml
input_number:
  caixa_agua_litros_anterior:
    name: Caixa d'água Litros Anterior
    min: 0
    max: 10000
    step: 1

  litros_inicio_abastecimento:
    name: Litros início abastecimento
    min: 0
    max: 1000
    step: 1
    unit_of_measurement: "L"

  litros_fim_abastecimento:
    name: Litros fim abastecimento
    min: 0
    max: 1000
    step: 1
    unit_of_measurement: "L"

  tempo_abastecimento_caixa:
    name: Tempo Abastecimento (min)
    min: 0
    max: 300
    step: 1
    unit_of_measurement: min

input_datetime:
  inicio_abastecimento_caixa:
    name: Início Abastecimento
    has_date: true
    has_time: true

  ultimo_abastecimento_caixa:
    name: Último Abastecimento
    has_date: true
    has_time: true
```

## 📦 6️⃣ Automações

📄 **automation.yaml**

```yaml
# ==========================================
# CAIXA D'ÁGUA – CONTROLE COMPLETO
# ==========================================

# ------------------------------------------
# 1️⃣ Atualiza litros anteriores (BASE)
# ------------------------------------------
- id: caixa_atualizar_litros_anteriores
  alias: Caixa - Atualizar litros anteriores
  trigger:
    - platform: state
      entity_id: sensor.caixa_agua_litros
  action:
    - service: input_number.set_value
      target:
        entity_id: input_number.caixa_agua_litros_anterior
      data:
        value: "{{ trigger.from_state.state | float(0) }}"
  mode: single


# ------------------------------------------
# 2️⃣ Detectar INÍCIO do abastecimento
# (subiu mais de 5 litros)
# ------------------------------------------
- id: caixa_inicio_abastecimento
  alias: Caixa - Início do abastecimento
  trigger:
    - platform: state
      entity_id: sensor.caixa_agua_litros
  condition:
    - condition: template
      value_template: >
        {{ trigger.from_state is not none and
           (trigger.to_state.state | float -
            trigger.from_state.state | float) > 5 }}
  action:
    - service: input_datetime.set_datetime
      target:
        entity_id: input_datetime.inicio_abastecimento_caixa
      data:
        timestamp: "{{ now().timestamp() }}"

    - service: input_number.set_value
      target:
        entity_id: input_number.litros_inicio_abastecimento
      data:
        value: "{{ trigger.from_state.state | float }}"
  mode: single


# ------------------------------------------
# 3️⃣ Detectar FIM do abastecimento
# (parou de subir por 5 minutos)
# ------------------------------------------
- id: caixa_fim_abastecimento
  alias: Caixa - Fim do abastecimento
  trigger:
    - platform: state
      entity_id: sensor.caixa_agua_litros
      for:
        minutes: 5
  action:
    - service: input_number.set_value
      target:
        entity_id: input_number.litros_fim_abastecimento
      data:
        value: "{{ states('sensor.caixa_agua_litros') | float }}"

    - service: input_datetime.set_datetime
      target:
        entity_id: input_datetime.ultimo_abastecimento_caixa
      data:
        timestamp: "{{ now().timestamp() }}"

    - service: input_number.set_value
      target:
        entity_id: input_number.tempo_abastecimento_caixa
      data:
        value: >
          {{ ((as_timestamp(now()) -
               as_timestamp(states('input_datetime.inicio_abastecimento_caixa')))
               / 60) | round(0) }}
  mode: single
```





### 📦 8️⃣ (Card Buton Pop UpCompleto)

```yaml
type: custom:button-card
template:
  - base
name: Caixa
entity: sensor.nivel_do_tanque_a_liquid_level
icon: mdi:projector-screen-variant
size: 100%
show_state: false
styles:
  card:
    - background-color: rgba(60, 60, 60, 0.3)
    - border: "0.5px solid #5c5b5b"
  name:
    - font-size: 13px
state:
  - operator: <=
    value: 20
    styles:
      icon:
        - color: "#ff3b30"
  - operator: <=
    value: 50
    styles:
      icon:
        - color: "#ffcc00"
tap_action:
  action: call-service
  service: browser_mod.sequence
  service_data:
    sequence:
      - service: script.pc_click
      - service: browser_mod.popup
        data:
          title: Nível da Caixa
          content:
            type: horizontal-stack
            cards:
              - type: custom:button-card
                styles:
                  card:
                    - width: 70px
                    - background: none
                    - box-shadow: none
                    - border: none
              - type: picture-elements
                image: /local/fundocaixa1.png
                card_mod:
                  style: |
                    ha-card {
                      background: transparent;
                      box-shadow: none;
                      border: none;
                      border-radius: 0;
                      height: 500px;
                      width: 350px;
                      overflow: hidden;
                    }
                elements:
                  - type: custom:bar-card
                    entity: sensor.nivel_do_tanque_a_liquid_level
                    direction: up
                    height: 395px
                    width: 69px
                    min: 0
                    max: 100
                    positions:
                      name: "off"
                      value: "off"
                      icon: "off"
                    style:
                      top: 56%
                      left: 72%
                      transform: translate(-50%, -50%)
                      "--bar-card-border-width": 0
                      "--bar-card-border-radius": 6px
                      "--bar-card-background": transparent
                      "--bar-card-color": "#2f8cff"
                      "--bar-card-padding": 0
                      "--bar-card-margin": 0
                      "--bar-card-shadow": none
                    card_mod:
                      style: |
                        ha-card {
                          background: transparent;
                          box-shadow: none;
                          border: none;
                        }
                  - type: custom:button-card
                    name: Quantidade em Litros
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 20px
                    style:
                      top: 26%
                      left: 29%
                  - type: state-label
                    entity: sensor.caixa_agua_litros
                    style:
                      top: 31%
                      left: 10%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 600
                  - type: custom:button-card
                    name: Consumo Mensal
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 19px
                    style:
                      top: 37%
                      left: 24%
                  - type: state-label
                    entity: sensor.consumo_caixa_mensal
                    style:
                      top: 42%
                      left: 12%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 500
                  - type: custom:button-card
                    name: Consumo Diário
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 19px
                    style:
                      top: 48%
                      left: 22%
                  - type: state-label
                    entity: sensor.consumo_caixa_diario
                    style:
                      top: 53%
                      left: 15%
                      color: "#0a0a0a"
                      font-size: 22px
                      font-weight: 500
                  - type: custom:button-card
                    name: Alerta
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                      name:
                        - color: "#000000"
                        - font-size: 11px
                        - font-weight: 500
                    style:
                      top: 89%
                      left: 89%
                  - type: custom:button-card
                    show_name: false
                    show_state: false
                    custom_fields:
                      circle: |
                        [[[ return `<div style="
                          width:14px;
                          height:14px;
                          border-radius:50%;
                          background:${states['sensor.status_caixa_agua'].state === 'baixo'
                            ? '#ff3b30'
                            : '#d3d3d3'};
                        "></div>`; ]]]
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                    style:
                      top: 93%
                      left: 89%
                  - type: custom:button-card
                    name: Médio
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                      name:
                        - color: "#000000"
                        - font-size: 11px
                        - font-weight: 500
                    style:
                      top: 55%
                      left: 89%
                  - type: custom:button-card
                    show_name: false
                    show_state: false
                    custom_fields:
                      circle: |
                        [[[ return `<div style="
                          width:14px;
                          height:14px;
                          border-radius:50%;
                          background:${states['sensor.status_caixa_agua'].state === 'medio'
                            ? '#ffcc00'
                            : '#d3d3d3'};
                        "></div>`; ]]]
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                    style:
                      top: 59%
                      left: 89%
                  - type: custom:button-card
                    name: Normal
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                      name:
                        - color: "#000000"
                        - font-size: 11px
                        - font-weight: 500
                    style:
                      top: 23%
                      left: 89%
                  - type: custom:button-card
                    show_name: false
                    show_state: false
                    custom_fields:
                      circle: |
                        [[[ return `<div style="
                          width:14px;
                          height:14px;
                          border-radius:50%;
                          background:${states['sensor.status_caixa_agua'].state === 'ok'
                            ? '#34c759'
                            : '#d3d3d3'};
                        "></div>`; ]]]
                    styles:
                      card:
                        - background: transparent
                        - box-shadow: none
                        - border: none
                    style:
                      top: 26%
                      left: 89%
                  - type: custom:button-card
                    name: Água que entrou
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 19px
                    style:
                      top: 59%
                      left: 22%
                  - type: state-label
                    entity: sensor.caixa_agua_entrou
                    style:
                      top: 64%
                      left: 12%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 500
                  - type: custom:button-card
                    name: Água que saiu
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 19px
                    style:
                      top: 69%
                      left: 20%
                  - type: state-label
                    entity: sensor.caixa_agua_saiu
                    style:
                      top: 74%
                      left: 10%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 500
                  - type: custom:button-card
                    name: Tpo. de Abastecimento
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 18px
                    style:
                      top: 78%
                      left: 30%
                  - type: state-label
                    entity: input_number.tempo_abastecimento_caixa
                    style:
                      top: 83%
                      left: 14%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 500
                  - type: custom:button-card
                    name: Último Abastecimento
                    show_icon: false
                    show_state: false
                    styles:
                      card:
                        - background: none
                        - border: none
                      name:
                        - color: "#9aa4af"
                        - font-size: 19x
                    style:
                      top: 88%
                      left: 28%
                  - type: state-label
                    entity: sensor.ultimo_abastecimento_caixa_formatado
                    style:
                      top: 93%
                      left: 25%
                      color: "#0a0a0a"
                      font-size: 19px
                      font-weight: 500
                      white-space: nowrap
                  - type: state-label
                    entity: sensor.nivel_do_tanque_a_liquid_level
                    style:
                      top: 10%
                      left: 94%
                      color: black
                      font-size: 14px
                      font-weight: bold
                      background: rgba(255,255,255,0.6)
                      padding: 3px 6px
                      border-radius: 10px
              - type: custom:button-card
                styles:
                  card:
                    - width: 0px
                    - background: none
                    - box-shadow: none
                    - border: none
                card_mod:
                  style: |
                    ha-card {
                      background-color: transparent;
                      border: none;
                      width: 100%;
                      height: auto;
                    }

```

📦 9️⃣ Card NORMAL (Picture Elements)

```yaml
type: picture-elements
image: /local/fundocaixa9.png
card_mod:
  style: |
    ha-card {
      background: transparent;
      box-shadow: none;
      border: none;
      border-radius: 0;
      height: 475px;
      width: 350px;
      overflow: hidden;
    }
elements:
  - type: custom:bar-card
    entity: sensor.nivel_do_tanque_a_liquid_level
    direction: up
    height: 395px
    width: 69px
    min: 0
    max: 100
    positions:
      name: "off"
      value: "off"
      icon: "off"
    style:
      top: 56%
      left: 72%
      transform: translate(-50%, -50%)
      "--bar-card-border-width": 0
      "--bar-card-border-radius": 6px
      "--bar-card-background": transparent
      "--bar-card-color": "#2f8cff"
      "--bar-card-padding": 0
      "--bar-card-margin": 0
      "--bar-card-shadow": none
    card_mod:
      style: |
        ha-card {
          background: transparent;
          box-shadow: none;
          border: none;
        }
  - type: custom:button-card
    name: Quantidade em Litros
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 20px
    style:
      top: 26%
      left: 29%
  - type: state-label
    entity: sensor.caixa_agua_litros
    style:
      top: 31%
      left: 10%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 600
  - type: custom:button-card
    name: Consumo Mensal
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 19px
    style:
      top: 37%
      left: 24%
  - type: state-label
    entity: sensor.consumo_caixa_mensal
    style:
      top: 42%
      left: 12%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500
  - type: custom:button-card
    name: Consumo Diário
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 19px
    style:
      top: 48%
      left: 22%
  - type: state-label
    entity: sensor.consumo_caixa_diario
    style:
      top: 53%
      left: 12%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500

  - type: custom:button-card
    name: Alerta
    show_icon: false
    show_state: false
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
      name:
        - color: "#000000"
        - font-size: 11px
        - font-weight: 500
    style:
      top: 89%
      left: 89%
  - type: custom:button-card
    show_name: false
    show_state: false
    custom_fields:
      circle: |
        [[[ return `<div style="
          width:14px;
          height:14px;
          border-radius:50%;
          background:${states['sensor.status_caixa_agua'].state === 'baixo'
            ? '#ff3b30'
            : '#d3d3d3'};
        "></div>`; ]]]
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
    style:
      top: 93%
      left: 89%
  - type: custom:button-card
    name: Médio
    show_icon: false
    show_state: false
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
      name:
        - color: "#000000"
        - font-size: 11px
        - font-weight: 500
    style:
      top: 55%
      left: 89%
  - type: custom:button-card
    show_name: false
    show_state: false
    custom_fields:
      circle: |
        [[[ return `<div style="
          width:14px;
          height:14px;
          border-radius:50%;
          background:${states['sensor.status_caixa_agua'].state === 'medio'
            ? '#ffcc00'
            : '#d3d3d3'};
        "></div>`; ]]]
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
    style:
      top: 59%
      left: 89%
  - type: custom:button-card
    name: Normal
    show_icon: false
    show_state: false
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
      name:
        - color: "#000000"
        - font-size: 11px
        - font-weight: 500
    style:
      top: 23%
      left: 89%
  - type: custom:button-card
    show_name: false
    show_state: false
    custom_fields:
      circle: |
        [[[ return `<div style="
          width:14px;
          height:14px;
          border-radius:50%;
          background:${states['sensor.status_caixa_agua'].state === 'ok'
            ? '#34c759'
            : '#d3d3d3'};
        "></div>`; ]]]
    styles:
      card:
        - background: transparent
        - box-shadow: none
        - border: none
    style:
      top: 26%
      left: 89%




      
  - type: custom:button-card
    name: Água que entrou
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 19px
    style:
      top: 59%
      left: 22%
  - type: state-label
    entity: sensor.caixa_agua_entrou
    style:
      top: 64%
      left: 12%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500
  - type: custom:button-card
    name: Água que saiu
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 19px
    style:
      top: 69%
      left: 20%
  - type: state-label
    entity: sensor.caixa_agua_saiu
    style:
      top: 74%
      left: 10%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500
  - type: custom:button-card
    name: Tpo. de Abastecimento
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 18px
    style:
      top: 78%
      left: 30%
  - type: state-label
    entity: input_number.tempo_abastecimento_caixa
    style:
      top: 83%
      left: 13%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500
  - type: custom:button-card
    name: Último Abastecimento
    show_icon: false
    show_state: false
    styles:
      card:
        - background: none
        - border: none
      name:
        - color: "#9aa4af"
        - font-size: 19x
    style:
      top: 88%
      left: 28%

  - type: state-label
    entity: sensor.ultimo_abastecimento_caixa_formatado
    style:
      top: 93%
      left: 25%
      color: "#0a0a0a"
      font-size: 19px
      font-weight: 500
      white-space: nowrap

    
  - type: state-label
    entity: sensor.nivel_do_tanque_a_liquid_level
    style:
      top: 10%
      left: 94%
      color: black
      font-size: 14px
      font-weight: bold
      background: rgba(255,255,255,0.6)
      padding: 3px 6px
      border-radius: 10px


```


E é isso.
Com esse projeto você consegue saber quanto de água tem na caixa, quanto entrou, quanto saiu e quanto você consumiu de verdade, tudo integrado ao Home Assistant usando sensores Tuya.

Se ficou alguma dúvida, deixa nos comentários.
Se esse conteúdo te ajudou, deixa o like e se inscreve no canal, porque tem mais projetos assim vindo aí.
