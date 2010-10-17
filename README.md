# shic - SHellscript Irc Client

`shic` is a simple IRC client written in pure BASH. It is inspired by [sic](http://tools.suckless.org/sic). `shic` supports coloring and configuration files.

`shic` came to be as a learning excercise of BASH. The design principles are:

* Pure BASH 
* High functionality-to-code ratio
* Few lines of code

## Options
`shic` accepts the following options

* -h hostname: hostname of IRC server
* -p port: port of IRC server
* -n nickname: nickname and username
* -k password: password for username
* -c configfile: load configfile (see below)
* -v : print version info and exit

## Configuration
Besides giving `shic` command line options, configuration can be stored in a separate file, and read with the -c option. `shic` `source`:s this file, so it must be valid BASH.

Below is and example of a valid configuration file, here set with the defaults


    SHIC_HOST="irc.freenode.net"
    SHIC_PORT=6667
    SHIC_NICK="$USER"
    SHIC_PASS=""
    # Red error, green background for private message, cyan for #archlinux,
    # white for conversations in and out, and gray for everything else
    SHIC_PREFIX=(
        "\e[31m::^ERROR"
        "\e[42m\e[30m::(^<[^@]*@[^#])"
        "\e[36m::#archlinux"
        "\e[0m::^<"
        "\e[0m::^->"
        "\e[1;30m::(.*)"
    )

`SHIC_PREFIX` is an array of prefix::regexp pairs. If an output line matches regexp (through the BASH `=~` operator), prefix will be printed before the line. This is best used to set different colors for different messages. *Gotcha/known bug: spaces are not allowed in the regexp.*

## Commands
All messages are sent to the current channel (set using `:s`, see below). All input starting with `:` is considered a command. If the command is not one of the below, the leading `:` is stripped and the command is sent to the server. Use this for other IRC features, e.g. `:NICK mynewnick`.

* `:j #channel` - join #channel
* `:l #channel` - leave #channel
* `:m #channel/user message` - send message to #channel or user
* `:s #channel/user` - set #channel or user as the current channel
* `:q` - exit

## FAQ

### How is shic licensed?
It is public domain. Do what you want with it, no warranties. And don't feed the trolls.

### shic doesn't do X, like Y does
Nope. But Y isn't 100-ish lines of BASH, including configuration and input. However, let me know if you have an idea.

### Chat messages destroy my input when I write
Yes, they do. If you know of a BASH way to solve this in few lines of code, let me know.

### Your BASH skills suck. You should use X instead of Y
Yes. I'm only now moving from beginner to not-as-much-beginner in shell scripting. If X is a better way to do Y in BASH, I'm happy to be taught.

### I found a bug
I'm sure you did. My improving BASH skills are only slighly better than my lacking knowledge of IRC. Please tell me, and I will try to fix it.

### shic made me write in the wrong channel / kicked me out of a conversation / ate my homework
No warranties. Sorry.

### This client is shit
SHellscript Irc Tool? Seriously, though: use it if you want to.

### This client is *the* shit
Glad you like it. Make me happy by telling mig. I'm occasionally at Freenode as halhen. Try `:m halhen Hey man. shic is chic!`. Or [send me an e-mail](mailto:halhen@k2h.se).
