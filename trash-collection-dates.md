# Trash collection dates

Up to date as of 

## Step 1 - Add calendar

Go to `/config/integrations/dashboard` and add "Local Calendar", select "Create an empty calendar" and name it "trash".

Add events with `Summary` equal to `collection`.

## Step 2 - Add helpers

Go to `/config/helpers` and add `Template`, select `Template a sensor`.

Name it `Next Trash`

You can use icon `mdi:trash-can`.

## Step 3 - Add automation to update the days

```yaml
- id: '1234'
  alias: Update collection
  description: ''
  triggers:
  - at: 00:10:00
    trigger: time
  - event: start
    trigger: homeassistant
  - entity_id: calendar.trash
    trigger: state
  conditions: []
  actions:
  - target:
      entity_id: calendar.trash
    data:
      start_date_time: '{{ now() }}'
      end_date_time: '{{ now() + timedelta(days=31) }}'
    response_variable: cal
    action: calendar.get_events
  - variables:
      next_general: "{% set ns=namespace(dt=None) %} {% for e in cal['calendar.trash'].events
        %}\n  {% if e.summary.lower() == 'zmieszane' %}\n    {% set d = as_datetime(e.start)
        %}\n    {% if ns.dt is none or d < ns.dt %}{% set ns.dt=d %}{% endif %}\n
        \ {% endif %}\n{% endfor %} {{ ns.dt }}"
  - if:
    - condition: template
      value_template: '{{ next_general is not none }}'
    then:
    - target:
        entity_id: input_datetime.next_trash_general_dt
      data:
        datetime: '{{ as_datetime(next_general) | as_local | string }}'
      action: input_datetime.set_datetime
  mode: single
```

## Step 4 - Add semsors

```yaml
template:
  - sensor:
      - name: Next Trash
        unique_id: next_trash_general
        icon: mdi:trash-can
        state: >-
          {% set d = states('input_datetime.next_trash_general_dt') %}
          {% if d not in ['unknown','unavailable','none'] %}
            {% set dt = as_datetime(d) %}
            {% set days = (dt.date() - now().date()).days %}
            {{ days }} days - {{ dt.strftime('%Y-%m-%d') }}
          {% else %}
            error
          {% endif %}
```

## Enjoy

In the example 3 different dates were added:

<img width="481" height="228" alt="image" src="https://github.com/user-attachments/assets/5ae47d94-1136-45ac-91a9-1d56fb653363" />

