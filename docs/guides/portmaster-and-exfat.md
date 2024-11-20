# :material-ferry: PortMaster and exFAT

!!! info "Don't let the technical mumbo-jumbo scare you away"

     This section will introduce you to an ongoing issue with PortMaster games on KNULLI. We **strongly recommend** that you **read** this section and **understand** the issue **before** you start setting up your KNULLI device to **avoid frustration**.

**PortMaster** is a GUI tool for managing and installing **video game ports** for **handheld devices**. The PortMaster library covers **retro classics** as well as **modern indie games** and lots of **freeware games** and contains far more than **500 games**. By default, KNULLI comes with an **installer** which will set up PortMaster on your device. You can find in in the *Ports* section. The PortMaster library is a great collection of awesome games that you **should not sleep on**: *Stardew Valley*, *Half-Life*, *TMNT: Shredder's Revenge*, *GTA Vice City*, *AM2R*, *Celeste* and *Owlboy* are just a few examples you might have heard of.

However, If you are **absolutely sure** that you are **not interested** in **ever** playing PortMaster games on your KNULLI device, you can stop reading now. This section is not for you. In any other case: Let's get started.

## The bad news

Unfortunately, many PortMaster games are currently **not compatible** with the **exFAT** file system on KNULLI. exFAT is the default file system of most common SD cards and USB flash drives. Many PortMaster games rely on a concept called **symbolic link** to make the games work on KNULLI. Simply put, a symbolic link is a **pointer** to a file or folder. It basically allows **the same** file or folder to be accessed from **different paths**. A symbolic link is **not a copy** but literally **the same file** in a different place at the same time.

The **exFAT** file system does **not** support symbolic links. Consequently, any PortMaster game which relies on symbolic links will not work as expected on an exFAT-formatted drive. Some games do **not work at all**, others are simply **unable to save and load** any settings or game progress.

## The good news

Luckily, you have a couple of options to enjoy PortMaster games on your KNULLI-driven device anyway. This guide will introduce you to 3 different options how to address the PortMaster/exFAT situation:

* [Stick with ext4](#stick-with-ext4)
* [Be patient](#be-patient)
* [Do something about it](#do-something-about-it)

## Stick with ext4

By default, KNULLI formats its *SHARE* partition to **ext4**. ext4 is a Linux file system which **supports symbolic links**. Consequently, if you simply stick with ext4, it will not make a difference to you whether the game employs symbolic links or bind mounts.

However, ext4 comes at a price: You will **not** be able to **access** your ext4 SD card from a **Windows computer**. Windows does not support the ext4 file system. Hence, you will be restricted to accessing your SD card via Wi-Fi as explained in the [Network Transfer section](../../play/add-games/network-transfer). Unfortunately, network transfer is significantly slower that direct access to the SD card.

Some users consider ext4 a great inconvenience, especially those who want to play on devices **without Wi-Fi capability** (e.g., the Anbernic RG28XX and RG35XX 2024). If Wi-Fi access is not an option for you, you might need to consider **reformatting to exFAT** as explained in the [Formatting section](../../play/add-games/formatting).

## Be patient

Even if you are unable to stick with ext4: **Don't worry**. The mighty Port Navigators, our friends at PortMaster, **have already found a solution** to our common problem.

Instead of relying on symbolic links, they started to employ a similar concept called **bind mount**. When put in layman's terms, the difference between those two concepts is rather subtle: A **symbolic link** is a **file** that is **stored in the file system** and points to another location. In contrast, a **bind mount** simply tells the running operating system to address a file from a different path. It is also a pointer, however, it does **not** have to be stored in the file system. Consequently, it will not cause any issues with **exFAT** drives.

The PortMaster developers have already started to update their games systematically. They will replace every occurrence of symbolic links with bind mounts. However, this process will take time. Right now, in the moment this guide is written, there are **773 ports** available on PortMaster. About **30** of them have **already been updated**. If you are in no rush, you can just sit this one out. Your favorite games will be updated at some point. However, **it might take some time**.

## Do something about it

!!! danger "This approach is for tech-savvy users"

    This subsection was written for **tech-savvy users** who already know their way around a plain text editor and maybe even already have some basic understanding of Linux and/or programming. If you are determined to try and learn, **do not let this stop you**. However, please be aware that unlikely (but possible) mistakes **might** cause **data loss** on your device.

Instead of waiting for the PortMaster maintainers do to the job for you, you could also get your hands dirty and update the games by yourself. All you need is

* the SD card of your KNULLI device (with the ports already installed).
* a proper text editor, e.g., [Notepad++](https://notepad-plus-plus.org).
* a little bit of time.

### Step 1: Update PortMaster and the respective port(s)

Before you go any further: Launch PortMaster and make sure to update all your ports and also PortMaster itself. This is **mandatory**.

### Step 2: Edit the launch script

Each of your PortMaster games will be installed in the `ports` folder within your `roms` folder. Each game will consist of eat least two components:

* a **folder** for game resources (e.g., `roms/ports/stardewvalley`)
* a **launch script** for the game (e.g., `roms/ports/StardewValley.sh`)

The file you want to edit now is the **launch script**. Open it with a **text editor**. **Do not** use a word processor program like Microsoft Word. Use a **plain** text editor. When in doubt, try [Notepad++](https://notepad-plus-plus.org).

### Step 3: Identify the culprit(s)

Search the file for ocurrences of `ln -s` (or something similar, like `ln -sfv`). `ln` is the Linux command for creating a **link**, the parameter `-s` makes the link a **symbolic** link. `f` implies that the creation is **forced**, even if another file is already in that place. `v` is short for **verbose** and simply makes sure that the `ln` command logs its activities to the command line while it is running.

Have a close look at the launch script of your game and make to identify **every occurrence** of `ln`. You will have to **replace all of them**.

In case of *Stardew Valley*, the culprit looks like this:

``` bash
# Setup savedir
$ESUDO rm -rf ~/.config/StardewValley
ln -sfv "$gamedir/savedata" ~/.config/StardewValley
```

* The first line of the above starts with the `#` symbol, which indicates that the line does **not** contain program code but simply a comment to help you understand what the code does.
* The second line launches the `rm` command to **remove** the existing symbolic link at `~/.config/StardewValley` from your device.
* The third line launches the `ln` command to **create a new symbolic link** at `~/.config/StardewValley` which will **point to** `"$gamedir/savedata"`. **Be sure to understand:** The **first** path is the **target** of the pointer, the **second** path is the **location** of the pointer.

In this specific case, you want to replace both lines with a single one like this:

``` bash
# Setup savedir
bind_directories ~/.config/StardewValley "$gamedir/savedata"
```

`bind_directories` is a new PortMaster feature which will create the **bind mount** for you. **Pay attention** though: **The order of both path arguments is reversed!** `bind_directories` expects the **location first**! The **target** of the pointer is the **second** parameter now!

That's it. You did it.

Try to launch your game and see if it works now, if saving and loading your game is possible, even after exiting the game and launching it again. Wasn't that hard now, was it?

### Step 4: Contribute

If you know your way around Git and if you have a Github account, you might even create a **pull-request** at the PortPaster repository now. By proposing your updated file to the Port Navigators, you can make your fix available to other players and contribute to the retro gaming community.