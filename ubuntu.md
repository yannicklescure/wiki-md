# Ubuntu setup

## Oh-my-zsh

Let's install the `zsh` plugin [Oh My Zsh](https://ohmyz.sh/).

In a terminal execute the following command:

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

If asked "Do you want to change your default shell to zsh?", press `Y`

### Custom plugins

### aptupdate

Update your system with the terminal by calling `aptupdate`.

In a terminal execute the following command:

```bash
mkdir ~/.oh-my-zsh/custom/plugins/aptupdate
touch ~/.oh-my-zsh/custom/plugins/aptupdate/aptupdate.plugin.zsh
x=~/.oh-my-zsh/custom/plugins/aptupdate/aptupdate.plugin.zsh; echo "#\!/bin/zsh\n\naptupdate() {\n\tTEXT_RESET='\\\e[0m'\n\tTEXT_YELLOW='\\\e[0;33m'\n\tTEXT_RED_B='\\\e[1;31m'\n\n\tsudo apt update\n\techo -e $TEXT_YELLOW\n\techo 'APT update finished...'\n\techo -e $TEXT_RESET\n\n\t# sudo apt dist-upgrade\n\t# echo -e $TEXT_YELLOW\n\t# echo 'APT distributive upgrade finished...'\n\t# echo -e $TEXT_RESET\n\n\tsudo apt -y upgrade\n\techo -e $TEXT_YELLOW\n\techo 'APT upgrade finished...'\n\techo -e $TEXT_RESET\n\n\tsudo apt -y autoremove\n\techo -e $TEXT_YELLOW\n\techo 'APT auto remove finished...'\n\techo -e $TEXT_RESET\n\n\tif [ -f /var/run/reboot-required ]; then\n\t\techo -e $TEXT_RED_B\n\t\techo 'Reboot required!'\n\t\techo -e $TEXT_RESET\n\tfi\n}\n" >> "${=x}"
l=$(grep -o "^plugins=.*[^\)]" ~/.zshrc | cut -d : -f 1); sed -i -e "s/$l/$l aptupdate/g" ~/.zshrc
source ~/.zshrc
```

To see the result, in a terminal execute the following command:

```bash
cat ~/.oh-my-zsh/custom/plugins/aptupdate/aptupdate.plugin.zsh
```

Result should look like that:

```zsh
#!/bin/zsh

aptupdate() {
    TEXT_RESET='\e[0m'
    TEXT_YELLOW='\e[0;33m'
    TEXT_RED_B='\e[1;31m'

    sudo apt update
    echo -e
    echo 'APT update finished...'
    echo -e

    # sudo apt dist-upgrade
    # echo -e
    # echo 'APT distributive upgrade finished...'
    # echo -e

    sudo apt -y upgrade
    echo -e
    echo 'APT upgrade finished...'
    echo -e

    sudo apt -y autoremove
    echo -e
    echo 'APT auto remove finished...'
    echo -e

    if [ -f /var/run/reboot-required ]; then
        echo -e
        echo 'Reboot required!'
        echo -e
    fi
}
```

Now, to update your applications using the terminal, execute the following command:

```bash
aptupdate
```
