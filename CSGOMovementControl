###############################
# JakeN9's CS:GO movement control script.
# This python script was created using the following

# Demo: https://www.youtube.com/watch?v=CEt46F9jePY

# Note: You'll need to turn off Raw Input in CS:GO if you have it enabled, as it messes with the control of the mouse.

###############################

# 1 . https://github.com/Devking/PythonMIDISynth - A Python Synthesier written by Wells Lucas Santo, for the final project component of  EE4163 at the NYU Tandon School of Engineering.
# 2 . http://pyautogui.readthedocs.io/en/latest/mouse.html - PyAutoGUI is a Python module for programmatically controlling the mouse and keyboard..
# 3 . 2 cups of coffee
# 4 . A ton of Google Searching

# Get a range of the frequencies for 120 keys, starting with the pitch C0
# This mathematically works because there is a separation of 1.059463 per
# pitch, in frequency. You may not be able to hear pitches until you get to
# around C4, which is the 48th key.
f = [16.35 * 1.059463 ** i for i in range(0, 120)]

# These macros come from the MIDI event values that my Alesis Q49 uses for
# detecting KEYUP and KEYDOWN events.
KEYDOWN   = 144
KEYUP     = 128

# Choose how many notes you want to be able to play at once
NOSTREAMS = 10

# Import all the necessary packages

import pyautogui

from pygame import midi
import pyaudio
import struct
import numpy as np
from math import sin, cos, pi

###############################
# Initialize sound parameters #
###############################

# This is based on the second-order difference equation code that we have used
# in the class, written by Professor Ivan Selesnick.

blockSize    = 32
sampleWidth  = 2
numChannels  = 1
samplingRate = 16000

Ta = 0.8
r = 0.01 ** (1.0 / (Ta * samplingRate))

# Calculate coefficients based on frequencies
om = [2.0 * pi * float(f1) / samplingRate for f1 in f]
a1 = [-2*r*cos(om1) for om1 in om]
a2 = r**2
b0 = [sin(om1) for om1 in om]

# Open the audio output streams
p = pyaudio.PyAudio()

# Put the streams into a circular buffer
# Use list comprehension to make the streams directly in this list
stream_buffer = [p.open(format            = p.get_format_from_width(sampleWidth),
                        channels          = numChannels,
                        frames_per_buffer = blockSize,
                        rate              = samplingRate,
                        input             = False,
                        output            = True)
                for i in range(NOSTREAMS)]

# Circular buffer of arrays
y = [np.zeros(blockSize) for i in range(NOSTREAMS)]
x = [np.zeros(blockSize) for i in range(NOSTREAMS)]

# Circular buffer of ints
pitch = [0 for i in range(NOSTREAMS)]
accesskey = 0
import ctypes
import time

#######################################
# Initialize Keyboard #
#######################################

SendInput = ctypes.windll.user32.SendInput

# C struct redefinitions 
PUL = ctypes.POINTER(ctypes.c_ulong)
class KeyBdInput(ctypes.Structure):
    _fields_ = [("wVk", ctypes.c_ushort),
                ("wScan", ctypes.c_ushort),
                ("dwFlags", ctypes.c_ulong),
                ("time", ctypes.c_ulong),
                ("dwExtraInfo", PUL)]

class HardwareInput(ctypes.Structure):
    _fields_ = [("uMsg", ctypes.c_ulong),
                ("wParamL", ctypes.c_short),
                ("wParamH", ctypes.c_ushort)]

class MouseInput(ctypes.Structure):
    _fields_ = [("dx", ctypes.c_long),
                ("dy", ctypes.c_long),
                ("mouseData", ctypes.c_ulong),
                ("dwFlags", ctypes.c_ulong),
                ("time",ctypes.c_ulong),
                ("dwExtraInfo", PUL)]

class Input_I(ctypes.Union):
    _fields_ = [("ki", KeyBdInput),
                 ("mi", MouseInput),
                 ("hi", HardwareInput)]

class Input(ctypes.Structure):
    _fields_ = [("type", ctypes.c_ulong),
                ("ii", Input_I)]

# Actuals Functions

def PressKey(hexKeyCode):
    extra = ctypes.c_ulong(0)
    ii_ = Input_I()
    ii_.ki = KeyBdInput( 0, hexKeyCode, 0x0008, 0, ctypes.pointer(extra) )
    x = Input( ctypes.c_ulong(1), ii_ )
    ctypes.windll.user32.SendInput(1, ctypes.pointer(x), ctypes.sizeof(x))

def ReleaseKey(hexKeyCode):
    extra = ctypes.c_ulong(0)
    ii_ = Input_I()
    ii_.ki = KeyBdInput( 0, hexKeyCode, 0x0008 | 0x0002, 0, ctypes.pointer(extra) )
    x = Input( ctypes.c_ulong(1), ii_ )
    ctypes.windll.user32.SendInput(1, ctypes.pointer(x), ctypes.sizeof(x))


#mouse = Controller()


#######################################
# Initialize input detection for MIDI #
#######################################

midi.init()
INPUTNO = midi.get_default_input_id()
input = midi.Input(INPUTNO)

print '*******************'
print '** Ready to play **'
print '*******************'

while True:

    # For ALL streams, set current input to 0, since nothing is being played
    # at the current moment (until a key is pressed)
    for n in range(NOSTREAMS):
        x[n][0] = 0.0

    if input.poll():
        eventslist = input.read(1000)

        for e in eventslist:
            event      = e[0]
            eventType  = event[0]
            eventID    = event[1]
            eventValue = event[2]

            if eventType == KEYDOWN:
                print 'Keydown on key', eventID, 'with intensity', eventValue

                if(eventID == 96):
                    PressKey(0xCB)

                if(eventID == 98):
                    PressKey(0xCD)

                if(eventID == 100):
                    PressKey(0xC8)

                if(eventID == 101):
                    PressKey(0xD0)

                if(eventID == 103):
                    pyautogui.mouseDown()

                # Trigger an impulse due to a keypress
                # Notice that we are triggering the impulse in the stream that
                # 'accesskey' refers to -- and then we will update 'accesskey'
                # to utilize our circular array of streams properly.
                x[accesskey][0] = 15000 * (eventValue / 130.0)
                pitch[accesskey] = eventID % 60;
                accesskey = (accesskey + 1) % NOSTREAMS

            elif eventType == KEYUP:
                print 'Keyup on key', eventID

                if(eventID == 96):
                    ReleaseKey(0xCB)

                if(eventID == 98):
                    ReleaseKey(0xCD)
                    
                if(eventID == 100):
                    ReleaseKey(0xC8)

                if(eventID == 101):
                    ReleaseKey(0xD0)

                if(eventID == 103):
                    pyautogui.mouseUp()

                if(eventID == 105):
                    pyautogui.moveRel(-300, 0)

                if(eventID == 107):
                    pyautogui.moveRel(+300, 0)

    # Update the value of the difference equation
    for n in range(blockSize):

        # Update output for all streams
        for i in range(NOSTREAMS):
            y[i][n] = b0[pitch[i]] * x[i][n] - a1[pitch[i]] * y[i][n-1] - a2 * y[i][n-2]

    # Output the value of all streams (this all happens at once!)
    # PyAudio will allow this to play the output CONCURRENTLY
    # (That is, sound from all streams will play at the same time, not one after the other!)
    for i in range(NOSTREAMS):
        y[i] = np.clip(y[i], -2**15+1, 2**15-1)
        data = struct.pack('h' * blockSize, *y[i])
        stream_buffer[i].write(data, blockSize)

# Close up all of the streams properly
for i in range(NOSTREAMS):
    stream_buffer[i].stop_stream()
    stream_buffer[i].close()

p.terminate()
midi.quit()
