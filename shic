#!/bin/bash

# Defaults
[[ -z $SHIC_HOST ]] && SHIC_HOST="irc.freenode.net"
[[ -z $SHIC_PORT ]] && SHIC_PORT=6667
[[ -z $SHIC_NICK ]] && SHIC_NICK="$USER"
[[ -z $SHIC_PASS ]] && SHIC_PASS=""
# Automatically execute these inputs at startup, separated by ;
# e.g: SHIC_SCRIPT=":j #archlinux; Heya all!; :s;
[[ -z $SHIC_SCRIPT ]] && SHIC_SCRIPT=""
# Red error, green background for private message, cyan for #archlinux,
# white for conversations in and out, and gray for everything else
[[ -z $SHIC_PREFIX ]] && SHIC_PREFIX=(
    "\e[31m::^ERROR"
    "\e[42m\e[30m::(^<[^@]*@[^#])"
    "\e[36m::#archlinux"
    "\e[0m::^<"
    "\e[0m::^->"
    "\e[1;30m::(.*)"
)

# Read config files
[[ -r "$HOME/.shicrc" ]] && source "$HOME/.shicrc"
_xdgconf="${XDG_CONFIG_HOME:-$HOME/.config}/shic/shicrc"
[[ -r "$_xdgconf" ]] && source "$_xdgconf"

# Don't exit at Ctrl-C
trap "echo" SIGINT

# Clean up children at exit
trap "kill 0" EXIT

# Send raw message to server
function _send() {
    printf "%s\r\n" "$*" >&3
}

# Print for user
function _output() {
    _prefix=""
    for rule in ${SHIC_PREFIX[@]}; do
        [[ "$@" =~ ${rule#*::} ]] && _prefix="${rule%%::*}$_prefix"
    done

    printf "$_prefix%s\e[0m\n" "$*"
}

# Handle user input
function _input() {
    local line="$@"
    if [[ "${line:0:1}" != ":" ]]; then
        [[ -z $channel ]] && _output "ERROR: No channel to send to" && return

        _send "PRIVMSG $channel :$line"
        _output "-> $channel> $line"
        return
    fi

    if [[ ${#line} == 2 || ${line:2:1} == " " ]]; then
        _txt="${line:3}"
        case ${line:1:1} in
            m ) read -r _to _msg <<< "$_txt" && _send "PRIVMSG $_to :$_msg" && _output "-> $_to> $_msg"; return;;
            l ) read -r _from _msg <<< "$_txt" && _send "PART $_from :$_msg"; return;;
            j ) _send "JOIN $_txt"; [[ -z $channel ]] && channel=$_txt; return;;
            s ) channel="$_txt";  return;;
            q ) _send "QUIT"; exit 0;;
        esac
    fi

    # Not recognized command, send to server
    _send "${line:1}"
}

# Parse command line
while getopts "h:p:n:k:c:v" flag; do
    case $flag in
        v) printf "shic v. 0.1, by halhen. Released to the public domain.\nSee http://github.com/halhen/shic for help.\n"; exit;;
        h) SHIC_HOST="$OPTARG";;
        p) SHIC_PORT="$OPTARG";;
        n) SHIC_NICK="$OPTARG";;
        k) SHIC_PASS="$OPTARG";;
        c) source "$OPTARG";;
        ?) printf "Unknown option. Usage: $0 [-h hostname] [-p port] [-n nick] [-k password] [-c configfile] [-v]\n" >&2; exit 1;;
    esac
done

# Open connection to server
exec 3<>/dev/tcp/$SHIC_HOST/$SHIC_PORT || exit 1

# Handle messages from server
# This runs as a separate process, which means that no variables are shared with 
# the input process. For better or for worse. Mostly for worse.
{
    while read _line; do
        [[ ${_line:0:1} == ":" ]] && _source="${_line%% *}" && _line="${_line#* }"
        _source="${_source:1}"
        _user=${_source%%\!*}
        _txt="${_line#*:}"

        case "${_line%% *}" in
            "PING")
                _send "PONG" ;;
            "PRIVMSG")
                _ch="${_line%% :*}"
                _ch="${_ch#* }"
                _output "<$_user@$_ch> $_txt" ;;
            *)
                _output "$_source >< $_line" ;;
        esac
    done
} <&3 &

# Introduce myself
[[ $SHIC_PASS ]] && _send "PASS $SHIC_PASS"
_send "NICK $SHIC_NICK"
_send "USER $SHIC_NICK localhost $SHIC_HOST :$SHIC_NICK"

function _trim() { echo $1; }

# Execute login script
IFS=";" read -ra C <<< "$SHIC_SCRIPT"
for _cmd in "${C[@]}"; do
    _input $(_trim "$_cmd")
done

# Handle input
while read -e line; do
    _input "$line"
done
