---
layout: post
title: My dotfiles
subtitle: Configuring your development environment and tools
---

Your dotfiles hold the configurations that control your entire work environment, from how they look to hows they should act. These are my dotfiles, and the system I use to manage them.

## What is in here?

First off, [here are my dotfiles](https://github.com/Avyiel/dotfiles). I version control everything, and use GitHub to host them. The stuff I save is really a small set of tools and/or programs, but it still accounts for almost every tool I use in the command line:

- tmux
- nvim
- zsh
- git
- terminfo

So, onto my setup: I use iTerm 2 as my terminal emulator with tmux to handle panels, zsh as my shell and nvim as my text editor (I also use VS Code, but that's for another post). I also use git as my version control tool.

I **strongly** recommend that you take a look at all the files before doing anything. Never run random scripts from the internet without reading them first, even if you "trust" the source.

## Pre-requisites

To get started with my dotfiles, there are a few pre-requisites:

1. Having Xcode Command Line Tools (CLT) installed
2. Having git installed

All the rest will be handled by the script. Git is there to clone the repository, and Xcode for its build tools. You will have to run `xcode-select --install` after installing Xcode, and git comes pre-installed in macOS so no extra work for you.

## Usage

To use my dotfiles, the easiest way is to clone the repository and run the `bootstrap.sh` script. It will handle installing homebrew (and all the bottles and casks you'll need), oh-my-zsh, vim-plug and much more! I'll go over the details below.

First off, clone the repository:

{% highlight shell %}
git clone git@github.com:Avyiel/lucasvienna.dev.git ~/dotfiles
{% endhighlight %}

This will download the GitHub repo I linked above and put it in `~/dotfiles`. Then, move into the cloned folder and run the script:

{% highlight shell %}
cd ~/dotfiles
./bootstrap.sh
{% endhighlight %}

And... that's it. Theoretically, you're done. The script will handle everything in a hands-off manner. You might have to press enter a few times to confirm changes or installations though.

If you want to host your own copy of these dotfiles, reset the local repo and initialize your own:

{% highlight shell %}
rm -rf .git
git init
git remote add <your repository URL here>
git add .
git commit -m 'Initial commit'
git push -u origin all
{% endhighlight %}

## Explanation

Or "what the hell will this script do to my beloved machine?".

### Bootstrap

Let's start with what's in `bootstrap.sh`:

{% highlight shell %}
git pull origin master
{% endhighlight %}

This line simply pulls the latest changes from the repository. Just to make sure you're up-to-date with my latest additions or fixes. Then we have:

{% highlight shell %}
function spaceship() {
  # This installs the spaceship theme for zsh
  # https://github.com/denysdovhan/spaceship-prompt
  if [ -d "$ZSH/custom/themes/spaceship-prompt" ]
  then
    echo "spaceship-prompt is already installed, skipping..."
  else
    git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH/custom/themes/spaceship-prompt"
    ln -s "$ZSH/custom/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH/custom/themes/spaceship.zsh-theme"
  fi
}
{% endhighlight %}

This defines a function called `spaceship` that will, as per the comment, install the spaceship theme for zsh. It checks for an already-present installation and then clones and symlinks the theme. Pretty simple stuff. Moving on:

{% highlight shell %}
function italicTerm() {
  tic -x terminfo/xterm-256color-italic.terminfo
  tic -x terminfo/tmux-256color.terminfo
}
{% endhighlight %}

Another function. This one adds the two terminfo files to your system. This enables TrueColor and italics for your terminal, so that it can look extra-pretty. Next up:

{% highlight shell %}
function doIt() {
  ./brew.sh

  # install homebrew
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  # install vim-plug
  curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
      https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  # install oh-my-zsh
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" "" --unattended

  # link dotfiles
  stow tmux
  stow nvim
  stow zsh
  stow git

  # install nvim plugins
  nvim +PlugClean! +qall
  nvim +silent +PlugInstall +qall
  python3 ~/.config/nvim/plugged/YouCompleteMe/install.py

  # launch the spaceship
  spaceship
}
{% endhighlight %}

The meat of this script. It "just does it" all. First off, we're calling another script: `brew.sh`. We'll go over what that one does a bit later. Then we install homebrew (an awesome package manager for macOS), vim-plug (a plugin manager for vim/neovim), and oh-my-zsh (a theme/plugin manager for zsh). Afterwards, it uses GNU `stow` to symlink all the actual dotfiles into your home directory. Stow is pretty neat in that it keeps the folder structure intact while doing the symlinking.
Then we set neovim up with its plugins, and lastly we install our theme.

{% highlight shell %}
if [ "$1" == "--force" -o "$1" == "-f" ]; then
  doIt
  italicTerm
else
  read -p "This may overwrite existing files in your home directory. Are you sure? (y/n) " -n 1;
  echo "";
  if [[ $REPLY =~ ^[Yy]$ ]]; then
    doIt
    italicTerm
  fi;
fi;
{% endhighlight %}

This part asks you if you actually want to do it. You can also call the script with `--force` or `-f` to skip the prompt and "just do it". Lastly:

{% highlight shell %}
unset doIt
unset italicTerm
unset spaceship
{% endhighlight %}

We unset the functions created so you won't accidentally mess your system up. Neat!

You should be saying something in the lines of "hold up! There's another script running here that you didn't explain yet", and you're absolutely right. Onto `brew.sh`.

### Homebrew Bottles & Casks

This pretty little script is a big mess, really. It calls `brew install` **2** times. I could use backslashes and do it all in a single command. I might change that at a later date, but for now it serves its purpose: to pour bottles and casks. That's homebrew slang for "install CLI programs and macOS applications". Instead of going over all the code in the script (I still recommend you go through it before running `bootstrap.sh`), I'll just list all the stuff it installs. If you don't need/want anything, simply remove that line or comment it out.

#### Bottles
- coreutils
- findutils
- gnu-sed
- grep
- openssh
- cmake
- wget
- curl
- stow
- gnupg
- pinentry-mac
- tmux
- reattach-to-user-namespace
- the_silver_searcher
- ag
- python3
- dart
- go
- docker
- docker-compose
- docker-machine
- git
- git-flow
- prettyping
- tree
- ssh-copy-id
- yarn
- neovim
- gradle
- usbmuxd
- libimobiledevice
- ideviceinstaller
- ios-deploy
- cocoapods

#### Casks
- adoptopenjdk8
- docker
- fastlane
- android-studio
- alfred
- google-chrome
- firefox
- slack
- skype
- discord

Git is in the list because the version bundled with macOS is usually outdated. The casks (or apps) I install are a basic set of widely used ones, such as browsers, comms apps, helpers, and tools. There are also tools required by nvim (such as silver searcher and ag), by Flutter (the last 5 bottles), the entire Docker suite and some random (but very useful) tools like prettyping and git-flow (if you happen to use that workflow).

Lastly, let's talk about the actual configuration in my dotfiles.

## The dotfiles themselves
### git
Stow will link two files: `.gitconfig` and `.gitignore_global`. The latter tells git which files to _not_ track. The first one sets git up. There are a few things you should change, under the `[user]` tag:

{% highlight shell %}
name = Lucas Vienna
email = dev@lucasvienna.dev
signingkey = 1E763A1C0C9B66CD
{% endhighlight %}

This is obviously my own information and data. You should put your own info in there. This config file also sets `vimdiff` as the merge tool, tells git to GPG-sign all commits, sets the default pull behavior to rebase instead of merge and adds a few handy aliases.

### tmux
Now this is where it gets interesting. tmux is argually the best **t**erminal **mu**ltiple**x**er out there, and I love using it. No more dropped SSH connections, no more fiddling around with a ton of tabs or windows and no more ugly emulator. Stuff you might want to change:

- `set -g prefix C-a`: the prefix for tmux commands. I use Control + a
- `set-option -g status-position top`: status bar position. I put it on top
- `set -g @plugin`: lots of things start with this command. Check to see if the plugins suit you

### neovim
IMO the best editor for the CLI. I use VS Code for the most part, but neovim is also excellent. There aren't any defaults you _must_ change (unlike git), so just go over the entire thing and modify it to your liking.

### zsh / oh-my-zsh
I love this shell. Autocompletion works wonders, plugins are amazing and we even get syntax highlighting. Things you should definitely change in `.zshrc`:

- `export ZSH="/Users/lucas/.oh-my-zsh"`: change this to point to your own home directory. Absolute path is better here
- `export SSH_KEY_PATH="~/.ssh/rsa_id"`: point this to wherever your key is

For the rest, go over the file and change it to suit you. It's heavily commented and you can always just nuke it (oh-my-zsh will create a new one for you).

Also check the stuff in `.zsh_aliases` under the `# Directories` and `# SSH` comments. These point to work dirs in my machine, maybe delete them or add your own stuff. Also, read over the rest of the aliases, some of them are really useful.

I don't have git aliases set up in there because of the git plugin for oh-my-zsh, except for a few extras not already included.

And... that's all I have for now. Have fun with your new dotfiles!