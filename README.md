NOTE: While the repo contains a fork of the original GPGTools/pinentry-mac, the instructions below use the
[homebrew cask](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/pinentry-mac.rb) which thus uses the original repository code.
As there is no real code change, that is perfect. This repo remains for the instructions on setting it up (and in case the GPGTools edition disappears due to their licensing plans)

# Pinentry MAC / OSX

[This](https://github.com/massar/pinentry-mac) is, obviously, a very quick fork of GPGTools/pinentry-mac.

pinentry-mac is the only component one needs to get a working combination of:
 * [Apple OS X Yosemite (10.10)](http://www.apple.com)
 * [Thunderbird](https://www.mozilla.org/thunderbird/)
 * [Enigmail](https://www.enigmail.net)
 * [Homebrew](http://brew.sh/)'s [version](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/gnupg2.rb) of [GnuPG2](https://www.gnupg.org/)

Without the need for GPGSuite/MacPGP and other such things.
Due to homebrew you'll always have an up to date gnupg2.

Possible [installer script](https://github.com/massar/pinentry-mac/issues/2) to make things easier might follow.

# Bugs / patches etc

Please file them in the [Issue Tracker](https://github.com/massar/pinentry-mac/issues)

# Code License

As the original license for the 'pinentry' program where 'pinentry-mac' was derived from is GPLv2 (this code is of course also GPLv2](https://github.com/massar/pinentry-mac/blob/master/COPYING).

# Installation

Following are the steps to get this all working.

# Homebrew

Follow instructions on http://brew.sh

Effectively:
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

# gnupg2 + gpg-agent + pinentry-mac

Just run the following to install these
```
brew install gnupg2 gpg-agent pinentry-mac
```

## Autostart gpg-agent

Install a Launch Agent that starts gpg-agent:

Replace YOURUSERNAME with your real username!

```
cat >~/Library/LaunchAgents/org.gnupg.gpg-agent.plist <<HERE
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
   "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>org.gnupg.gpg-agent</string>
    <key>ProgramArguments</key>
    <array>
      <string>/Users/YOURUSERNAME/bin/start-gpg-agent.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
  </dict>
</plist>
HERE

mkdir ~/bin
chmod 700 ~/bin
cat >~/bin/start-gpg-agent.sh <<HERE
if test -f \$HOME/.gpg-agent-info && \
  kill -0 \`cut -d: -f 2 \$HOME/.gpg-agent-info\` 2>/dev/null; then
    GPG_AGENT_INFO=\`cat \$HOME/.gpg-agent-info\`
    export GPG_AGENT_INFO
else
  eval \`/usr/local/bin/gpg-agent --daemon --write-env-file\`
fi
export GPG_TTY=\`tty\`
HERE
chmod 700 ~/bin/start-gpg-agent.sh
```

Append the following to ~/.bash_profile
```
export GPG_TTY=$(tty)
if [ -f "${HOME}/.gpg-agent-info" ]; then
  . "${HOME}/.gpg-agent-info"
  export GPG_AGENT_INFO
fi

PS1='\u@\h:\w\$ '
```
and make sure it is executable:
```
chmod 700 ~/.bash_profile
```

Configure GPG Agent. Replace GITPATH with where you typed 'git clone ...'
```
echo >~/.gnupg/gpg-agent.conf <<HERE
default-cache-ttl 600
max-cache-ttl 999999
enable-ssh-support
pinentry-program=GITPATH/pinentry-mac/build/Release/pinentry-mac.app/Contents/MacOS/pinentry-mac
HERE
```

Now, reboot (because of launchd and environment vars).
 
## Thunderbird

Go to https://www.mozilla.org/en-US/thunderbird/ and install it

## Enigmail

Start previously installed Thunderbird, go to Addons and install Enigmail.

## Configure gnupg2

Make sure that you set up your public/secret key etc. Standard GnuPG setup applies.

## Use it :)

As all the pieces are there, Enigmail should autodetect gnupg2; the environment is properly set up and all thus should in theory work.

# What is different from the GPGSuite edition?

You only get pinentry-mac from it.

Thus you do not get the rest of the GPGSuite functions, but for usage on the commandline (for decrypt/encrypt) or inside Enigmail that is not needed.

