# Google CTF 2021 Beginner Quest 3
__Challenge Title :  High Speed Chase__  
__Category : Misc__  
__Challenge Description :__   

__You chase them through city streets until you reach the high way. The traffic is pretty rough for a car and you see them gaining ground - should have hotwired a motorbike as well! Too late for that. You look around your car to spot anything useful, and you notice this is actually one of the new self driving cars. You turn on the autopilot, pull out your laptop, connect it to the system, and enter the not-so-hidden developer's mode. It's time to re-program the autopilot to be a bit more useful in a chase! To make it easier, you replace the in-car LiDAR feed with a feed from an overhead sattelite - you also display it on the the entertainment system. Now all that's left to do, is to write a better controlCar function!__  

__Link : https://high-speed-chase-web.2021.ctfcompetition.com/__

After visiting the link, it seems like this is a programming challenge.  
I also checked the source code to see if there was anything interesting. I wasn't wrong. The source code had this -  

         Note that this is a PROGRAMMING challenge, not a reverse-engineering  
         one. It's way easier to solve it in the intended way (by writing the
         code).  
         That said, I'm a sign, not a cop.  

So, it was clear that we just need to program the car to drive automatically.
The challenge was self explanatory.  


    Car Self-Driving Interface

    You need to re-implement the controlCar function.

    To implement it in JavaScript use the editor on the left.

    When implemented, controlCar function will be called several times per second during the chase to allow for course corrections.

    The controlCar function takes a single parameter – scanArray – which is   
    an array containing 17 integers denoting distance from your car to the nearest obstacle:

    [indexes 0-7]: on the left side of the car (index 7 is the measurement at the left headlight),
    [index 8]: at the center of the car,
    [indexes 9-16]: on the right side of the car (index 9 is the measurement at the right headlight).
    See also this image (it's not precise, but will give you an idea what you are looking at).

    All measurements are parallel to each other.

    A negative measurement might appear if the obstacle is very close behind our car.

    The controlCar must return an integer denoting where the car should drive:

    -1 (or any other negative value): drive more to the left,
    0: continue straight / straighten up the car,
    1 (or any other positive value): drive more to the right.

And there was also this image attached -  

![task3explained](https://github.com/0xL30N3/Writeups/blob/main/Images/task3explained.png)

I suggest taking some time to read the challenge explanation and the description.  
An issue I faced when solving this task is that, scanArray[0] and scanArray[16] are always 0 which isn't mentioned in the description.  
With that in mind, I wrote this very simple script that solved the challenge :  

    if(scanArray[10] > 13){
             return 1;
       }

    else if(scanArray[6] > 13){
             return -1;
       }

    else if(scanArray[3] > 13){
             return -1;
       }

    else if(scanArray[13] > 13){
             return 1;
       }

I will now break it down line by line.

    if(scanArray[10] > 13){
             return 1;
       }

    else if(scanArray[6] > 13){
             return -1;
       }

This piece of code scans which side is clearer, left or right and then switch accordingly.  
We compare it with 13 as it seemed like the sweet spot.  
Just this alone would work if there weren't obstacles blocking 2 lanes instead of 1.  
For this, we just need to write the same code but with further sensors to make the car switch 2 lanes at once.  
And there we have it.  

__Flag - CTF{cbe138a2cd7bd97ab726ebd67e3b7126707f3e7f}__




