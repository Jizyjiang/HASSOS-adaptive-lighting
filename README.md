# HASS OS 自动适应灯光照明组件

![](https://github.com/home-assistant/brands/raw/b4a168b9af282ef916e120d31091ecd5e3c35e66/core_integrations/adaptive_lighting/icon.png)

_Try out this code by adding https://github.com/basnijholt/adaptive-lighting to your custom repos in [HACS (Home Assistant Community Store)](https://hacs.xyz/) and install it!_


adaptive_lighting组件可以在一天中更改灯光设置。它使用太阳的位置来计算最适合一天中该时间的色温和亮度。科学研究表明，这有助于维持你的自然昼夜节律（你的生物钟），并可能改善睡眠、情绪和总体幸福感。

实际上，这意味着太阳落山后，灯光的亮度将降低到某个最低亮度，而色温将在中午处于最冷的色温，之后会降低，并在日落时达到最暖的颜色。
在日出前后，相反的情况会发生。

此外，该模块还提供了一个“睡眠模式”下定义和设置灯光的方法。
当启用“睡眠模式”时，灯光将处于最低亮度，并具有非常温暖的颜色。

本集成拥有4个开关(在本例中，组件的名称为`"living_room"`):
1. `switch.adaptive_lighting_living_room`, 这将打开或关闭自适应照明集成。它有几个属性显示当前灯光设置。
2. `switch.adaptive_lighting_sleep_mode_living_room`, 激活后，将打开“睡眠模式”（您可以设置特定的“sleep_brightness` 和 `sleep_color_temp`).
3. `switch.adaptive_lighting_adapt_brightness_living_room`, 设置集成是否应调整灯光亮度（如果灯光支持）。
4. `switch.adaptive_lighting_adapt_color_living_room`,设置集成是否应调整灯光的色温（如果灯光支持）

## Taking back control

Although having your lights automatically adapt is great most of the time, there might be times at which you want to set the lights to a different color/brightness and keep it that way.
For this purpose, the integration (when `take_over_control` is enabled) automatically detects whether someone (e.g., person toggling the light switch) or something (automation) changes the lights.
If this happens *and* the light is already on, the light that was changed gets marked as "manually controlled" and the Adaptive Lighting component will stop adapting that light until it turns off and on again (or if you use the service call `adaptive_lighting.set_manual_control`).
This mechanism works by listening to all `light.turn_on` calls that change the color or brightness and by noting that the component did not make the call.
Additionally, there is an option to detect all state changes (when `detect_non_ha_changes` is enabled), so also changes to the lights that were not made by a `light.turn_on` call (e.g., through an app or via something outside of Home Assistant.)
It does this by comparing a light's state to Adaptive Lighting's previously used settings.
Whenever a light gets marked as "manually controlled", an `adaptive_lighting.manual_control` event is fired, such that one can use this information in automations.

## 配置

This integration is both fully configurable through YAML _and_ the frontend. (**Configuration** -> **Integrations** -> **Adaptive Lighting**, **Adaptive Lighting** -> **Options**)
Here, the options in the frontend and in YAML have the same names.

```yaml
# Example configuration.yaml entry
adaptive_lighting:
  lights:
    - light.living_room_lights
```

### 选项
| 选项                | 描述                                                                                                                                                                                                                   | 必须要求   | 默认   | 类型    |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|-----------|---------|
|name                  | 显示此开关时要使用的名称。                                                                                                                                                                                 | False      | default   | string  |
| light                | 自适应照明需要控制的光实体列表（可能为空）。                                                                                                                                                      | False      | list      | []      |
| prefer_rgb_color      |是否使用RGB色调代替原生光色温                                                                                                                                                | False      | False     | boolean |
| initial_transition    | How long the first transition is when the lights go from `off` to `on` (or when "sleep mode" is toggled).                                                                                                                     | False      | 1         | time    |
| transition            | How long the transition is when the lights change, in seconds.                                                                                                                                                                | False      | 45        | integer |
| interval              | How often to adapt the lights, in seconds.                                                                                                                                                                                    | False      | 90        | integer |
| min_brightness        | 设置灯光的最低亮度百分比。                                                                                                                                                                       | False      | 1         | integer |
| max_brightness        | 设置灯光的最大亮度百分比。                                                                                                                                                                       | False      | 100       | integer |
| min_color_temp        | The warmest color temperature to set the lights to, in Kelvin.                                                                                                                                                                | False      | 2000      | integer |
| max_color_temp        | The coldest color temperature to set the lights to, in Kelvin.                                                                                                                                                                | False      | 5500      | integer |
| sleep_brightness      | 睡眠模式时的灯光亮度。                                                                                                                                                                        | False      | 1         | integer |
| sleep_color_temp      | 睡眠模式时的灯光色温。                                                                                                                                                                  | False      | 1000      | integer |
| sunrise_time          | 用固定的时间覆盖日出时间。                                                                                                                                                                                 | False      | time      |         |
| sunrise_offset        | C用正负偏移改变日出时间。                                                                                                                                                                   | False      | 0         | time    |
| sunset_time           | 用固定的时间覆盖日落时间。                                                                                                                                                                               | False      | time      |         |
| sunset_offset         | 用正负偏移改变日落时间。                                                                                                                                                                    | False      | 0         | time    |
| only_once             | 是继续调整灯（False），还是只在灯打开后立即调整（true）).                                                                                                                 | False      | False     | boolean |
| take_over_control     | 如果另一个源在灯亮起并正在适应时调用light.turn_on，请禁用自适应照明。                                                                                                                 | False      | True      | boolean |
| detect_non_ha_changes | Whether to detect state changes and stop adapting lights, even not from `light.turn_on`. Needs `take_over_control` to be enabled. Note that by enabling this option, it calls 'homeassistant.update_entity' every 'interval'! | False  | False     | boolean |
| separate_turn_on_commands | Whether to use separate `light.turn_on` calls for color and brightness, needed for some types of lights | False | False | boolean |

完整实例:

```yaml
# Example configuration.yaml entry
adaptive_lighting:
- name: "default"
  lights: []
  prefer_rgb_color: false
  transition: 45
  initial_transition: 1
  interval: 90
  min_brightness: 1
  max_brightness: 100
  min_color_temp: 2000
  max_color_temp: 5500
  sleep_brightness: 1
  sleep_color_temp: 1000
  sunrise_time: "08:00:00"  # override the sunrise time
  sunrise_offset:
  sunset_time:
  sunset_offset: 1800  # in seconds or '00:15:00'
  take_over_control: true
  detect_non_ha_changes: false
  only_once: false

```

### 服务

`adaptive_lighting.apply` 将自适应照明设置应用于按需照明。

| Service data attribute    | Optional | Description                                                             |
|---------------------------|----------|-------------------------------------------------------------------------|
| `entity_id`               |       no | 带有要应用设置的开关的 `entity_id`。      |
| `lights`                  |       no | 应用设置的灯（或灯列表）。                   |
| `transition`              |      yes | 过渡的秒数。                        |
| `adapt_brightness` | yes | 是否改变光线的亮度。                                       |
| `adapt_color`      | yes | 是否对配套灯进行颜色的调整。                                            |
| `prefer_rgb_color` | yes | 是否尽可能选择RGB颜色调整而不是本色温度。 |
| `turn_on_lights`   | yes |是否打开当前关闭的灯。                                          |

`adaptive_lighting.set_manual_control` can mark (or unmark) whether a light is "manually controlled", meaning that when a light has `manual_control`, the light is not adapted.

| Service data attribute | Optional | Description                                                                                                                          |
|------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------|
| `entity_id`            |       no | The `entity_id` of the switch in which to (un)mark the light as being "manually controlled".                                         |
| `lights`               |       no | A light (or list of lights) to apply the settings to.                                                                                |
| `manual_control`       |      yes | Whether to mark (true) or unmark (false) the light as "manually controlled", when not specified it selects all lights in the switch. |


## Automation examples

Reset the `manual_control` status of a light after an hour.
```yaml
- alias: "Adaptive lighting: reset manual_control after 1 hour"
  mode: parallel
  trigger:
    platform: event
    event_type: adaptive_lighting.manual_control
  variables:
    light: "{{ trigger.event.data.entity_id }}"
    switch: "{{ trigger.event.data.switch }}"
  action:
    - delay: "01:00:00"
    - condition: template
      value_template: "{{ light in state_attr(switch, 'manual_control') }}"
    - service: adaptive_lighting.set_manual_control
      data:
        entity_id: "{{ switch }}"
        lights: "{{ light }}"
        manual_control: false
```

Toggle multiple Adaptive Lighting switches to "sleep mode" using an `input_boolean.sleep_mode`.

```yaml
- alias: "Adaptive lighting: toggle 'sleep mode'"
  trigger:
    - platform: state
      entity_id: input_boolean.sleep_mode
    - platform: homeassistant
      event: start  # in case the states aren't properly restored
  variables:
    sleep_mode: "{{ states('input_boolean.sleep_mode') }}"
  action:
    service: "switch.turn_{{ sleep_mode }}"
    entity_id:
      - switch.adaptive_lighting_sleep_mode_living_room
      - switch.adaptive_lighting_sleep_mode_bedroom
```

# Other

See the documentation of the PR at https://deploy-preview-14877--home-assistant-docs.netlify.app/integrations/adaptive_lighting/ and [this video on Reddit](https://www.reddit.com/r/homeassistant/comments/jabhso/ha_has_it_before_apple_has_even_finished_it_i/) to see how to add the integration and set the options.

This integration was originally based of the great work of @claytonjn https://github.com/claytonjn/hass-circadian_lighting, but has been 100% rewritten and extended with new features.

# Having problems?
Please enable debug logging by putting this in `configuration.yaml`:
```yaml
logger:
  default: warning
  logs:
    custom_components.adaptive_lighting: debug
```
and after the problem occurs please create an issue with the log (`/config/home-assistant.log`).


### Graphs!
These graphs were generated using the values calculated by the Adaptive Lighting sensor/switch(es).

##### Sun Position:
![cl_percent|690x131](https://community-home-assistant-assets.s3.dualstack.us-west-2.amazonaws.com/original/3X/6/5/657ff98beb65a94598edeb4bdfd939095db1a22c.PNG)

##### Color Temperature:
![cl_color_temp|690x129](https://community-home-assistant-assets.s3.dualstack.us-west-2.amazonaws.com/original/3X/5/9/59e84263cbecd8e428cb08777a0413672c48dfcd.PNG)

##### Brightness:
![cl_brightness|690x130](https://community-home-assistant-assets.s3.dualstack.us-west-2.amazonaws.com/original/3X/5/8/58ebd994b62a8b1abfb3497a5288d923ff4e2330.PNG)

# Maintainers

- @basnijholt
- @RubenKelevra

