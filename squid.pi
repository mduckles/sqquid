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
    
