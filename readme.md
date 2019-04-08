
# Micah's Squid 

![](https://jduckles-dropshare.s3-us-west-2.amazonaws.com/IMG_20190324_141346.jpg)

---

## Setup the ESP8266 
```bash
# Erase Flash
esptool.py --port /dev/tty.SLAB_USBtoUART \
--baud 460800 erase_flash 
```

---

# Write Micropython Image 

```bash
esptool.py --port /dev/tty.SLAB_USBtoUART \
--baud 115200 write_flash \
--verify  \
--flash_mode=dout \
--flash_size=detect 0 \
~/Downloads/esp8266-20190125-v1.10.bin

```

---

## Wire up 

D5 = Pin 14 -> Yellow
Gnd = -> Brown
VIN = -> Red

We can run PWM on al I/O pins except for Pin 16

---

## Servo Class 

Python class to control servo:

```
from machine import PWM
import math


class Servo:
    """
    A simple class for controlling hobby servos.

    Args:
        pin (machine.Pin): The pin where servo is connected. Must support PWM.
        freq (int): The frequency of the signal, in hertz.
        min_us (int): The minimum signal length supported by the servo.
        max_us (int): The maximum signal length supported by the servo.
        angle (int): The angle between the minimum and maximum positions.

    """
    def __init__(self, pin, freq=50, min_us=600, max_us=2400, angle=180):
        self.min_us = min_us
        self.max_us = max_us
        self.us = 0
        self.freq = freq
        self.angle = angle
        self.pwm = PWM(pin, freq=freq, duty=0)

    def write_us(self, us):
        """Set the signal to be ``us`` microseconds long. Zero disables it."""
        if us == 0:
            self.pwm.duty(0)
            return
        us = min(self.max_us, max(self.min_us, us))
        duty = us * 1024 * self.freq // 1000000
        self.pwm.duty(duty)

    def write_angle(self, degrees=None, radians=None):
        """Move to the specified angle in ``degrees`` or ``radians``."""
        if degrees is None:
            degrees = math.degrees(radians)
        degrees = degrees % 360
        total_range = self.max_us - self.min_us
        us = self.min_us + total_range * degrees // self.angle
        self.write_us(us)


```

# Pin mapping to Micropy
```
Micropython | Board
0|D3
2|D4 (also Led1 but inverse)*
4|D2
5|D1
9|SD2
10|SD3
12|D6
13|D7
14|D5
15|D8
16|D0 (also Led2 but inverse)*

```

```
# Upload Servo Class to ESP8266

ampy -p /dev/tty.SLAB_USBtoUART put servo.py

```

---

## Test servos

At the micropy prompt try to control servo on D8 
```
# Connect to console
picocom /dev/tty.SLAB_USBtoUART -b115200

# Get a Python Prompt 
>>>
```

---

```
import time

# Create an instance of the Servo class on Pin 14

def swing_servo(p, a0=0, a1=180, sleep=1, pin=14, repeat=2):
    '''
    This function swings a servo from angle `a0` to angle `a1` and
    back with a delay time of `sleep` set in seconds.
    '''
    s = Servo(Pin(pin))
    for i in range(repeat):
        time.sleep(sleep)
        s.write_angle(a0)
        time.sleep(sleep)
        s.write_angle(a1)

def switch_pushed(p):
    swing_servo(p, pin=14, repeat=4)

def register_switch():
    switch = Pin(0, Pin.IN)
    switch.irq(trigger=Pin.IRQ_FALLING,  handler=switch_pushed)
    
```

