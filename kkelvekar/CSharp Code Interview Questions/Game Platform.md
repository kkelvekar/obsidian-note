![[Pasted image 20250309204803.png]]

``` C#
using System;

public class GamePlatform
{
    public static double CalculateFinalSpeed(double initialSpeed, int[] inclinations)
    {
       double currentSpeed = initialSpeed;
       foreach(int slope in inclinations) 
       {
           currentSpeed -= slope; 
           if(currentSpeed <= 0) return 0;
           
       }
       return currentSpeed; 
    }
    
    public static void Main(string[] args)
    {
        Console.WriteLine(CalculateFinalSpeed(60, new int[] { 0, 30, 0, -45, 0 }));
    }
}
```