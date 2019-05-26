Formed in 2012, Feather Information Technology LLC is committed to exploring technological frontiers, helping defend our country, and staying true to our core values.

* **Integrity**: We’re fully committed to ethical business practices and treating our customers and employees with the respect they deserve. This means doing things right the first time, confronting issues head-on, and treating everyone as a valued member of the team.
* **Innovation**: No mission is cookie-cutter-simple and every project we work on, whether it’s reshaping communications networks or streamlining cloud computing interfaces, we strive to to think outside the box.
* **Environmental Impact**: Everything we build affects our environment for better or for worse; and today industry is beginning to catch up with FeatherIT’s environmentally friendly stance, making this an exciting time to marry technology with a color (green!).



# dotfiles

## Nobody expects dotfiles to be in Ansible

When I first created a dotfiles repository I think I had the usual motivations and plan.

1. Get excited that my configuration files could be centrally managed & accessible.
2. Put .bashrc in an empty git repository
3. Profit

I'm also really comfortable with bashing the shells so I completed my beautiful, clean, **simple dotfiles repo** by adding a `setup.bash`.^1


```
#!/bin/bash

echo ""
echo "---------------------------------"
echo "SETTTING UP YOUR BASH ENVIRONMENT"
echo "---------------------------------"

DOTFILES=$(pwd)
echo -e "\nDotfiles are in ${DOTFILES}\n"

echo "------"
echo "BASHRC"
echo "------"
echo ""

if [ -f ~/.bashrc ]; then
    if [ ! -L ~/.bashrc ]; then
        backup_to="~/.bashrc.$(date +"%Y%m%d%H%M%S").bak)"
        echo "Backing up bashrc to ${backup_to}"
        mv ~/.bashrc ~/.bashrc.$(date +"%Y%m%d%H%M%S").bak
    else
        echo "Your bashrc is a symlink which is being deleted"
        file ~/.bashrc
        rm ~/.bashrc
    fi
fi
if [ ! -e ~/.bashrc ]; then
    echo "linking in the new bashrc"
    ln -s $(pwd)/.bashrc ~/.bashrc
else
    echo "something unexpected is at ~/.bashrc, we're aborting and not overwriting it!"
fi

echo -e "\n"
```
Time passed and I added [Oh My Tmux](https://github.com/gpakosz/.tmux). This required pulling down a git repo and dropping a couple of config files in place. I'm totally graduating to a **mature dotfiles repo** that has multiple configurations and a single install method!

The only problem was that my new section was looking really similar to my previous section- soft linking the tmux config files was literally a copy-paste effort from my .bashrc code

```
...
if [ -f ~/.tmux.conf.local ]; then
    if [ ! -L ~/.tmux.conf.local ]; then
        backup_to="~/.tmux.conf.local.$(date +"%Y%m%d%H%M%S").bak)"
        echo "Backing up tmux config to ${backup_to}"
        mv ~/.tmux.conf.local ~/.tmux.conf.local.$(date +"%Y%m%d%H%M%S").bak
    else
        echo "Your .tmux.conf.local is a symlink which is being deleted"
        file ~/.tmux.conf.local
        rm ~/.tmux.conf.local
    fi
fi
...
```

Ok, no need to panic. I can computer the codes. What I needed was a simple way to not rewrite code and leverage pre-existing work... I already wrote that freaking soft link code... hmmm... definitely time for function library.

I know, you read the title of this article already and know that this idea was at best sub-optimal. But that's what happens in the real world. It doesn't matter how much effort you put into something ahead of time, eventually flaws in your system jump out at you and you need to refactor. Admittedly, I jumped into this project without a lot of planning but I had seen dotfile repos before and the pattern I was following was fairly common. Well, at least the part where people throw config files into a repo and write a quick install script. Anyway, back to our hero and his misguided quest to use 


Wait a second. I can do better than this, right? I'm totally reusing chunks of code , before we get too far it's time for a function library

```
#!/usr/bin/env bash

function drop_in_file() {
    local var_path=$1
    local var_filename=$2

    local var_date=$(date +"%Y%m%d%H%M%S")    

    local URL=${1}/${2}
    local URL_BAK=${URL}.${var_date}.bak

    if [ -f ${URL} ]; then
        if [ ! -L ${URL} ]; then
                echo "* Backing up ${URL} to ${URL_BAK}"
                mv ${URL} ${URL_BAK}
        else
                echo "* ${URL} is a symlink which is being deleted"
                rm ${URL}
        fi
    fi
    if [ ! -e ${URL} ]; then
        echo "* linking in the new ${var_filename}"
        ln -s $(pwd)/${var_filename} ${URL}
    else
        echo "* something unexpected is at ${URL}, we're aborting and not overwriting it!"
    fi


}
...
```
After a while my repo started looking as you might expect- a giant pile of junk; random configuration files everywhere. Not a problem, I'm good at the bashing so I leveled up to a *mature dotfiles repo* by creating a `setup.sh` that makes sure my configs and their binaries are installed. It went something like this


Well dang. Every time I added a new configuration file my setup.sh needed a new section and my sections were beginning to look sort of similar. I'm super good at the bashes so I knew that the next best thing was to make an *advanced dotfiles repo*

1. create lib.sh



2. realize there's no good way to insall them so make a setup.sh
3. The concept we're going with is

4. install ansible & git on your own
5. pull down the dotfile repo
6. run whichever playbook makes sense for the machine


## Playbooks

| Playbook | Description
|---|---|
| workstation.yml | Install all the things
| remote-ws.ymp | tmux, git,  


^1: Yes, that's right- I use `.bash` for most of my shell scripts. It's not because [Ubuntu aliases /bin/sh to dash](https://wiki.ubuntu.com/DashAsBinSh), it's because I actually care about bash features and occasionally use things like mapfile and associative arrays which don't exist in /bin/sh.
