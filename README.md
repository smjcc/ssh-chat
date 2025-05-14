# ssh-chat

A very simple daemonless chat via ssh (or [mosh][11]) that works through your existing [openSSHd][4], written entirely in [execline][5] for security and ease of maintenance.

Users all log into your existing sshd as user 'chat', using their ed25519 public key. Their 'nick' and other settings are associated with that key and preserved.

When accessed via the [mosh][11] client, this is effectively a UDP chat.

## Note

I am actively soliciting feature requests.

## History

I was inspired by the post "[Why aren’t we using SSH for everything?][10]", and thereby discovered the 'real' [ssh-chat][1], but could not get it to compile and run under [musl][2]. [Shazow's][3] ssh-chat is arguably superior on many counts, including that it runs as a dameon, and therefore can do things this ssh-chat cannot, even when no users are online, and that it is compiled and therefore runs with much more potential efficiency and scalability. (execline makes extensive 'execv' calls)

## Differences from [Shazow's][3] [ssh-chat][1]

* This ssh-chat is accessed through your existing [openSSHd][4], so it shares the same port, and adds no additional attack surface.

* Each login is an instance of the 'chat' script, and each instance is in 'peer' communication with each other through pipes. There is no 'server' daemon. When everyone logs out, nothing is running but the sshd.

* Easier to modify and test by editing the [execline][5] script. No server to restart. (chat users have to log out and back in, or issue a 'clear' command, to see the changes)

### [Execline][5]

If you're experienced with the unix command shell environment, and have not yet discovered [execline][5], you're in for a treat. The beauty, simplicity, and resultant security which is inherent to this system, can be fully appreciated only when you fully understand the unix 'exec' system.

An execline script is not interpreted by a shell.  In fact, no shell of any kind is involved anywhere in the entire process of running this ssh-chat.  There *is* a 'parser' (execlineb), but it runs only once, and terminates before running the 'script'. (it 'execs' into the single command line built from the script) This means there is no parser/interpreter running to be confused/tricked by user input. Auditing the security of the script is trivial, at least compared to a shell script.

Making changes to the script is obviously much simpler than to a compiled object, so by all means, personalize your copy...  and let me know what you've done so I can can steal it from you!

## How to install

* Ensure that you have [sys-apps/coreutils][9] and [dev-lang/execline][5] installed on your system. Unless you edit the 'chat' script to use the unix default versions, you'll need to install [sys-apps/s6-portable-utils][6]. I've also used [app-admin/pwgen][7] to assign an inital random 'nick', and [sys-libs/ncurses][8] for colors, and for a fake "status line".

* Create the user account for chat.  I use 'chat' as the username. Ensure that the account entry in /etc/shadow has '*' in the password field (not '!'), so that sshd will allow only public key logins, and ensure that the chat script is the user's shell. e.g.:

/etc/passwd
``` code
chat:x:2000:2000::/home/chat:/home/chat/.bin/chat
````
/etc/group
```` code
chat:x:2000:
````
/etc/shadow
```` code
chat:*:17868::::::
````
/etc/gshadow
```` code
chat:!::
````

* Install the script, and the directory structure to the user's home directory, and set file and directory permissions. e.g:

```` console
# mkdir /home/chat/.bin /home/chat/.channels /home/chat/.keys /home/chat/.nicks
# mkdir /home/chat/.channels/main
# mv /path-to/chat /home/chat/.bin/
# chown root:chat -R /home/chat/
# chmod 770 -R /home/chat/
# chmod 710 /home/chat /home/chat/.bin
# chmod 650 /home/chat/.bin/chat
````

* Edit your /etc/ssh/sshd_config ( or /etc/ssh/sshd_config.d/whatever ) to add the following, which will allow everyone with an ed25519 key to log in as user 'chat':

``` code
PermitUserEnvironment chatkey

Match User chat
	AuthorizedKeysCommand /bin/echo environment=\"chatkey=%k\" ssh-ed25519 %k
	AuthorizedKeysCommandUser nobody
```

## License

I've chosen the ISC license out of deference to, and respect for skarnet.

## Foibles

* I wrote this over a weekend. There are doubtless many tweaks to be made or improved, but I know of no bugs. (editing and testing the code is so easy, that I'd just fix them if I knew of any)

* I've done very minimal testing.  I expect there to be ncurses issues with access from terminals other than linux, lemme know.  It also needs a /proc filesystem available.

* If a user changes his window size (SIGWINCH), 'chat' will behave poorly until the user logs out and back in again, or issues a 'clear' command, which reloads the script. (the SIGWINCH occurs on the user's client) I suppose I could 'poll' for changes, but why bother?

[1]: https://github.com/shazow/ssh-chat
[2]: https://musl.libc.org
[3]: https://github.com/shazow
[4]: https://www.openssh.com/
[5]: https://www.skarnet.org/software/execline/
[6]: https://www.skarnet.org/software/s6-portable-utils/
[7]: https://sourceforge.net/projects/pwgen/
[8]: https://www.gnu.org/software/ncurses/
[9]: https://www.gnu.org/software/coreutils/
[10]: https://medium.com/@shazow/ssh-how-does-it-even-9e43586e4ffc
[11]: https://mosh.org
