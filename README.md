# GPG wrapper for Enigmail

Currently, Enigmail does not offer any support to select specific subkeys for
signing.

Enigmail can, for example, not be used with
[offline master/primary keys](https://wiki.debian.org/Subkeys?action=show&redirect=subkeys)
or when specific subkeys should be used for signing different data (mail, commits, ...).

Unfortunately, the Enigmail developers do not seem to be willing to change this
situation (comments to [this](https://sourceforge.net/p/enigmail/bugs/256/) or
[this bug](https://sourceforge.net/p/enigmail/bugs/426/) or to
[this request in the forum](https://sourceforge.net/p/enigmail/forum/support/thread/37b7a5c8/)).
They justify this decision claiming there are no valid use cases in which one
would want to select a specific subkey or that such use cases are too advanced for
Enigmail users.

This wrapper simply replaces the '--local-user' option with the key selected by
Enigmail and enforces the use of a subkey ID specified in a mapping file.

## Installation

1. Clone this repository
2. Edit the mapping file (`subkey_map.conf`).
    An example mapping can be found in the Example section below.
	The file contains further information on the format used.

    I would recommend to copy this file in the gnupg user folder (default: ~/.gnupg).
3. Configure the path to the mapping file in `enigmail_gpg_wrapper` by changing
    the `CONF_FILE` variable in line 3.

	By default, the variable is set to use a `subkey_map.conf` file in gnupg home.
4. Set file permissions: `chmod 0500 enigmail_gpg_wrapper` and `chmod 0500 subkey_map.conf`
	Remember to also set the permissions for files that you have copied previously.
5. Configure Enigmail to use the wrapper:
	Go to `Enigmail -> Settings -> Basic Settings`:
    In `Files and Directories` override the GPG path with the path to the
	`enigmail_gpg_wrapper` script.


## Example

Imagine the following key setup:
```
$ gpg -K --with-subkey-fingerprint
sec#  rsa4096 1970-01-01 [SC] [expires: 2038-01-19]
      DEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEF
uid           [ultimate] John Doe <johndoe@example.com>
ssb   rsa4096 1970-01-01 [E]
      ABCDEF1234ABCDEF1234ABCD001234ABCDEF1234
ssb   rsa4096 1970-01-01 [S] [expires: 2038-01-19]
      1234COFFEECOFFEECOFFEEEE1234COFFEECOFFEE
```

The private key of 0xDEADBEEFDEADBEEF (primary key) is missing meaning this key
can not be used for signing.
Enigmail, however, is trying to use this key, anyway.

In order to use the signing subkey 0x1234COFFEECOFFEE instead, the following
line needs to be added to the configuration file `subkey_map.conf`.
```
# mapping for user johndoe, 0xDEADBEEFDEADBEEF to 0x1234COFFEECOFFEE
0xDEADBEEFDEADBEEF                         0x1234COFFEECOFFEECOFFEEEE1234COFFEECOFFEE
0xDEADBEEFDEADBEEFDEADBEEFDEADBEEFDEADBEEF 0x1234COFFEECOFFEECOFFEEEE1234COFFEECOFFEE
<johndoe@example.com>                      0x1234COFFEECOFFEECOFFEEEE1234COFFEECOFFEE
```

Now, mails can be signed using the correct key.

## Why not patch Enigmail? / Why the overhead?

Since there have already been proposals for patches that were declined, I don't
have any hope, mine would be merged.
Forking Enigmail and maintaining a patched version would be an option, however,
someone would have to keep the fork up-to-date, provide an alternative add-on
download, and so on to keep it comfortably usable for end-users.

Adding this wrapper in between Enigmail and GPG adds
overhead (parsing the config, replacing the ID, ...)
to singing and encryption operations.
Given that Enigmail modifications are not required and therefore the known update
channels can be used, the overhead is (imho) acceptable.


# Disclaimer

As far as I can currently tell, this does not affect the operation of Enigmail.
However, since this wrapper is placed in between Enigmail and GPG, a (local)
attacker might use it to leek or modify sensitive information.
If you choose to use this wrapper script, make sure to protect it 
(and the configuration file) from any unauthorized manipulation.
