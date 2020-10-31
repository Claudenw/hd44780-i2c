# hd44780-i2c
This is a Linux kernel driver for Hitachi HD44780 LCDs attached to I2C bus via PCF8574 I/O expander. Ideal to use with Raspberry Pi and other small devices running Linux where I2C bus is available.

`$ uname -snrmv | tr -d '\n' > /dev/lcd0`
![hd44780-i2c driver in action](http://i.imgur.com/ct9uiRb.jpg)

### Features
The main goal was to expose HD44780-based LCDs behind regular Linux device files normally found in `/dev` directory. Thus, writing to the display is as easy as `echo Hello, world! > /dev/lcd0`.

There are no imposed limitations on number of concurrently attached devices. In practice, a single I2C bus allows up to 128 uniquely addressable devices. Furthermore, standard HD44780 LCD to I2C adapters usually use the same, hardcoded I2C address (like 0x27) and it's not possible to change them. The PCF8574 IC supports up to 8 different addresses, so a custom-build adapter might be the solution.

Multiple LCD geometries are supported (20x4, 16x8 and 8x1) and it's trivial to add new ones if needed.

Supported escape sequences:
* `\r` - carriage return
* `\n` - line feed (new line)

Supported control codes:
* `Ctrl-S`/`0x13` - backlight off
* `Ctrl-Q`/`0x11` - backlight on

Supported VT100 terminal control escape sequences:
* `<ESC>[2J` - erase screen
* `<ESC>[H` - cursor home
* `<ESC>[row;colH` - cursor to row, col (zero index origin)
* `<ESC>[nA` - Move cursor up n lines
* `<ESC>[nB` - Move cursor down n lines
* `<ESC>[nC` - Move cursor right n characters
* `<ESC>[nD` - Move cursor left n characters

**Note**: if column or row is beyond the display it the position will wrap around, it does not scroll.

Modified VT100 control escape sequences:
* `<ESC>[m>`/`<ESC>[0m` - turn off cursor attributes
* `<ESC>[4m` - turn on cursor display
* `<ESC>[5m` - turn on cursor blink
    
Device attributes exported via sysfs (`/sys/class/hd44780/<device_name>`):
* `backlight` - controls LCD backlight. Possible values: `0`, `1`
* `geometry` - sets LCD geometry. Possible values: `20x4`, `16x2`, `8x1`
* `cursor_blink` - controls cursor blink. Possible values: `0`, `1`
* `cursor_display` - displays or hides cursor. Possible values: `0`, `1`
* `char0` through `char7` - sets or displays custom character 0 thorugh 7 respectivly.  Characters are encoded as 8 5-bit patterns encoded in base 32.

#### Character encoding

The custom characters are 8 lines of 5 bits.  Each line is encoded as a base 32 digit.  The values of the encodings are [0-9A-V].  The following shows the encoding for an "X" pattern:

```
10001 - encodes as -> H
01010 --------------> A
00100 --------------> 4
01010 --------------> A
10101 --------------> H
```
To set this a character 0 execute `echo HA4AH > /sys/class/hd44780/lcd0/char1`

the site https://www.unitconverters.net/numbers/binary-to-base-32.htm will provide assistance with the encoding.

To print the character simply print `\x0`

### Usage
1. Insert kernel module: `insmod hd44780.ko`.
2. Let the I2C adapter know that there's a new device attached: `echo hd44780 0x27 > /sys/class/i2c-adapter/i2c-1/new_device`.
You may need to replace the device's I2C address and adapter path with proper values.
3. At this point a new device should appear (`/dev/lcd0`) and you should be able to write to it.
