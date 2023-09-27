# iBus Input Method

Status: Operational

Reference: https://wiki.debian.org/I18n/ibus

### Steps (without using im-config)

```
sudo apt install ibus ibus-libpinyin
sudo reboot
```
After restarting, you need to manually add the Chinese input method to the input options:

System Tray Icon -> Right-click -> Preferences
![](images/ibus_1.png)

Click on the Input Method tab -> Add, which will open the following window:
![](images/ibus_2.png)

Click on Chinese -> Intelligent Pinyin -> Add
![](images/ibus_3.png)