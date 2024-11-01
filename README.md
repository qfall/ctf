# qFALL-ctf

This part of the [qFALL project](https://qfall.github.io/) is based on the CTF platform [CTFd](https://github.com/CTFd/CTFd).
We integrate a [plugin](https://github.com/MelonShooter/ctfd-individual-flags-plugin) to individualize the challenges, provide cryptographic challenges and an easy setup of TLS.

This repository is currently being developed by the project group [qFALL - quantum resistant fast lattice library](https://cs.uni-paderborn.de/cuk/lehre/veranstaltungen/ws-2022-23/project-group-qfall) in the winter term 2022 and summer term 2023 by the Codes and Cryptography research group in Paderborn.

This project is dedicated to the educational part of qFALL.
We provide this CTF platform and its challenges to enable lecturers among the world to provide even more engaging lectures on cryptography (specifically focussing on lattice-based cryptography).

Please refer to [our website](https://qfall.github.io/) as a central information point.
To use our library for your project, please refer to [our tutorial](https://qfall.github.io/book/index.html).
It provides a step-by-step guide to install the required libraries and gives further insights into the usage of our crates.

## Disclaimer

Currently, we are in the development phase and interfaces might change.
Feel free to check out the current progress, but be aware, that the content will
change in the upcoming weeks and months. An official release will most likely be published in the second half of 2024.

## What does qFALL-ctf offer?
This repository contains a configuration of the open-source platform [CTFd](https://github.com/CTFd/) that any lecturer can setup within 10 minutes to provide engaging CTF challenges for their students.
Below, you can find step-by-step instructions to deploy our instance using [Docker](https://docker.com) based on any Debian-based Linux distribution.
If you want to test the deployment on your localhost first, please ignore and remove any certificate-related commands and lines in the configuration files.

### Setup

Requirements
- A server with ...
    - at least two threads (four are recommended)
    - at least 1 GB of RAM (2 GB are recommended)
    - a static public IP address
    - an operating system based on Debian
- A registered domain that points to this IP address
- Opened HTTP (80), HTTPS (443), SSH (22) ports
    - Preferably close all other ports - if there is no firewall in place, you can use [ufw](https://wiki.ubuntu.com/UncomplicatedFirewall) to setup one
- An SSH connection to your server

#### Setup CTF Instance
Once you are connected to your server via SSH, we make sure that all required packages are installed including [snap](https://snapcraft.io/), [git](https://git-scm.com/), and [Docker](https://docker.com).
```bash
sudo apt-get update
sudo apt-get install snapd git
sudo snap install snapd docker
```

Then, clone this git repository and enter the directory using the following command.
```bash
git clone https://github.com/qfall/ctf.git && cd ./ctf/
```

#### Customize Configuration
Now, we need to make some changes to the following files: `docker-compose.yml` and `conf/nginx/http.conf` using your favorite editor.
- In the `docker-compose.yml` file, please change the `SECRET_KEY` value and adjust the number of `WORKERS` appropriately to your system setup.
- In the `conf/nginx/http.conf` file, adjust the number of `worker_processes` according to your setup and replace `<your_domain.org>` by your domain *everywhere*. Please make sure that you made all 5 substitutions of the domain!

Now, it's time to start the system.
```bash
docker compose up -d
```
Note that for some systems, `docker-compose` might need to be connected via a dash.
The `-d` tells the docker container to run detached from the console.
You can always join the feed using `docker compose attach`.
You can also `stop`, `restart`, `pause`, `unpause`, ... via `docker compose`.
If you ever want to remove it, call `docker compose down`.

#### Obtain your Certificate
Currently, your system is up and running, but there is no certificate available to establish secure connections.
To get a certificate using [Let'sEncrypt](https://letsencrypt.org/) and [Certbot](https://certbot.eff.org/), execute the following command (replacing `your_domain.org` by your domain).
```bash
docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ -d your_domain.org
```
Attach `--dry-run` to test this command in order to avoid hitting the maximum number of signing requests if something went wrong.
Enter your email and any other requested information to get a signed certificate.
After successfully obtaining a signed certificate, restart the [Nginx](https://nginx.org) server by executing the following line.
```bash
docker compose restart nginx
```
Connect to your server for the first time. You should be able to setup your new CTFd instance.

#### Setup Auto-Renewal of Certificate
Your certificate is valid for three months.
Renewing the certificate will be done by the following command if it is near to expiration.
```bash
docker compose run --rm certbot renew
```
Furthermore, you need to restart the Nginx server afterwards to load the new certificates.

As you probably don't want to run these commands every third month, we setup a 'cron job', which executes these commands daily.
As the certificate will only be renewed close to its expiration date, the commands will only restart Nginx if this happens.

Setup a `cron job` by executing the following line.
```bash
sudo crontab -e
```
Place the following at the end of the file and replace `directory_of_your_ctf_instance` by the path to your `ctf` directory.
```bash
30 3 * * * cd /path_to_ctf_directory/ && docker compose run --rm certbot renew --post-hook "docker compose restart nginx"
```
Now, the server will try to renew the certificate every day at 3:30 am and restart the `nginx` server if a new certificate was issued.
