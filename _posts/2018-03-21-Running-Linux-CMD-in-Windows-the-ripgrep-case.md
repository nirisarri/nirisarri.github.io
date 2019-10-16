Not a new thing, lots of posts there about it..
I have been trying to get it to a nice point, and I think I have been able to.
I installed a new CMD client that actually works better than the one OOB.

Go [HERE](https://github.com/goreliu/wsl-terminal) to get it. It allows changing the default font, which is necessary if you want to run the cool Oh My ZSH extension, on the zsh shell.

Now, I installed `Angular CLI`, `Docker CLI`, `GIT`, and I am in a better place.

Recently I needed to use `RipGrep` on my shell, so I tried the mythical `apt-get`, with no luck. I Went online to find another way to do it, and the installation notes on RipGrep tell you to install using `snappy` (for Linux). That didn't work either. Snappy is not functional in WSL 😖.

However, the Debian install did work! so I ran the commands stated for Debian :

``` sh
$ curl -LO https://github.com/BurntSushi/ripgrep/releases/download/0.8.1/ripgrep_0.8.1_amd64.deb$ sudo dpkg -i ripgrep_0.8.1_amd64.deb
```
That allowed me to run `rg` from my CMD!
