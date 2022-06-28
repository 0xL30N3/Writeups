# Google CTF 2021 Beginner Quest 1
__Challenge Title :  Vienna - Chemical plant__  
__Category : pwn/pwn-re
__Challenge Description__ :  

 "You must wonder why we have summoned you, AGENT? It has come to our attention that something terrible is about to take place. There is still time to prevent the disaster, and we could not think of anyone more suited for this task than you. We believe that if you can’t solve this quest, neither can anybody else. You have to travel to Vienna, and investigate a suspicious chemical plant. This mission must be executed in secrecy. It’s classified, and it regards the safety of the whole world, therefore we can’t tell you anything more just yet. Go now, you have the fate of the world in your hands." 
 
 Link : http://cctv-web.2021.ctfcompetition.com/
 
 After visting the site, we are greeted with a login panel.
 
 ![site](https://github.com/0xle0ne/Writeups/blob/main/Images/quest1_img1.png?raw=true)
 
 We're gonna check the source code to see if there's anything interesting.  
 The only thing that caught my eyes first was the this function.
 ```
 const checkPassword = () => {
  const v = document.getElementById("password").value;
  const p = Array.from(v).map(a => 0xCafe + a.charCodeAt(0));

  if(p[0] === 52037 &&
     p[6] === 52081 &&
     p[5] === 52063 &&
     p[1] === 52077 &&
     p[9] === 52077 &&
     p[10] === 52080 &&
     p[4] === 52046 &&
     p[3] === 52066 &&
     p[8] === 52085 &&
     p[7] === 52081 &&
     p[2] === 52077 &&
     p[11] === 52066) {
    window.location.replace(v + ".html");
  } else {
    alert("Wrong password!");
  }
}
```
I'm gonna breakdown the code to better understand the function.
`const v = document.getElementById("password").value;` takes the value you entered in the password field and stored it in the variable __v__.  
After some googling, I found out that `Array.from(v)` creates an array from a string.
__Example__
```
console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]

console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]
```
So, `const p = Array.from(v).map(a => 0xCafe + a.charCodeAt(0));`  Takes your password input. Turns it into an array.  
It uses the map function to add `0xCafe` to the character.  
`a.charCodeAt(0)` will take the character and turn it into UTF-16 char code.  

So, let's say you entered "123" in the password field, "1" would become `0xCafe+49 = 52015`.  
"2" would be `0xcafe+50 = 52016`.  
"3" would be `0xcafe+51 = 52017`.  
The final array would be `[52015, 52016, 52017]`  
__0xCafe = 51966__  
__UTF char code for 1 is 49, 2 is 50 and 3 is 51__  

Now, we can move on to the next part of the code.
```
  if(p[0] === 52037 &&
     p[6] === 52081 &&
     p[5] === 52063 &&
     p[1] === 52077 &&
     p[9] === 52077 &&
     p[10] === 52080 &&
     p[4] === 52046 &&
     p[3] === 52066 &&
     p[8] === 52085 &&
     p[7] === 52081 &&
     p[2] === 52077 &&
     p[11] === 52066) {
    window.location.replace(v + ".html");
  } else {
    alert("Wrong password!");
  }
```
It checks the array with the correct values.  
`p[0] === 52037` The char code of the first word of our password + 0xCafe must be the same with 52037.  
__Note: p[0] means the array's first value(Arrays start with 0)__  

52037 = 51966 + charcode(0xCafe = 51966)  
charcode = 52037 - 51966 = 71  
71 is the charcode for the letter "G".  
Now we know that the password starts with the letter G.  
Now I'll write a python script to decode the other values  
```
#Take all values from the source code and store them in an array
char = [52037, 52077, 52077, 52066, 52046, 52063, 52081, 52081, 52085, 52077, 52080, 52066]
#Then create a loop to decode all values
for x in char:
    y=x-0xCafe #y is going to be the correct charcode
    l=y.to_bytes(2, 'little') #Turn integers to bytes before decoding
    print(l.decode('UTF-16'), end='') #Print the decoded values
```
After running the script we get the password.  
`GoodPassword`  

Login with the password and you'll get to a page that shows some CCTV footages and the flag.  
![flag](https://github.com/0xle0ne/Writeups/blob/main/Images/quest1_img2.png?raw=true)  

__Flag = CTF{IJustHopeThisIsNotOnShodan}__
