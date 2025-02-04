```yaml
sensor:
  - platform: template
    sensors:
      bom_current_text:
        value_template: >
            {% set val = states('sensor.kariong_short_text_0').split('.')[0] %} 
            {{ val | title }}

      uv_cat_formatted:
        value_template: "{{ states('sensor.kariong_uv_category_0') | replace('veryhigh', 'Very High') | title }}"

      bom_uv_alert:
        value_template: >
            UV Today: Sun Protection 
            {{ as_timestamp(states('sensor.kariong_uv_start_time_0'),default='n/a') | timestamp_custom(' %I:%M%p',default='n/a') | lower | replace(" 0", "") }} to {{ as_timestamp(states('sensor.kariong_uv_end_time_0'),default='n/a') | timestamp_custom(' %I:%M%p',default='n/a') | lower | replace(" 0", "") }}, UV Index predicted to reach {{ states('sensor.kariong_uv_max_index_0') }} [{{ states('sensor.uv_cat_formatted') }}]

      bom_fire_danger:
        value_template: "Fire Danger Today: {{ states('sensor.kariong_fire_danger_0') }}"

# Beaufort
# https://en.wikipedia.org/wiki/Beaufort_scale
      beaufort:
        value_template: >
            {%- if states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 118 -%}
            12
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 103 -%}
            11
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 89 -%}
            10
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 75 -%}
            9
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 62 -%}
            8
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 50 -%}
            7
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 39 -%}
            6
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 29 -%}
            5
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 20 -%}
            4
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 12 -%}
            3
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 6 -%}
            2
            {%- elif states('sensor.gosford_wind_speed_kilometre') | float(default='n/a')  >= 2 -%}
            1
            {%- else -%}
            0
            {%- endif -%}

# Heatindex
# https://en.wikipedia.org/wiki/Heat_index
      heatindex:
        unit_of_measurement: °C
        device_class: temperature
        value_template: >
            {%- if states('sensor.gosford_temp') | float(default=0) > 27 and states('sensor.gosford_humidity') | float(default=0) > 40 -%}
            {% set T = states('sensor.gosford_temp') | float(default='n/a') %}
            {% set R = states('sensor.gosford_humidity') | float(default='n/a') %}
            {% set c1 = -8.78469475556 %}
            {% set c2 = 1.61139411 %}
            {% set c3 = 2.33854883889 %}
            {% set c4 = -0.14611605 %}
            {% set c5 = -0.012308094 %}
            {% set c6 = -0.0164248277778 %}
            {% set c7 = 0.002211732 %}
            {% set c8 = 0.00072546 %}
            {% set c9 = -0.000003582 %}
            {% set HI = c1 + (c2 * T ) + (c3 * R) + ( c4 * T * R ) + ( c5 * T**2 ) + ( c6 * R**2 ) + ( c7 * T**2 * R ) + ( c8 * T * R**2 ) + ( c9 * T**2 * R**2 ) %} 
            {{ HI | round }}
            {%- else -%}
            n/a
            {%- endif -%}
      heatindexrating:
        value_template: >
            {%- if states('sensor.heatindex') == 'n/a' -%}
            Out of range
            {%- elif states('sensor.heatindex') | float(default='n/a')  >= 54 -%}
            Extreme danger: heat stroke imminent
            {%- elif states('sensor.heatindex') | float(default='n/a')  >= 41 -%}
            Danger: cramps, exhaustion heat stroke probable
            {%- elif states('sensor.heatindex') | float(default='n/a')  >= 32 -%}
            Extreme caution: cramps and exhaustion possible
            {%- elif states('sensor.heatindex') | float(default='n/a')  >= 26 -%}
            Caution: fatigue possible
            {%- else -%}
            Normal
            {%- endif -%}

      bom_forecast_0:
        friendly_name: "Today"
        value_template: >
          {% if states('sensor.kariong_temp_min_0') == 'unknown' %} {% set min = states('sensor.bom_today_min') %} {% else %} {% set min = states('sensor.kariong_temp_min_0') %} {% endif %}
          {% if states('sensor.kariong_temp_max_0') == 'unknown' %} {% set max = states('sensor.bom_today_max') %} {% else %} {% set max = states('sensor.kariong_temp_max_0') %} {% endif %}
          {{ max|round(0,default='none')}}°/{{ min|round(0,default='none')}}°/{{states('sensor.kariong_rain_chance_0')|round(0,default='none')}}%
        entity_picture_template: >-
          {%- if states('sun.sun') == 'below_horizon' and (states('sensor.kariong_icon_descriptor_0') == 'fog' or states('sensor.kariong_icon_descriptor_0') == 'haze' or states('sensor.kariong_icon_descriptor_0') == 'hazy' or states('sensor.kariong_icon_descriptor_0') == 'light-showers' or states('sensor.kariong_icon_descriptor_0') == 'partly-cloudy' or states('sensor.kariong_icon_descriptor_0') == 'showers' or states('sensor.kariong_icon_descriptor_0') == 'shower' or states('sensor.kariong_icon_descriptor_0') == 'light_showers' or states('sensor.kariong_icon_descriptor_0') == 'light_shower' or states('sensor.kariong_icon_descriptor_0') == 'partly_cloudy' or states('sensor.kariong_icon_descriptor_0') == 'mostly_sunny') -%}
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_0') ~ '-night.png' }}
          {%- else -%}
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_0') ~ '.png' }}
          {%- endif -%}

      bom_forecast_1:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (1 * 86400 ) -%}
          {{ date | timestamp_custom('Tomorrow (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_1')|round(0)}}°/{{states('sensor.kariong_temp_min_1')|round(0)}}°/{{states('sensor.kariong_rain_chance_1')|round(0)}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_1') ~ '.png' }}

      bom_forecast_2:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (2 * 86400 ) -%}
          {{ date | timestamp_custom('%A (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_2')|round(0)}}°/{{states('sensor.kariong_temp_min_2')|round(0)}}°/{{states('sensor.kariong_rain_chance_2')|round(0)}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_2') ~ '.png' }}

      bom_forecast_3:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (3 * 86400 ) -%}
          {{ date | timestamp_custom('%A (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_3')|round(0)}}°/{{states('sensor.kariong_temp_min_3')|round(0)}}°/{{states('sensor.kariong_rain_chance_3')|round(0)}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_3') ~ '.png' }}

      bom_forecast_4:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (4 * 86400 ) -%}
          {{ date | timestamp_custom('%A (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_4')|round(0)}}°/{{states('sensor.kariong_temp_min_4')|round(0)}}°/{{states('sensor.kariong_rain_chance_4')|round(0)}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_4') ~ '.png' }}

      bom_forecast_5:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (5 * 86400 ) -%}
          {{ date | timestamp_custom('%A (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_5')|round(0)}}°/{{states('sensor.kariong_temp_min_5')|round(0)}}°/{{states('sensor.kariong_rain_chance_5')|round(0)}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_5') ~ '.png' }}

      bom_forecast_6:
        friendly_name_template: >
          {%- set date = as_timestamp(now(),default='n/a') + (6 * 86400 ) -%}
          {{ date | timestamp_custom('%A (%-d/%-m)',default='n/a') }}
        value_template: >
          {{states('sensor.kariong_temp_max_6')|round(0,default='unknown')}}°/{{states('sensor.kariong_temp_min_6')|round(0,default='unknown')}}°/{{states('sensor.kariong_rain_chance_6')|round(0,default='unknown')}}%
        entity_picture_template: >-
          {{ '/local/icons/bom_icons/' ~ states('sensor.kariong_icon_descriptor_6') ~ '.png' }}

# ONLY need these templates if you use the Average sensor
# ONLY USE ONE bom_today_max below:
      bom_today_max:
        value_template: >
          {{ state_attr('sensor.today_temp_bom', 'max_value') }}

#      bom_today_max:
#        value_template: >
#          {%- if states('sensor.kariong_temp_max_0') == 'n/a' -%} 
#            {{ state_attr('sensor.today_temp_bom', 'max_value') }}
#          {% else %}
#            {{ states('sensor.kariong_temp_max_0') }}
#          {% endif %}

# ONLY need these templates if you use the Average sensor
# ONLY USE ONE bom_today_min below:
      bom_today_min:
        value_template: >
          {{ state_attr('sensor.today_temp_bom', 'min_value') }}

#      bom_today_min:
#        value_template: >
#          {%- if states('sensor.kariong_temp_min_0') == 'n/a' -%} 
#            {{ state_attr('sensor.today_temp_bom', 'min_value') }}
#          {% else %}
#            {{ states('sensor.kariong_temp_min_0') }}
#          {% endif %}

# IMPORTANT NOTE IF YOU USE average, you must comment out the below statistics sensor. Both cannot exist
# https://github.com/Limych/ha-average
  - platform: average
    name: today_temp_bom
    entities:
      - sensor.gosford_temp
    start: '{{ now().replace(hour=0).replace(minute=0).replace(second=0) }}'
    end: '{{ now() }}'

# IMPORTANT NOTE IF YOU USE statistics, you must comment out the above average sensor and min/max templates. Both cannot exist
# You don't need the below templates to extract min/max temp if you use the statistics sensors
#  - platform: statistics
#    name: today_temp_bom_min
#    sampling_size: 150
#    entity_id: sensor.gosford_temp
#    state_characteristic: value_min
#    max_age:
#      hours: 24

#  - platform: statistics
#    name: today_temp_bom_max
#    sampling_size: 150
#    entity_id: sensor.gosford_temp
#    state_characteristic: value_max
#    max_age:
#      hours: 24
