# Linear Clock
---
![Linear Clock](images/linear-clock.jpg)

A clock made from a cardboard box using LED strips.

**Difficulty: Moderate -- soldering required.**

**Objective: Build a simple clock using real time clock Click module.**

**Note: This project requires the RTC 6 click module and an LED light strip incorporating the LPD8806S driver chip.**

## How it Works
This project uses the RTC 6 click module and LED light strips as the basis for a BrainPad based clock. The BrainPad program provides a means of setting the time on the clock module and drives the LED light strip. When you remove power from the BrainPad the battery backup on the RTC 6 click module will keep the clock from losing time. When the BrainPad is powered up, it automatically reads the time from the clock module. Press the left and right buttons together to set the time.

## The Code in C#
> [!Tip]
> Make sure the namespace in your program matches your project's namespace.  Your project's namespace can be found in the BrainPad Helper file by clicking on the BrainPad1.cs tab.  [**More Info**](../go-beyond/csharp/intro.md#a-few-words-about-namespaces).

```
using GHIElectronics.TinyCLR.Devices.Spi;
using GHIElectronics.TinyCLR.Devices.I2c;

namespace ModifyThis {
    class Program {
        private static byte[] i2cReadData = new byte[1];
        private static byte[] i2cWriteData = new byte[2];
        private static byte[] i2cMultipleReadData = new byte[3];
        private static byte hours, minutes, seconds, cursorLocation, secondsCurrent;
        private static int secondsBrightness;

        static void Main() {
            byte[] colors = new byte[3 * 36];
            byte[] zeros = new byte[3 * ((36 + 63) / 64)];
            bool ok = false;

            var i2cSettings = new I2cConnectionSettings(0b01101111) {
                SharingMode = I2cSharingMode.Shared,
                BusSpeed = I2cBusSpeed.FastMode,
            };
            var realTimeClock = I2cDevice.FromId(GHIElectronics.TinyCLR.Pins.BrainPad.Expansion.I2cBus.I2c1, i2cSettings);

            var spiSettings = new SpiConnectionSettings(GHIElectronics.TinyCLR.Pins.BrainPad.Expansion.GpioPin.Cs) {
                Mode = SpiMode.Mode0,
                ClockFrequency = 20000000,
                DataBitLength = 8,
            };
            var ledStrip = SpiDevice.FromId(GHIElectronics.TinyCLR.Pins.BrainPad.Expansion.SpiBus.Spi1, spiSettings);

            while (true) {
                i2cReadData[0] = (0x00);
                realTimeClock.WriteRead(i2cReadData, i2cMultipleReadData);

                seconds = (byte)((i2cMultipleReadData[0] & 0b00001111) + ((i2cMultipleReadData[0] & 0b01110000) >> 4) * 10);
                minutes = (byte)((i2cMultipleReadData[1] & 0b00001111) + (i2cMultipleReadData[1] >> 4) * 10);
                hours = (byte)((i2cMultipleReadData[2] & 0b00001111) + ((i2cMultipleReadData[2] & 0b00010000) >> 4) * 10);

                if (BrainPad.Buttons.IsLeftPressed() && BrainPad.Buttons.IsRightPressed()) SetTime();

                for (var i = 0; i < colors.Length; ++i) colors[i] = 0x80;

                if (seconds != secondsCurrent) {
                    BrainPad.Display.Clear();
                    BrainPad.Display.DrawText(18, 23, hours.ToString("D2") + ":" + minutes.ToString("D2") + ":" + seconds.ToString("D2"));
                    BrainPad.Display.RefreshScreen();
                    secondsCurrent = seconds;
                    secondsBrightness = 0;
                    BrainPad.Buzzer.StartBuzzing(1);
                    BrainPad.Buzzer.StopBuzzing();
                }

                colors[hours * 3 - 2] = 0xFF;
                colors[(int)(minutes / 5 + 0.5) * 3 + 36] = (byte)(0x80 | (((minutes % 5) + 1) * ((minutes % 5) + 1) * ((minutes % 5) + 1)));
                colors[(int)(seconds / 5 + 0.5) * 3 + 72 + 2] = (byte)(0x80 | secondsBrightness);
                if (secondsBrightness < 127) secondsBrightness += 1;

                ledStrip.Write(colors);
                ledStrip.Write(zeros);
                BrainPad.Wait.Milliseconds(25);
            }

            void SetTime() {
                cursorLocation = 0;
                ok = false;
                ShowSetTimeScreen();

                while (!ok) {
                    if (BrainPad.Buttons.IsLeftPressed()) {
                        if (cursorLocation > 0) cursorLocation--;
                        ShowSetTimeScreen();
                    }

                    if (BrainPad.Buttons.IsRightPressed()) {
                        if (cursorLocation < 5) cursorLocation++;
                        ShowSetTimeScreen();
                    }

                    if (BrainPad.Buttons.IsUpPressed()) {
                        switch (cursorLocation) {
                            case 0:
                                hours++;
                                if (hours > 12) hours = 1;
                                break;

                            case 1:
                                minutes += 10;
                                if (minutes > 59) minutes -= 60;
                                break;

                            case 2:
                                if ((minutes + 1) % 10 == 0) minutes -= 9;
                                else minutes++;
                                break;

                            case 3:
                                seconds += 10;
                                if (seconds > 59) seconds -= 60;
                                break;

                            case 4:
                                if ((seconds + 1) % 10 == 0) seconds -= 9;
                                else seconds++;
                                break;

                            case 5:
                                ok = true;
                                break;
                        }
                        ShowSetTimeScreen();
                    }

                    if (BrainPad.Buttons.IsDownPressed()) {
                        switch (cursorLocation) {
                            case 0:
                                hours--;
                                if (hours < 1) hours = 12;
                                break;

                            case 1:
                                if (minutes > 9) minutes -= 10;
                                else minutes += 50;
                                break;

                            case 2:
                                if (minutes % 10 == 0) minutes += 9;
                                else minutes--;
                                break;

                            case 3:
                                if (seconds > 9) seconds -= 10;
                                else seconds += 50;
                                break;

                            case 4:
                                if (seconds % 10 == 0) seconds += 9;
                                else seconds--;
                                break;

                            case 5:
                                ok = true;
                                break;
                        }
                        ShowSetTimeScreen();
                    }
                }


                i2cWriteData[0] = (0x00);
                i2cWriteData[1] = (byte)(0x80 | ((seconds / 10) << 4) | seconds % 10);
                realTimeClock.Write(i2cWriteData);

                i2cWriteData[0] = (0x01);
                i2cWriteData[1] = (byte)(((minutes / 10) << 4) | minutes % 10);
                realTimeClock.Write(i2cWriteData);

                i2cWriteData[0] = (0x02);
                i2cWriteData[1] = (byte)(0x40 | ((hours / 10) << 4) | hours % 10);
                realTimeClock.Write(i2cWriteData);
            }

            void ShowSetTimeScreen() {
                BrainPad.Display.Clear();
                BrainPad.Display.DrawText(16, 0, "Set Time");
                BrainPad.Display.DrawText(55, 47, "OK");
                if (cursorLocation < 5) BrainPad.Display.DrawText(18 + (cursorLocation + 1) * 12 + (cursorLocation + 1) / 2 * 12, 28, "_");
                else BrainPad.Display.DrawRectangle(50, 44, 32, 20);
                BrainPad.Display.DrawText(18, 23, hours.ToString("D2") + ":" + minutes.ToString("D2") + ":" + seconds.ToString("D2"));
                BrainPad.Display.RefreshScreen();
            }
        }
    }
}
```