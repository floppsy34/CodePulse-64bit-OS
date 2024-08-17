Let's breakdown the contents of the Dockerfile...

    FROM randomdude/gcc-cross-x86_64-elf`

If you are familar with Python, the *FROM* command is similar to the `pip install` command for python libraries. Docker will search it's database to find the 'library' with this name so we can download it to our computer and use what's inside, awesome! The base image or 'library' in this case is `randomdude/gcc-cross-x86_64-elf`.

By running the following command in our terminal, we can open the base image to see what's inside. Please note: this command will download the base image locally to your computer, but once we're done exploring it will remove it for us so we don't need to worry about deleting it.

    docker run -it --rm <base-image> /bin/sh

After opening, the terminal line will have a # symbol at the front indicating we are inside the container, neat! Let's list what's inside the container by running:

    ls

The output should be something like this:

`bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var`

What's this? This is looking suspiciously like a small Debian OS image. A Debian image will usually have the package manager 'dpkg' installed, let's have a look for it using:

    ls usr/bin

It should be there in the listed output. More importantly from this list we can see a whole bunch of useful stuff is installed:

- The C compiler `gcc`
- The whole C compiler toolchain (GNU - including the assembler `x86_64-linux-gnu-as` and the linker `x86_64-linux-gnu-ld`)

We can certainly cook up an OS with this! Remember to run the following command to exit from and remove the base image from our computer when we are done exploring.

    exit

So when we run this file, Docker puts this small Debian image (with C compiler and compiler toolchain) into a small container inside our computer. It's put in this container so it cannot mess with anything else on our computer (and if needed we can just nuke it). We will use it to build our operating system. Groovy!

    RUN apt-get update

Runs apt-get update in our Debian image.

    RUN apt-get upgrade -y

Runs apt-get upgrade in our Debian image and automatically answers 'yes' when prompted if you are sure you want to proceed with the upgrade.

    RUN apt-get install -y nasm

Installs `nasm` onto our Debian image which stands for Netwide Assembler and is specifically build to compile assembly code for x86 architectures (including 16-bit, 32-bit and 64-bit architectures).

But didn't I say we have an assembler already, `x86_64-linux-gnu-as`? Why not just use that one? Well, you can if you want! But this is why you might want to use NASM instead:

- It uses 'Intel syntax' not 'AT&T syntax' (like GNU assembler does). Which is a more readable way to write assembly code that more people can understand across many different professional backgrounds. For open-source development, readability means accessibility for everyone!
- You can use macros, conditional assembly, and includes in your assembly code which can really make complex code simple, and we do like making things simpler!

    RUN apt-get install -y xorriso

Installs `xorriso` onto our Debian image, this tool creates, modifies and extracts ISO 9660 filesystem images. We will use this to create our final bootable image!

    RUN apt-get install -y grub-pc-bin

Downloads `grub-pc-bin` onto our Debian image, which is a library (or package) containing the binary files used by GRUB (Grand Unified Bootloader). Our bootloader will be based on GRUB and the files we are downloading are needed to configure our bootloader properly. Bootloaders need GRUB if they use the BIOS (Basic Input Output system). The bootloader's job is to find and boot our kernel.

    RUN apt-get install -y grub-common

Downloads `grub-common` onto our Debian image. Same deal as the previous command but is fetching the common files needed by GRUB.

    VOLUME /root/env

You know how I said before that the Debian image is isolated from the rest of our system so it can't mess with anything important? Well, if we are working on something really cool and for whatever reason we accientally delete the Debian container, how do we get our work back. Answer is that you can't, it's gone forever. The `VOLUME` command in Docker creates a directory on our computer that sticks around even if we delete the Debian container. So it's a good idea to work in this directory so none of the outputs we create (e.g., iso files) are deleted with our Debian container. Yay!

    WORKDIR /root/env

Sets the working directory for any future commands. All commands will be executed from this directory within the container. If the directory does not exist, it will be created.