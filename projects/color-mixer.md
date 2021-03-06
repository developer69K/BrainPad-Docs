# Color Mixer
---
![Color wheel](images/color-wheel.jpg)

Did you know that you can get any color by simply mixing different levels of red, green and blue?

**Difficulty: Easy.**

**Objective: Science.**

## How It Works
We build a simple selection menu to select a color, which we then can increase or decrease. We are limiting the light to max level of 10 to protect your eyes since you will be looking directly at it. The max level is actually 100.

## The Code in C#
> [!Tip]
> Make sure the namespace in your program matches your project's namespace.  Your project's namespace can be found in the BrainPad Helper file by clicking on the BrainPad1.cs tab.  [**More Info**](../go-beyond/csharp/intro.md#a-few-words-about-namespaces).

```
namespace ModifyThis {
    class Program {
        static void Main() {
            int selection = 0;
            double[] LightBulbColor = new double[3];

            BrainPad.Display.DrawScaledText(0, 45, "Up/Down to select", 1, 1);
            BrainPad.Display.DrawScaledText(0, 55, "Right/Left to level", 1, 1);
            BrainPad.Display.DrawScaledText(12, 9 * 0, "Red  : 0.0", 1, 1);
            BrainPad.Display.DrawScaledText(12, 9 * 1, "Green: 0.0", 1, 1);
            BrainPad.Display.DrawScaledText(12, 9 * 2, "Blue : 0.0", 1, 1);

            while (true) {
                // The selectino pointer
                BrainPad.Display.DrawScaledText(0, selection * 9, " ", 2, 1);

                if (BrainPad.Buttons.IsDownPressed() || selection == -1) {
                    BrainPad.Buzzer.Beep();
                    selection++;
                    if (selection >= 3)
                        selection = 0;
                    while (BrainPad.Buttons.IsDownPressed())
                        BrainPad.Wait.Minimum();
                }

                if (BrainPad.Buttons.IsUpPressed() || selection == -1) {
                    BrainPad.Buzzer.Beep();
                    selection--;
                    if (selection < 0)
                        selection = 2;
                    while (BrainPad.Buttons.IsUpPressed())
                        BrainPad.Wait.Minimum();
                }
                BrainPad.Display.DrawScaledText(0, selection * 9, ">", 2, 1);

                // Change the color level
                if (BrainPad.Buttons.IsRightPressed()) {
                    LightBulbColor[selection] += 0.3;
                    if (LightBulbColor[selection] > 10) LightBulbColor[selection] = 10;
                    BrainPad.Display.ClearPart(60, 9 * selection, 18, 8);
                    BrainPad.Display.DrawScaledText(54, 9 * selection, LightBulbColor[selection].ToString("N1"), 1, 1);
                }

                if (BrainPad.Buttons.IsLeftPressed()) {
                    LightBulbColor[selection] -= 0.3;
                    if (LightBulbColor[selection] < 0) LightBulbColor[selection] = 0;
                    BrainPad.Display.ClearPart(60, 9 * selection, 18, 8);
                    BrainPad.Display.DrawScaledText(54, 9 * selection, LightBulbColor[selection].ToString("N1"), 1, 1);
                }

                // Set the color and show the screen
                BrainPad.LightBulb.TurnColor(LightBulbColor[0], LightBulbColor[1], LightBulbColor[2]);
                BrainPad.Display.RefreshScreen();
                BrainPad.Wait.Minimum();
            }
        }
    }
}
```