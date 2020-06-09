# arduino-bootloader-findings

Given the correct circumstances, via a `ATTR_NO_INIT` or a (?)`volatile` pointer, a **magic key** is able to be placed in a specific place in an AVR's memory, keeping constant through a reset (although, likely not a full power cycle).

This key is mostly used for a few things:

1. The Arduino core (or just a program in general) telling the bootloader to start its magic once the sketch is told to reset.

2. Telling if an end-user pressed the reset button twice (used in [Sparkfun's Caterina](https://github.com/sparkfun/Arduino_Boards/tree/master/sparkfun/avr/bootloaders/caterina), [HoodLoader2](https://github.com/NicoHood/HoodLoader2/tree/master/avr/bootloaders/HoodLoader2), and likely more.

3. An extension of 2, the bootloader talking to itself through reset. As in "Hey, we likely didn't have a good flash. Come back to main() if you see this."

This function, often named, `Application_Jump_Check`, is run *before* main(), via a special attribute in the program's header file 

Example:

```cpp

	/* Function Prototypes: */
		static void SetupHardware(void);

		void Application_Jump_Check(void) ATTR_INIT_SECTION(3);
```

From LUFA's HID Bootloader

---

For a bootloader not used by Arduinos, its not uncommon to see it placed anywhere in memory the compiler deems satisfactory. However, to use (1.), that won't do. There seem to be two "valid" places for this.

`0x800` and `RAMEND`

The former was the standard for a while, but it can cause problems for microcontrollers without at least 2k ram (Atmel's 16u2, for example). So often times, it is placed via the `RAMEND` macro.

(As of writing, I don't know how it works. Tis magic)

One thing I did notice, is that often the key `0x7777` is used. However, many bootloaders only check for an 8-bit `0x77`

