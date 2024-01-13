# Installation - Terminal

## Byobu

```
sudo apt install -y byobu
```

## Oh-my-zsh

Remove the existing installation of oh-my-zsh
```
rm -rf .oh-my-zsh/
```
Install oh-my-zsh
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## P10k

Follow the instructions on https://github.com/romkatv/powerlevel10k#installation

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

Don't forget to logou&login to load p10k

### Configure plugins

vi .zshrc

plugins=(
  evalcache
  git
  git-extras
  debian
  tmux
  screen
  history
  extract
  colorize
  web-search
  docker
  zsh-syntax-highlighting
  zsh-autosuggestions
)


### Install missing plugins

```
git clone https://github.com/mroth/evalcache ~/.oh-my-zsh/custom/plugins/evalcache
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

## fzf

```
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

## Other tools

Add the user to the sudoers.
Install additional tools:
- kubectl
- helm
