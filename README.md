## ESP32_D0WD_Q6 Patch ESP-IDF v5.2 in LINUX (UBUNTU)
# Introduction
ESP32 chips do have a new function esp_wifi_80211_tx(), which is used to send router raw frames.

# Problem
They made a function to filter frames that may be illegal to send like deauthentification or dissociation frames, sadly this filter sits inside their precompiled library and can't be modified in c code and I saw many guys reporting all this over forums that c0 frame not working.
```
E (1084305) wifi:unssuported frame type: 0c0
```
# Solution
Spent several hours analyzing decompiled library which is libnet80211.a
lib has object files merged together
Function esp_wifi_80211_tx() sits here
libnet80211.a -> ieee80211_output.o
So I disassembled and edited ieee80211_output.o so that sanity check would be ignored and you can now send any packet

# How to setup
1. Download 'libnet80211.a' from this repo (current one compiled using ESP-IDF v5.2)
2. Copy to folder $your_esp_location/esp/esp-idf/components/esp_wifi/lib/esp32/ and replace older one
3. Try compile your project now, if it not compilling, you can try manually inject ieee80211_output.o to your existing libnet80211.a (previous one)
4. In that case download 'ieee80211_output.o' from this repo
5. Copy to folder $your_esp_location/esp/esp-idf/components/esp_wifi/lib/esp32/
6. Run Following commands
```
cd $your_esp_location/esp/esp-idf/components/esp_wifi/lib/esp32/
ar rcs libnet80211.a ieee80211_output.o
```
7. Now it should be injected, try compile your project
8. If still cannot compile, it means something changed inside ieee80211_output.o due newer espressif version
# Results
Frame sending deauth worked :)
![Screenshot from 2023-06-25 23-57-08](https://github.com/Hex2424/esp32_deauth_patch/assets/81779693/1a9236da-a9d7-4e05-bbda-178871e16f5e)
![Screenshot from 2023-06-25 23-57-35](https://github.com/Hex2424/esp32_deauth_patch/assets/81779693/29828edc-40e1-4c7c-b238-422a0a9277dd)
![Screenshot from 2023-06-25 23-58-15](https://github.com/Hex2424/esp32_deauth_patch/assets/81779693/e248af49-37af-4108-8c09-894b6f31dc5d)
![Screenshot from 2023-06-25 23-59-26](https://github.com/Hex2424/esp32_deauth_patch/assets/81779693/6633013a-3c01-4454-ac5f-85ad5bad45c1)
