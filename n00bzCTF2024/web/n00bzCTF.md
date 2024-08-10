## web/File Sharing Portal
#### Category: Web
#### Points: 478

#### Challenge description: `Welcome to the file sharing portal! We only support tar files! Author: NoobMaster + NoobHacker`
#### Attachment: 
```bash
└── chall.zip
   ├── Dockerfile
   ├── requirements.txt
   ├── server.py
   └── templates
       └── index.html
```

#### Approach

The webapp is fairly simple. You submit a tar file and it extracts the content of it and hosts it on the site for a certain amount of time. Let's check the source code to see how it does that. Our point of interest is this snippet of code:
```python
elif request.method == 'POST':
	file = request.files['file']
	if file.filename[-4:] != '.tar':
		return render_template_string("<p> We only support tar files as of right now!</p>")
	name = sha256(os.urandom(16)).digest().hex()
	os.makedirs(f"./uploads/{name}", exist_ok=True)
	file.save(f"./uploads/{name}/{name}.tar")
	try:
		tar_file = tarfile.TarFile(f'./uploads/{name}/{name}.tar')
		tar_file.extractall(path=f'./uploads/{name}/')
		return render_template_string(f"<p>Tar file extracted! View <a href='/view/{name}'>here</a>")
	except:
		return render_template_string("<p>Failed to extract file!</p>")
```
When you upload a file(POST request), it checks the filename of the file that you uploaded and checks to see if the filename ends with `.tar`. Then, it's gonna create a random SHA256 string, which becomes the variable `name`.  Then, it's gonna create a directory under the server's `./uploads/` using the sha256 encrypted name. Then, it uploads the tar file on that directory with the same name(sha256). After that, it tries to extract the content from the tar file and places the extracted content in the same directory as the tar file. Then, if it succeeds, it provides a url to `/view/{name}` for you to check your files. Viewing the files extracted from the tar file is handled by this code snippet:

```python
@app.route('/view/<name>')
def view(name):
    if not all([i in "abcdef1234567890" for i in name]):
        return render_template_string("<p>Error!</p>")
        #print(os.popen(f'ls ./uploads/{name}').read())
            #print(name)
    files = os.listdir(f"./uploads/{name}")
    out = '<h1>Files</h1><br>'
    files.remove(f'{name}.tar')  # Remove the tar file from the list
    for i in files:
        out += f'<a href="/read/{name}/{i}">{i}</a>'
       # except:
    return render_template_string(out)
```

The webapp gets all files under the directory using `files = os.listdir(f"./uploads/{name}")`. 
Then, it appends all file names onto a python variable named `out` .  This is done using `out += f'<a href="/read/{name}/{i}">{i}</a>'`. where i is the filename of files inside the tar file. Then, the entirety of the `out` string which contains the filenames of files inside the tar file is gonna get rendered using the `render_template_string` function which is a function of the `jinja2` template engine which is used by `flask`.
#### The problem

The main problem I noticed was the fact that the application does not sanitize the filename, which is user input, before rendering it which can cause Server Side Template Injection(SSTI)

**What is SSTI?**<br/>

[Server Side Template Injection](https://portswigger.net/web-security/server-side-template-injection) is a type of vulnerability found in template engines where an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side

I can quickly test out this hypothesis by creating a file with the name `{{config}}` and compressing it into a tarfile(the name doesn't matter here). The Jinja2 template engine SSTI payload `{{config}}` calls the python object `config` where you can find all configured env variables. After we upload the tar file and go check the uploaded files, we can see the payload worked and the site is now giving us all configured env vars.

![config](https://raw.githubusercontent.com/0xL30N3/Writeups/main/Images/ssti.png)

Now that we know SSTI works, we can figure out the payload we're gonna use to read the flag. I'm using the [shortest possible payload](https://twitter.com/podalirius_/status/1655970628648697860) known to achieve RCE in jinja2, 
`{{lipsum.__globals__.os.popen("ls").read()}}`. 

`{{ ... }}`: These are used by Jinja2 to indicate expression that should be evaluated and rendered. <br/>
[lipsum](https://jinja.palletsprojects.com/en/2.11.x/templates/#lipsum): `lipsum` is a jinja2 function used to generate lorem ipsum<br/>
`__globals__`: lipsum can access the global namespace which can be used to call global variables and functions<br/>
[os](https://docs.python.org/3/library/os.html): `os` is a python module that we can use to run OS commands, allowing us to achieve RCE.<br/>
[popen("ls")](https://docs.python.org/3/library/os.html#os.popen):  This is the function from the os module that we're gonna use to execute the os command `ls` which will give us all file names in the current directory as an open file object.<br/>
`.read()`:  This will return the open file object as a string<br/>

We can then upload the tar file which has the file with the payload name of `{{lipsum.__globals__.os.popen("ls").read()}}`.  This gives us all files in the current directory as expected.<br/>

![ls](https://raw.githubusercontent.com/0xL30N3/Writeups/main/Images/ls.png)

Now we can see the flag file `flag_15b726a24e04cc6413cb15b9d91e548948dac073b85c33f82495b10e9efe2c6e.txt`. All we gotta do now is to `cat` the flag.
We create a new file with the payload `{{lipsum.__globals__.os.popen("cat flag*").read()}}` as the filename and compress it into a tar file. Finally, after uploading it, we can view the flag.

![config](https://raw.githubusercontent.com/0xL30N3/Writeups/main/Images/flag.png)

#### The Fix?
A possible fix I see is to sanitize the filenames by removing special characters like `{`, `}` and `%`. 
