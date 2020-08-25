# Module 1 Worksheet
## ORCA Login & Getting Started with the Terminal
### *Axel Hauduc*
### 11 September 2020
Welcome to the first MICB-405 lab! Today, we're going to get started with the essential skills and tools you'll be using for the rest of the course.

## Accessing Bash on Linux or an equivalent
In order to use the server your computer will either need a terminal/command-line interface or an emulator. These will be necessary for using the software tools throughout this course, so don’t think you can get away with not familiarizing yourself with one of these! 

If you are running a Unix-derived operating system like macOS, or a Linux distribution like Ubuntu, simply run the "Terminal" app.

For the those with Windows machines, there are several options available.

If you are running Windows 10, I would highly recommend the Ubuntu [Windows Subsystem for Linux](https://www.microsoft.com/en-ca/p/ubuntu/9nblggh4msv6?activetab=pivot:overviewtab). You will need to follow the instructions on the store page for installation. While it takes slightly longer to install than other solutions, it will allow you to have full Bash/Linux-style functionality on your machine. You can (carefully) practice Bash commands on files in your system just like you were running a Unix-derived system!

If you have a version of Windows earlier than Windows 10, or the Windows Subsystem for Linux simply doesn't work for you, I would recommend installing [Git Bash](https://gitforwindows.org/). With Git Bash you can ssh into the server and do all the work needed for the course, but has more limited functionality on your local machine.

## Logging in

At this point you should be ready to [log in](https://media2.giphy.com/media/LcfBYS8BKhCvK/giphy.gif?cid=ecf05e4747b1d69a24ea3b94dd23c9634105af0c7416ebb9&rid=giphy.gif). On whichever terminal you are using, you should be able to use the following command, replacing username with your actual username:

```bash
ssh username@orca1.bcgsc.ca
```

Enter your password when prompted. DON’T FREAK OUT WHEN CHARACTERS DON’T APPEAR - this is a security feature.

Let’s break this command down: ssh is the command and stands for “secure-shell”. It allows users to log on to remote (opposite of ‘local’, which is your laptop in this instance) servers. All following text is the argument. There may be many arguments and each of these would be separated by spaces. username is mostly obvious, but crucially this positions your shell in the ‘home’ directory of username on the server’s system with the correct permissions. If everyone were to log on as root (“Administrator” in Windows-speak) this would be bad. orca-wg.bcgsc.ca (everything after the '@') is known as the hostname or domain name and is the name of the device on the network it is connected to. Super-nerds sometimes replace this with the IP address.
