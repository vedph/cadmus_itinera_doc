# Quick Start

To have a quick glance at the Itinera project system using mock data, you can run it on your own machine following these directions:

1. save the [docker-compose.yml file](https://github.com/vedph/cadmus_itinera_app/blob/master/docker-compose.yml) somewhere in your machine.  **Note** that you should just download the page you open, as this downloads the GitHub HTML page including the script, rather than the raw script itself. An easy way to download the code is clicking `Raw` at the top-right of the page. When code only gets displayed, just save it somewhere with name `docker-compose.yml`.

2. from a command prompt, enter the directory where you saved the `docker-compose.yml` file, and type the command `docker-compose up` (use `sudo` for Linux). This fires the backend and frontend services, and the first time it creates and seeds mock databases; so this will take one minute or two, depending on the performance of your machine. You will see a number of log information being dumped on the screen, and a total of 100 items being seeded into the newly created database. You should wait until the dump stops with no more messages. Just ensure that there are no error messages. Please notice that to allow other services startup, the API waits for 25 seconds before continuing; so, depending on the speed of your host system, it might happen that the other services are already done and the API is still waiting, and in this interval of time you won't see any message dumped; it will resume as soon as the API continues.

3. if everything went OK, open your browser at `localhost:4200`. To login:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of the [API project](https://github.com/vedph/cadmus_itinera_api). Please notice that in production they are always replaced with true credentials, set in server environment variables.
