# Google CTF 2021 Beginner Quest 2
__Challenge Title :  Logic Lock__  
__Category : Misc__  
__Challenge Description :__  
It turned out suspect's appartment has an electronic lock.  
After analyzing the PCB and looking up the chips you come to the conclusion that it's just a set of logic gates!  

This time, the challenge gave us a file to download.  
The description mentioned logic gates so I'm assuming the file has something to do with logic gates.  
After downloading the file, use the command `file <filename>` to check the file type.  
It returned `Zip archive data, at least v2.0 to extract, compression method=store` so it's a Zip file.  
Extract it using `unzip <filename>` and we'll have the `logic-lock.png` file.  

![logic lock](https://github.com/0xL30N3/Writeups/blob/main/Images/logic-lock.png?raw=true)  

The image gave us logic gates and some hints.  
`If only inputs A,B and C must be set, the flag will be CTF{ABC}`  
In this case, "set" probably means the values for A,B and C is 1.  
I assumed we need to find which letter's values are set to 1.  
The first thing we need to do is understand logic gates.  
I referenced this image.  
![logic gates](https://github.com/0xL30N3/Writeups/blob/main/Images/logic_gates.png?raw=true)  
The output for th logic lock is 1.  
We need to trace back from the output to the inputs.  

__Example :__  
If we trace back from the output 1, we first meet an "AND gate".  
For the "AND gate" to output 1, the inputs also need to both be "1".  
If we also trace that 1 back to `A` and `B` we meet a "NOR gate".  
"NOR gate" needs both inputs to be "0" for it to output "1".  
So, now we get that `A` = "0".  
Which mean A can be removed from the flag string.  
If we trace the other 0, we come across a "NOT gate".  
A "NOT gate" just inverses the given value.  
So, our 0 becomes a 1.  
Which means B is part of the flag string.  

If we continue to trace the inputs from the outputs, we'll finally have these values.  
- A = 0
- B = 1
- C = 1
- D = 0
- E = 0
- F = 1
- G = 0
- H = 0
- I = 1
- J = 1  

__Flag : CTF{BCFIJ}__
