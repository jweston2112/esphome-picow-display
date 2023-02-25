# rpipicow-esphome
Raspberry PI Pico W running ESPHome using TFT_eSPI

## Installation
You will first need to do a manual installation by putting the pico.yaml file into your esphome folder then using the modern format in ESPHome to get a local copy of the firmware and finally use https://web.esphome.io/ to install over USB.

### Method
Download a copy of this code and place the picodisplay folder in your esphome folder. Also place the pico.yaml and secrets.yaml into the esphome folder, ensuring to change the pico.yaml api and secrets.yaml details to your own credentials.

From your ESPHome dashboard, create a local copy of the s3 firmware by clicking the three dots > Install > Manual Download > Modern Format

To set the board into flash mode, hold the button on the left of the USB port while plugging in the USB cable that is connected to your desktop machine.

Navigate to https://web.esphome.io/ in a compatible browser (Chrome, Edge etc.) and click "Connect". Select the corresponding COM Port and click connect again. 

When the board has connected, click "install" and select the firmware that you have created, this will erase the board and write the new firmware.

When the firmware has been written to the board you will need to unplug and reconnect the USB cable.

Also modify the User_Setup_Select.h to include the Hardware Setup Indformation
```c++
//#include <User_Setup.h>           // Default setup is root library folder
#include <User_Setups/HackerBox_Weather_Pi.h>
```
## Expanding on this for use in Home Assistant
In a file editor, you will need to move the line ```TFT_eSPI tft = TFT_eSPI();``` from the Private: section at the bottom of /esphome/picodisplays/tft_espi_display.h
And move it to under the Public section towards the top of the file as so:

	double barSize = 0;
	public:
	TFT_eSPI tft = TFT_eSPI();
	std::string time = "init";}

Then save the file. The next time you edit / compile and upload to the pico w board it will expose the tft class to esphome. You can then just use the standard tft_espi drawing functions (not the typical esphome "display:" section / drawing components). (Thanks  @jamesarm97)

To diplay a string of text from a text sensor add the sensor to your pico.yaml (make sure to edit the entity to match your own setup):

```
text_sensor:
  - platform: homeassistant
    id: kitchenmusictrack
    entity_id: media_player.kitchen_display
    attribute: media_title
  - platform: homeassistant
    id: kitchenmusicartist
    entity_id: media_player.kitchen_display
    attribute: media_artist
```

And amend your pico.yaml file to call these in the lambda script:
```
interval:
  - interval: 1s
    then:
    - lambda: |-
        displayControllerComponent->clear();
        displayControllerComponent->tft.setFreeFont(FMB12);
        displayControllerComponent->tft.drawString(id(kitchenmusicartist).state.c_str(),10,105);
        displayControllerComponent->tft.drawString(id(kitchenmusictrack).state.c_str(),10,125);
```
Save the pico.yaml file, create another Modern Format firmware, put the board into flash mode upload, the code and finally unplug and re-connect the USB cable. Assuming you have used the correct credentials, you can now power the T-Display and as long as it is in Wifi range of your router, will display the (in the case of this example) Artist and Title of the song playing on the media player.
