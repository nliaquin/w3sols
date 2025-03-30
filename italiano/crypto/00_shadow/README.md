# shadow
#### Challenge Category: crypto

## Challenge
During his last penetration testing session, one of our consultants successfully obtained root privileges on a Linux server using a local privilege escalation vulnerability, and retrieved the /etc/shadow below. Your mission is to crack the root password. A good wordlist might be useful...

#### Challenge File: *shadow*

## Guide
The shadow file on unix-based systems stores a list of user rows which the system uses to track which users exist on the system. Additionally, each user row contains a string of encrypted information which is used to validate logins whenever the user enters their password.

From the terminal, you can view the entire shadow file provided by using the cat command on it like so:
```bash
$ cat shadow
root:$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1:15758:0:99999:7:::
daemon:*:14971:0:99999:7:::
bin:*:14971:0:99999:7:::
sys:*:14971:0:99999:7:::
sync:*:14971:0:99999:7:::
games:*:14971:0:99999:7:::
man:*:14971:0:99999:7:::
lp:*:14971:0:99999:7:::
mail:*:14971:0:99999:7:::
news:*:14971:0:99999:7:::
uucp:*:14971:0:99999:7:::
proxy:*:14971:0:99999:7:::
www-data:*:14971:0:99999:7:::
backup:*:14971:0:99999:7:::
list:*:14971:0:99999:7:::
irc:*:14971:0:99999:7:::
gnats:*:14971:0:99999:7:::
nobody:*:14971:0:99999:7:::
libuuid:!:14971:0:99999:7:::
Debian-exim:!:14971:0:99999:7:::
statd:*:14971:0:99999:7:::
```

We only care about the first row, the root user entry.
```bash
root:$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1:15758:0:99999:7:::
```

Each colon, ':', is a separator which is used to help parse the row. Let's break down all nine fields of this row:
- root is the user.
- $1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1 is the encrypted password string. We will break this down further in a moment.
- 15758 is how many days since epoch, Jan 1st of 1970, has the password changed.
- 0 represents the minimum number of days before the user is allowed to change their password.
- 99999 represents how many days before the password must change.
- 7 represents the number of days before expiration to warn the user to change their password.
- The next number would be how many days after password expires that the account is still usable.
- The next number would be the account expiration date, and it would be represented in days since epoch again.
- The final empty spot after the last colon has been reserved for the future. In other words, this spot has never been used officially before. It was put there in case a new piece of information would eventually be added to the shadow file. That day never came. Some custom Linux distros might use this for something.

If you ever need to translate any strings representing days since epoch, run the following commands:
```bash
$ passLastChanged=15758
$ date -d "1970-01-01 +$passLastChanged days"
Fri Feb 22 12:00:00 AM EST 2013
```

Now that you understand shadow file rows, let's talk about the encrypted password string.
```bash
$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1
```

This string technically contains separators as well, which are dollar signs in this case. Let's break it down:
- $1$ is the encryption standard used to *hash* the password. In this case, it's MD5.
- .rquYmlo is the *salt* used to randomize the hash. It's standard practice to salt the password at the time of hashing as to prevent *rainbow table attacks*.
- yFWfaSKplZmp1Id2VZ6iT1 is the hash itself, produced using the MD5 algorithm on the plain text password in combination with the randomized salt.

Salts are randomly produced and assigned to each user so that if Bob and Steve have the same password, the hashes are still different. You can play with this yourself by running the following command and seeing the different outputs.

Using *openssl* which is almost certainly built into most Linux distros:
```bash
$ openssl passwd -1 -salt .rquYmlo password
$1$.rquYmlo$aDeMxN0Khkx8FTtd3pBWj/
$ openssl passwd -1 -salt .rquYmlO password
$1$.rquYmlO$5RrE1KzItVMT20Eut4V3a1
```
Note in openssl:
-1 after passwd represents MD5 encryption standard
-5 after passwd represents SHA-256 encryption standard
-6 after passwd represents SHA-512 encryption standard

Using *mkpasswd* found within the **whois** package on most Linux distros:
```bash
$ mkpasswd -m md5 password .rquYmlo
$1$.rquYmlo$aDeMxN0Khkx8FTtd3pBWj/
$ mkpasswd -m md5 password .rquYmlO
$1$.rquYmlO$5RrE1KzItVMT20Eut4V3a1
```

There are a lot more encryption standards out there, so it's good to use a program that can identify the encryption standard for you. One such program is *hashid*, available on most Linux distros via their package manager.
```bash
$ hashid '$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1'
Analyzing '$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1'
[+] MD5 Crypt
[+] Cisco-IOS(MD5)
[+] FreeBSD MD5
```

Knowing the encryption standard used to create this hash is half the battle. Next we need to try and *crack* this hash using a program or an online resource. I'm going to cover two different programs; *john* and *hashcat*. You can always use some online resources for unsalted hashes, but it's very rare to come across unsalted hashes in this day and age.

**How do these programs work in general?** They attempt to *reverse* the hash, though many people use the word *crack*. Both of these programs will require you use a file containing a list of words from a dictionary, a list of known passwords, etc.

The default behavior of these programs is to directly hash each row of the list with the given encryption standard and salt to try and match the encrypted password string. This is known as a *dictionary attack*. There are some different flags that can be applied to try more advanced methods like *mask attacks*, *rule-based attacks*, etc.

The time these programs take depends on how powerful your computer hardware is, the length of the wordlist, and whether we're using standard *dictionary attacks* or more advanced attacks. For the sake of this challenge, we only need a standard dictionary commonly found online titled *rockyou.txt*. If you're on Kali Linux, this file is found at **/usr/share/wordlists/rockyou.txt**, or if you downloaded the *seclists* package available on various distros, **/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz**

Moving forward, you should be using the *seclists* since it contains some of the most advanced dictionaries available for future hash-cracking challenges.

Arch (make sure Multilib is enabled in pacman.conf):
```bash
$ sudo pacman -S seclists
```

Kali or other Debian-based distros:
```bash
$ sudo apt update && sudo apt install seclists
```

You might also have to extract and untar *rockyou.txt*, so run the following commands to do so:
```bash
$ cd /usr/share/seclists/Passwords/Leaked-Databases/
$ sudo tar -xvzf rockyou.txt.tar.gz
```

Before we do anything else, I have copied **$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1** into a file called *hash.txt*. Make a habit of storing your hashes in a text file so that you can easily feed them into programs like *john* and *hashcat*. Copy the string from *shadow* and use the following command with single quotes:
```bash
$ echo 'root:$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1:15758:0:99999:7:::' > john_hash.txt
$ echo '$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1' > hash.txt
```

Using *john*, short for **John the Ripper**, is fairly simple and defaults to using CPU power. I will be leaving out the exact solution output so that you can try this yourself. FYI, *john* requires that unix hashes like the ones in *shadow* contains the user followed by a colon, which is why I created two different *hash.txt* files earlier. Also, unlike *hashcat*, *john* can take an entire row from the *shadow* file as input.

If you're running *john* on this hash for the first time, the following will occur:
```bash
$ john john_hash.txt --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-opencl"
Use the "--format=md5crypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
*password*     (root)
1g 0:00:00:00 DONE (2025-03-30 13:35) 3.846g/s 301292p/s 301292c/s 301292C/s t1234567..myinmortal
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
The password will be where I put in *password* near **(root)**. If you run this program exactly the same again, it won't show the password again unless you use the following command:
```bash
$ john --show john_hash.txt
```

*john* saves all reversed passwords in **/home/$USER/.john/john.pot**. You can now confirm that your *password* is the same as the one in **passwd.txt** by running the following command:
```bash
$ diff <(echo *password*) passwd.txt
```
If nothing gets outputed, then you solved it!

Let's quickly go over using *hashcat*, which will be ideal for longer, more complicated hashes. We use *hashcat* on a system with a graphics card in order to run through dictionary attacks quicker. You should first check whether *hashcat* knows there is a GPU in place and that it is usable:
```bash
$ hashcat -I
```
This will output any usable GPUs. Each device gets an index number which you will make note of for use with the next command. Let's assume the device ID of your GPU is 1.
```bash
$ hashcat -m 500 -d 1 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
...
Dictionary cache building /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt: 33553435 Dictionary cache building /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt: 134213745Dictionary cache built:
* Filename..: /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 1 sec

$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1:*password*

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5))
Hash.Target......: $1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1
Time.Started.....: Sun Mar 30 15:18:35 2025 (0 secs)
Time.Estimated...: Sun Mar 30 15:18:35 2025 (0 secs)
...
```
I cut out a bunch of information specific to my system, but you get the gist. Where I put *password* is where it would show up if *hashcat* is successful. You will also know if it was successful if **Status: Cracked** appears. If **Status: Exhausted**. To see if *hashcat* already has the cracked hash stored in **/home/$USER/.local/share/hashcat/hashcat.potfile**, you can run the following command on the hash file:
```bash
$ hashcat -m 500 hash.txt --show
$1$.rquYmlo$yFWfaSKplZmp1Id2VZ6iT1:*password*
```

While *rockyou.txt* is enough for this exercise, it won't always be enough in the future. The *seclists* package has many different wordlists and dictionaries for you to play with, and it even includes user lists, leaked database passwords, hacked website password lists, etc.

I went into a lot of details about the basics here, but I'll try to stay focused on writing straightforward guides moving forward. This guide gave you all of the foundational knowledge involved in the basics of hash reversing/cracking, but there is much more to learn.
 
#### Solution File: *passwd.txt*
