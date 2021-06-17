---
layout: post
title: Handling credentials in Python
tags: programming encryption python
---

Logins to online services, and online banking in particular, seem to be becoming increasingly more elaborate; depending on the particular service I'm dealing with, I always run into (in my opinion) nonsensical security measures and obfuscation attempts that just make the login procedure as cumbersome as possible.
To provide an example, one of my banks provides a login form with a numerical keypad for password entry, but where the buttons are unordered, like this:

```C
7 6 2 5 8
1 9 0 3 4
```

The layout of the keypad is always randomized, and, to add insult to injury, it's impossible to paste anything into the input field, so I need to click on the numbers one by one with the mouse every single time.
I can only assume this was done because of a "mobile first" attitude (read: they don't want to maintain both a desktop and a mobile version of the site), making it extremely annoying to do something as simple as checking the current account balance.
Fortunately, after some googling, I found out that this particular bank actually _does_ provide an API, and that someone has developed a Python interface for it.
With a bit of bumbling around, I managed to get the interface to work, so I could now easily access the account from the CLI, though I still had the problem of inputting the login credentials every single time I called the script, so I decided to look around for ways to streamline that part of the process as well.
I had the following requirements in mind for handling the credentials:

* the credentials themselves should be recoverable in plaintext, i.e. I don't just need a hash of them or something
* the credentials should _not_ be stored on the disk. Alternatively, if this cannot be achieved, the credentials should not be stored _unencrypted_ (i.e. in plaintext) on the disk
* I should only have to input the credentials once (or as few times as possible) per user session (by session, I mean the one spawned by the operating system, i.e. after a reboot I should be prompted for them again)

Of course, people's security requirements vary, so what works for me may not work for someone else.  
In any case, here's a brief outline of the different solutions I've tried out.

## Solution 1: ask for credentials every time

Initially, I didn't have a lot of info to go with, so I decided to prompt for the credentials in the CLI every time using `input`.
For sensitive data, such as passwords, I didn't want to make the text visible on input; this can be achieved using the `getpass` Python module, as follows:

```python
import getpass

password = getpass.getpass('Please input the password: ')

# use `password` for login to the service
```

This solution has the benefit of security, but it's not very convenient since I have to input the credentials every time, albeit this is still superior to the bank's own website, since now at least I can directly paste the password from the clipboard[^password-clipboard].

In short, I'd use this solution only if it's _really_ necessary, and no other options are available.

## Solution 2: environmental variables

My first stop when trying to find solutions to programming issues is almost always Stackoverflow, and this time was no exception.
One of [the solutions][so-env] mentioned for storing secrets involved setting a bunch of environmental variables, which can then be read by the program.
Setting an environmental variable in the shell is in principle simple[^python-env]:

```bash
export MY_VARIABLE='value'
```

And so is reading it in Python:

```python
import os

my_variable = os.getenv('MY_VARIABLE')
```

Unfortunately, this doesn't work, since the above shell command only sets the environmental variable for the current shell and its subshells, _but not anything else_.
This means that, if I open a new terminal window, `echo $MY_VARIABLE` will show nothing, so I would have to revert back to inputting the credentials again.
As explained in [this unix.SE answer][unix-env], the only way to make the environmental variable visible to other processes is, you guessed it, by _writing it to a file_, which is the exact issue I was trying to avoid in the first place, so environmental variables are a no-go.

## Solution 3: GNU privacy guard (GPG)

As mentioned earlier, one of the requirements I have is to minimize the number of times I get prompted for credentials, as these things quickly become a source of friction.
Reading up a bit more, I've noticed that [GNU privacy guard][wiki-gpg], the battle-tested solution for encrypted data communication, may be the answer to my problems, as:

* it supports symmetric encryption, i.e. I can encrypt a file with a password, and then retrieve its contents by inputting the password again
* similarly to the SSH agent, it has a daemon, the [GPG agent][arch-gpg-agent], which can store credentials in-memory, so I don't have to keep inputting the same thing over and over again, at least for some period of time

Therefore, the basic 2-step procedure I've envisioned is as follows:

1. make an encrypted file containing the credentials
2. retrieve the credentials from the file, and then, once they're cached, from the GPG agent

Both of those can be achieved using the `gnupg` [Python module][pypi-gnupg] (which, if you look inside the code more closely, is just a thin wrapper for the `gpg` command-line utility);
the first step can be accomplished with the help of `getpass` from [method 1](#solution-1-ask-for-credentials-every-time), and only has to be run once:

```python
import getpass
import gnupg

gpg = gnupg.GPG()

# get the credentials (or via `input`)
credentials = getpass.getpass()

# encrypt them and store them in a file
# note: the second arguments is mandatory, but
# can be ignored in the case of symmetric encryption
# note 2: the algorithm used for encryption can be specified
# by passing a string to `symmetric`, like `symmetric='AES256'`
content = gpg.encrypt(
    credentials, None, symmetric=True,
    output='/path/to/file',
)
```

Running the above, I'm first prompted for the credentials, and then greeted by a graphical prompt for a password for the file itself, after which all of the credentials will be stored in the specified output file.

Decrypting the file is easy:

```python
import gnupg

# use the currently-running agent to retrieve credentials
# (this is so it doesn't prompt for password every time)
gpg = gnupg.GPG(use_agent=True)
# decrypt the file
content = gpg.decrypt_file(open('/path/to/file', 'rb'))
# note: the (binary) content needs to be decoded beforehand
content_string = content.data.decode('utf-8')
# now it can be used as a regular string
```

which opens up a prompt (or asks on the CLI) for inputting the passphrase which the file was encrypted with.
Additionally, it's generally a good idea to check whether `content.ok` is set to `True` after running `decrypt_file`, because in case it's not, `content.data` will be empty.

## Final words

And that's about it, a simple way of making encrypted files with GPG, which fulfills my 2 essential requirements, namely, that the data is encrypted on-disk, and that I'm not badgered all of the time for the password.

Bonus: as I have multiple credentials I'd like to save in the same file, it's a good idea to save the data in a structured format such as [JSON][wiki-json].
This can easily be done using Python's built-in `json` module:

```python
import json

# let's say we have 2 strings,
# `username` and `password`
# we can just map the string names to the values in a dict
json_string = json.dumps(
    {
        'username' : username,
        'password' : password,
    }
)

# then just encrypt this string with the method above
```

Later, running `json.loads(content_string)` on the decrypted string gives back the original dictionary.

## Appendix: tweaking the GPG agent cache time

By default, the GPG agent has two parameters which control how long the password for the file (as well as any other passwords entered) are cached: `max-cache-ttl` (default 2 hours), and `default-cache-ttl` (default 10 minutes).
To stop the prompt from appearing so frequently, their values (in seconds) should be set to something very high in the `~/.gnupg/gpg-agent.conf` file[^arch-gpg-agent]:

```plaintext
default-cache-ttl-ssh 60480000
max-cache-ttl-ssh 60480000
```

### Footnotes

[^password-clipboard]: the question of whether or not a password should even _be_ in the clipboard in the first place is a whole other can of worms, but with an adequate password manager + clipboard manager combo, shouldn't be that big of a deal IMHO. Again, it all depends on how sensitive the data you're accessing is.
[^arch-gpg-agent]: for more info, see <https://wiki.archlinux.org/title/GnuPG#Cache_passwords>
[^python-env]: they can also be set in Python using `os.environ['MY_VARIABLE'] = 'value'`, but this suffers from the same visibility problem as the one mentioned for the shell

[so-env]: https://stackoverflow.com/a/53027302
[unix-env]: https://unix.stackexchange.com/a/8344/364999
[wiki-gpg]: https://en.wikipedia.org/wiki/GNU_Privacy_Guard
[arch-gpg-agent]: https://wiki.archlinux.org/title/GnuPG#gpg-agent
[pypi-gnupg]: https://pypi.org/project/python-gnupg/
[wiki-json]: https://en.wikipedia.org/wiki/JSON
