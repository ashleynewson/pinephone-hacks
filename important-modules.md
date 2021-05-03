# Kernel Modules

These notes are primarily targetted towards experimentation in an initramfs, where it's not always clear what modules you need to get hardware or software working.

## USB - anx7688

Required for getting certain USB functionality to work. For example, independently supplying power to a USB peripheral such as a keyboard.

Things like JumpDrive also won't work without this.

## Display / Touch Screen - panel-sitronix-st7703

Required to get the display working. It may also be necessary for touch input - but the display output is pretty much a necessity for that anyway.

## uinput

Needed if you want to use fbkeyboard
