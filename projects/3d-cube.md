# 3D Cube
---
![3D Cube](images/3d-cube.gif)

A simple wire-frame drawing of a three-dimensional cube.

**Difficulty: Easy.**

**Objective: 3D Math.**

## How It Works

This program renders a three-dimensional cube on the BrainPad screen. As you tilt the BrainPad the cube is rotated simulating what it would look like if you were rotating the cube itself. The output of the X and Y axes of the accelerometer is also shown.

## The Code in C#
> [!Tip]
> Make sure the namespace in your program matches your project's namespace.  Your project's namespace can be found in the BrainPad Helper file by clicking on the BrainPad1.cs tab.  [**More Info**](../go-beyond/csharp/intro.md#a-few-words-about-namespaces).

```
using System;

namespace ModifyThis {
    class Program {
        class Vector3 {
            public double X;
            public double Y;
            public double Z;
            public Vector3(double X, double Y, double Z) {
                this.X = X;
                this.Y = Y;
                this.Z = Z;
            }
        }

        class Vector2 {
            public double X;
            public double Y;
            public Vector2(double X, double Y) {
                this.X = X;
                this.Y = Y;
            }
        }

        static void Translate3Dto2D(Vector3[] Points3D, Vector2[] Points2D, Vector3 Rotate, Vector3 Position) {
            int OFFSETX = 64;
            int OFFSETY = 32;
            int OFFSETZ = 50;

            double sinax = Math.Sin(Rotate.X * Math.PI / 180);
            double cosax = Math.Cos(Rotate.X * Math.PI / 180);
            double sinay = Math.Sin(Rotate.Y * Math.PI / 180);
            double cosay = Math.Cos(Rotate.Y * Math.PI / 180);
            double sinaz = Math.Sin(Rotate.Z * Math.PI / 180);
            double cosaz = Math.Cos(Rotate.Z * Math.PI / 180);

            for (int i = 0; i < 8; i++) {
                double x = Points3D[i].X;
                double y = Points3D[i].Y;
                double z = Points3D[i].Z;

                double yt = y * cosax - z * sinax;  // rotate around the x axis
                double zt = y * sinax + z * cosax;  // using the Y and Z for the rotation
                y = yt;
                z = zt;

                double xt = x * cosay - z * sinay;  // rotate around the Y axis
                zt = x * sinay + z * cosay;         // using X and Z
                x = xt;
                z = zt;

                xt = x * cosaz - y * sinaz;         // finally rotate around the Z axis
                yt = x * sinaz + y * cosaz;         // using X and Y
                x = xt;
                y = yt;

                x = x + Position.X;                 // add the object position offset
                y = y + Position.Y;                 // for both x and y
                z = z + OFFSETZ - Position.Z;       // as well as Z

                Points2D[i].X = (x * 64 / z) + OFFSETX;
                Points2D[i].Y = (y * 64 / z) + OFFSETY;
            }
        }

        static void Main() {
            // our object in 3D space
            Vector3[] cube_points = new Vector3[8]
            {
            new Vector3(10,10,-10),
            new Vector3(-10,10,-10),
            new Vector3(-10,-10,-10),
            new Vector3(10,-10,-10),
            new Vector3(10,10,10),
            new Vector3(-10,10,10),
            new Vector3(-10,-10,10),
            new Vector3(10,-10,10),
            };

            // what we get back in 2D space!
            Vector2[] cube2 = new Vector2[8]
            {
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            new Vector2(0,0),
            };

            // the connections between the "dots"
            int[] start = new int[12] { 0, 1, 2, 3, 4, 5, 6, 7, 0, 1, 2, 3 };
            int[] end = new int[12] { 1, 2, 3, 0, 5, 6, 7, 4, 4, 5, 6, 7 };

            Vector3 rot = new Vector3(0, 0, 0);
            Vector3 pos = new Vector3(0, 0, 0);

            while (true) {
                int accelX = BrainPad.Accelerometer.ReadX();
                int accelY = BrainPad.Accelerometer.ReadY();
                BrainPad.Display.Clear();

                rot.X = 360 - accelY * 0.5;
                rot.Y = accelX * 0.5;

                Translate3Dto2D(cube_points, cube2, rot, pos);

                for (int i = 0; i < start.Length; i++) {    // draw the lines that make up the object
                    int vertex = start[i];                  // temp = start vertex for this line
                    int sx = (int)cube2[vertex].X;          // set line start x to vertex[i] x position
                    int sy = (int)cube2[vertex].Y;          // set line start y to vertex[i] y position
                    vertex = end[i];                        // temp = end vertex for this line
                    int ex = (int)cube2[vertex].X;          // set line end x to vertex[i+1] x position
                    int ey = (int)cube2[vertex].Y;          // set line end y to vertex[i+1] y position
                    BrainPad.Display.DrawLine(sx, sy, ex, ey);
                }
                BrainPad.Display.DrawSmallText(0, 55, "X: " + accelX.ToString());
                BrainPad.Display.DrawSmallText(80, 55, "Y: " + accelY.ToString());
                BrainPad.Display.RefreshScreen();
                BrainPad.Wait.Minimum();
            }
        }
    }
}

```

## More Information About 3D Graphics

With the advent of fast computers and powerful video game systems, three dimensional (3D) graphics have become commonplace. It used to be much more difficult for older, slower computers to display 3D graphics.  Early attempts were rendered using a wire-frame representation -- just like this sample program does.

To display this cube on the BrainPad, first a 3D model of the cube must be created. This model consists of X, Y, and Z coordinates for the location of each of the eight corners of the cube.

The BrainPad display, as well as computer monitors and videogame displays, is only two dimensional. The three-dimensional cube must be translated into two dimensions before it can be displayed on the screen. This is done mathematically using geometry.

An easy way to understand this is to pretend you are looking at the 3D cube through a window with one eye closed. The window would represent the BrainPad display or a computer screen. Now draw an imaginary line from each corner of the cube through the window to your open eye. Each of the corners of the 3D cube will have a corresponding point on the window where the imaginary line from that corner goes through (or intersects) the window.

Each corner of the 3D cube is represented by three coordinates (X,Y, and Z). Each point on the 2D window is represented by only two coordinates (X and Y). Thus, we now have a 2D representation (the points on the window) of a 3D object (the cube outside of the window).

Now we can draw the lines that represent the edges of the cube between the points on the window. We now have a wire-frame 2D image of the 3D cube. Every time the cube is rotated we must make a new 3D to 2D translation.

Today's computers are so fast that they can render complex, almost life-like three-dimensional models quickly enough to make the realistic video games we have today. The computing power needed to do this is enormous -- some video game systems are capable of performing hundreds of billions of calculations per second! Yet the math used is all based on the same math we use to render this cube.