# To Train Donkeycar With Xbox One Bluetooth Controller

[**(中文)**](./README_CN.md)

[Donkeycar](https://github.com/autorope/donkey2) is a minimalist version of autonomous driving platform using machine learning capability to auto-drive a RC transportation vehicle. It is designed to control a RC car but could be extended into other type of transportation control applications.

The original development of the training is based on a web control mechanism and then a [bluetooth controller](https://github.com/autorope/donkeypart_bluetooth_game_controller) "part" was developed. The official 'part' from the donkeycar repository is designed for Nintendo Wiiu controller as default while I have already got an Xbox One bluetooth controller. So to develop an Xbox controller version of the part became a natual decison.

It turned out not an easy task as the command structure of the Xbox One controller is very much different from the command structure commonly used with Wiiu or PS3/4. My friend Chris as a passionate maker was able to crack the code and make the Donkeycar trained successfully with a Xbox One controller and offered his code for me to use. As I plan to build more controller-enabled robots in future I decided to do my own homework thjis time to understand the details of an Xbox One Bluetooth Controller's command structure and how to make it work with an Linux-based SOC board like Raspberry Pi or BeagleBone. Here is the step by step process and the learning with a few important tricks to get it work.

First, here is the modified [config.yml](https://github.com/autorope/donkeypart_bluetooth_game_controller) file used in donkeycar project based on the data collected from Xbox One Bluetooth controller as an input device on Raspberry Pi 3:

**xbox_config.yml:**

```yml

device_search_term: 'xbox'


#Map the button codes to the button names

button_map:

  0x00: 'LEFT_STICK_X'
  0x01: 'LEFT_STICK_Y'
  0x02: 'RIGHT_STICK_X'
  0x05: 'RIGHT_STICK_Y'
  #right trigger
  0x09: 'RIGHT_BOTTOM_TRIGGER'
  #left trigger
  0x0a: 'LEFT_BOTTOM_TRIGGER'
  #all button pressed
  0x04: 'BUTTON'
  
  #D-PAD LEFT -0.00078125, D-PAD RIGHT  0.0078125  
  0x10: 'PAD_RIGHT_LEFT'
  #D-PAD UP -0.0078125, D-PAD DOWN 0.0078125
  0x11: 'PAD_UP_DOWN'

  # 'LEFT_TOP_TRIGGER' #left shoulder 589829
  # 'RIGHT_TOP_TRIGGER' #right shoulder 589830
  # 'SELECT' # no such button on xbox, use button A instead
  # 'BACK' #589831
  # 'START' # 589832
  # 'LEFT_STICK_PRESS' #589833
  # 'RIGHT_STICK_PRESS' #589834
  # 'A' #589825
  # 'B' #589826
  # 'X' #589827
  # 'Y' #589828
  # 'Xbox' #786979

joystic_max_value: 65535
# joystic_max_value: 1280
```

**part.py:** major modification for Xbox One Bluetooth controller is as below,

```python
            '''
            Adding customized button:value dictionary for xbox one bluetooth controller
            as it provide one constant button event index of '0x04' for all button pressing event and use value to differentiate the button type. This 'button:value' is collected with the built-in tool from Donkeycar bluetooth controller part. 
            '''

            # if BUTTON
            evt_pressing_btn_map = {
                0x90001: 'A',
                0x90002: 'B',
                0x90003: 'X',
                0x90004: 'Y',
                0x90005: 'LEFT_TOP_TRIGGER',
                0x90006: 'RIGHT_TOP_TRIGGER',
                0x90007: 'BACK',
                0x90008: 'START',
                0x90009: 'LEFT_STICK_PRESS',
                0x9000a: 'RIGHT_STICK_PRESS',
                0xc0223: 'XBOX'
            }

            # if PAD_RIGHT_LEFT
            evt_pressing_pad_l_r_map = {
                1: 'PAD_RIGHT',
                -1: 'PAD_LEFT'
            }

            # if PAD_UP_DOWN
            evt_pressing_pad_u_d_map = {
                1: 'PAD_DOWN',
                -1: 'PAD_UP'
            }

            if btn == 'BUTTON':
                btn = evt_pressing_btn_map.get(val, 'UNDEFINED')
                return btn, val
            
            if btn == 'PAD_UP_DOWN':
                btn = evt_pressing_pad_u_d_map.get(val, 'CENTER')
                return btn, val

            if btn == 'PAD_RIGHT_LEFT':
                btn = evt_pressing_pad_l_r_map.get(val, 'CENTER')
                return btn, val

            '''
            The axis output value format of xbox one bluetooth controller is different from Wiiu or PS3/4.
            The value range is between 0-65535 for the Left and Right Stick movement and 0-1023 for the Left and Right Bottom Triggers.
            '''	
            
            if btn == 'LEFT_BOTTOM_TRIGGER' or btn == 'RIGHT_BOTTOM_TRIGGER':
                val = float(val/1023)
                return btn, val
            else:
                val = float(val-32767)/32768
            return btn, val
```

Just clone the original Bluetooth Controller repo and replaced the part.py with the modified version from this repo and copy the xbox_config.yml file into the [donkeypart_bluetooth_game_controller/donkeypart_bluetooth_game_controller/](https://github.com/autorope/donkeypart_bluetooth_game_controller/tree/master/donkeypart_bluetooth_game_controller). Don't need to delete the config file for Wiiu as I have change the config file look-up setup in part.py to the new xbox_config.yml file already.

Now you need to get your Xbox One Bluetooth controller connected with Raspberry Pi which is another challenge. The instruction to come. 

After all this hassles, you can finally start to use Xbox One Bluetooth controller just as Wiiu or PS3/4 in your Donkeycar project.

Have fun!
