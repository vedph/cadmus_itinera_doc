# Quick Start

To have a quick glance at the Itinera project system using mock data, you can run it on your own machine following the directions in this page.

## Prerequisites

You can use a Windows, Linux or MacOS machine. The only requirement is that you have Docker installed. You can refer to [this page](https://github.com/vedph/cadmus_doc/blob/master/deploy/docker-setup.md) about installing it.

Also, you should be able to open a terminal window in a folder:

- on Windows, open the folder you want in file explorer's `File` menu and pick `open Windows Powershell`.
- for MacOS users, open the folder and then from `Finder/Services` pick `New Terminal at Folder`. Should this not be displayed, see [this help page](https://www.howtogeek.com/210147/how-to-open-terminal-in-the-current-os-x-finder-location/).
- it's safe to assume that Linux users already know how to this.

## Starting the System

1. save the [docker-compose.yml file](https://github.com/vedph/cadmus_itinera_app/blob/master/docker-compose.yml) somewhere in your machine. **Note** that you should just download the page you open, as this downloads the GitHub HTML page including the script, rather than the raw script itself. An easy way to download the code is clicking `Raw` at the top-right of the page. When code only gets displayed, just save it somewhere with name `docker-compose.yml`.

2. from a command prompt, enter the directory where you saved the `docker-compose.yml` file, and type the command `docker-compose up` (**note**: use `sudo` for Linux or MacOS; usually this will prompt you for your computer's account password). This fires the backend and frontend services, and the first time it creates and seeds mock databases; so this will take one minute or two, depending on the performance of your machine. You will see a number of log information being dumped on the screen, and a total of 100 items being seeded into the newly created database. You should wait until the dump stops with no more messages. Just ensure that there are no error messages. Please notice that to allow other services startup, the API waits for a minute or less before continuing; so, depending on the speed of your host system, it might happen that the other services are already done and the API is still waiting, and in this interval of time you won't see any message dumped; it will resume as soon as the API continues.

3. if everything went OK, open your browser at `localhost:4200`. To login:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of the [API project](https://github.com/vedph/cadmus_itinera_api). Please notice that in production they are always replaced with true credentials, set in server environment variables. If no page opens when navigating to this address, this means that something went wrong in the previous passages, so be sure to check for error messages there.

## Maintenance

When you have finished playing with the system, just issue a `docker-compose down` command in the folder where you downloaded the composer script. This will clear everything, so that the next time you start the script again, it will restart fresh, rebuilding all the data.

If you just want to stop playing for a bit and then re-enter the system without resetting it, just break out of the terminal window you opened when launching it with `docker-compose up` (e.g. CTRL+C in Windows). The next time you launch the compose up command, you will find the same data you left.

Whenever a new version of the system is published, and you want to upgrade, you just have to `docker-compose down` your current version, download the `docker-compose.yml` file again, and restart.
