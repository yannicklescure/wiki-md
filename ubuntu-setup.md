# Ubuntu setup

You will find below some useful instructions to set up your computer.

Let's start :rocket:

## Le Wagon setup

Before anything, I recommend following the [Le Wagon setup](https://github.com/lewagon/setup/) guide.

## Oh-my-zsh

Let's install the `zsh` plugin [Oh My Zsh](https://ohmyz.sh/).

In a terminal execute the following command:

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

If asked "Do you want to change your default shell to zsh?", press `Y`

### Display full path

To display `~` instead of the full path `/home/<username>` in your Zsh prompt while still keeping the trailing slash, you can use the `%~` symbol, which represents the current directory but replaces the home directory with `~`.

#### Modify Your Prompt

Here's how to adjust your existing prompt definition:

- Open your `robbyrussell.zsh-theme` file:

  ```bash
  nano ~/.oh-my-zsh/themes/robbyrussell.zsh-theme
  ```

- Update the `PROMPT` to use `%~` instead of `%c`:

  ```bash
  PROMPT="%(?:%{$fg_bold[green]%}➜ :%{$fg_bold[red]%}➜ ) %{$fg[cyan]%}%~%{$reset_color%}\$ "
  PROMPT+='$(git_prompt_info)'
  ```

#### Explanation of Changes

- **`%~`**: Displays the current directory, replacing `/home/<username>` with `~`.
- **`/%`**: Adds the trailing slash after `~` if you are in the home directory.

#### Apply the Changes

After saving your changes, reload your configuration:

```bash
exec zsh
```

#### Final Result

Now, your prompt will show `~` when you are in your home directory:

```bash
 ➜ ~$
```

When you're in other directories, it will show the full path:

```bash
 ➜ /path/to/other/directory$
```

### Additional plugins

- Visit those links to install additional plugins:

  - [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
  - [zsh-completions](https://github.com/zsh-users/zsh-completions)
  - [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
  - [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)

- Update the `plugins` line in the `.zshrc` file:

  ```bash
  plugins=(
    git
    gitfast
    last-working-dir
    common-aliases
    zsh-syntax-highlighting
    zsh-completions
    zsh-autosuggestions
    history-substring-search
    pyenv
    ssh-agent
  )
  ```

### Custom plugins

#### update

Update your system with the terminal by calling `update`.

Create the following file:

```bash
touch ~/.oh-my-zsh/custom/plugins/update/update.plugin.zsh
```

Edit and add the following:

```bash
#!/bin/zsh

update() {
	TEXT_RESET='\e[0m'
	TEXT_YELLOW='\e[0;33m'
	TEXT_RED_B='\e[1;31m'

	sudo snap refresh

	flatpak update -y

	sudo apt update
	echo -e $TEXT_YELLOW
	echo 'APT update finished...'
	echo -e $TEXT_RESET

	# sudo apt dist-upgrade
	# echo -e $TEXT_YELLOW
	# echo 'APT distributive upgrade finished...'
	# echo -e $TEXT_RESET

	sudo apt -y upgrade
	echo -e $TEXT_YELLOW
	echo 'APT upgrade finished...'
	echo -e $TEXT_RESET

	sudo apt -y autoremove
	echo -e $TEXT_YELLOW
	echo 'APT auto remove finished...'
	echo -e $TEXT_RESET

	echo "Cleaning up package cache..."
	sudo apt clean

	echo "System update complete!"

	if [ -f /var/run/reboot-required ]; then
		echo -e $TEXT_RED_B
		echo 'Reboot required!'
		echo -e $TEXT_RESET
	fi
}

# Add the update function to the shell
alias update=update
```

Update the `plugins` line in the `.zshrc` file:

```bash
plugins=(
  ...
  update
)
```

Now, to update your applications in the terminal execute:

```bash
update
```

## Google Chrome

In a terminal execute the following command:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i ~/Downloads/google-chrome-stable_current_amd64.deb

```

## Other applications

### From apt

- Gimp
- Libre Office
- FileZilla

```bash
sudo apt install gimp libreoffice filezilla gnome-tweaks
```

### Flathub

```bash
sudo apt install flatpak
sudo apt install gnome-software-plugin-flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Restart your computer!

### Flathub applications [flathub](https://flathub.org/home)

- [Extension Manager](https://github.com/mjakeman/extension-manager)
- [Audacity](https://www.audacityteam.org/)
- [Audacious](https://audacious-media-player.org/)
- [Vlc](https://www.videolan.org/vlc/)
- [KdeNlive](https://kdenlive.org/en/)
- [Telegram](https://desktop.telegram.org/)
- [Signal](https://signal.org/)
- [Video Trimmer](https://gitlab.gnome.org/YaLTeR/video-trimmer)

```bash
# Extension Manager
flatpak install flathub com.mattjakeman.ExtensionManager
# Audacity
flatpak install flathub org.audacityteam.Audacity
# Audacious music player
flatpak install flathub org.atheme.audacious
# VLC media player
flatpak install flathub org.videolan.VLC
# Kdenlive Video editor
flatpak install flathub org.kde.kdenlive
# Telegram
flatpak install flathub org.telegram.desktop
# Signal
flatpak install flathub org.signal.Signal
# Video Trimmer
flatpak install flathub org.gnome.gitlab.YaLTeR.VideoTrimmer
```

## GNOME Shell Extensions

- [Caffeine](https://extensions.gnome.org/extension/517/caffeine/)
