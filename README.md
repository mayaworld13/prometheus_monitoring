# Prometheus Setup

This guide will help you create a dedicated Linux user for Prometheus and set up the Prometheus monitoring system on your server.

## Introduction

Prometheus is an open-source systems monitoring and alerting toolkit. To enhance security and simplify administration, we will create a dedicated system user for Prometheus.

## Creating a System User

To create a system user for Prometheus, run the following command:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```


This command does the following:

–system – Will create a system account.
–no-create-home – We don’t need a home directory for Prometheus or any other system accounts in our case.
–shell /bin/false – It prevents logging in as a Prometheus user.
Prometheus – Will create a Prometheus user and a group with the same name.

By creating a separate user for Prometheus, you enhance security and make it easier to manage and track resources related to Prometheus.

## Installing Prometheus

Let’s check the latest version of Prometheus from the [download](https://prometheus.io/download/) page.

You can use the curl or wget command to download Prometheus.
```bash
wget  https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
```
Then, we need to extract all Prometheus files from the archive.
```bash
tar -xvf prometheus-2.53.0.linux-amd64.tar.gz
```
![image](https://github.com/mayaworld13/prometheus_monitoring/assets/127987256/dd699334-96f8-4f94-8f4a-28ed5838d49f)

Usually, you would have a disk mounted to the data directory. For this tutorial, I will simply create a /data directory. Also, you need a folder for Prometheus configuration files.

```bash
sudo mkdir -p /data /etc/prometheus
```

Now, let’s change the directory to Prometheus and move some files and move the Prometheus binary and a promtool to the /usr/local/bin/. promtool is used to check configuration files and Prometheus rules.

```bash
cd prometheus-2.53.0.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
```

Optionally, we can move console libraries to the Prometheus configuration directory. Console templates allow for the creation of arbitrary consoles using the Go templating language. You don’t need to worry about it if you’re just getting started.

```bash
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

To avoid permission issues, you need to set the correct ownership for the /etc/prometheus/ and data directory.

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```
You can delete the archive and a Prometheus folder when you are done.

```bash
cd
rm -rf prometheus-2.47.1.linux-amd64.tar.gz
```

Verify that you can execute the Prometheus binary by running the following command:

```bash
prometheus --version
```

We’re going to use Systemd, which is a system and service manager for Linux operating systems. For that, we need to create a Systemd unit configuration file.

```bash
sudo vim /etc/systemd/system/prometheus.service
```
`prometheus.service`

```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle
[Install]
WantedBy=multi-user.target

```
# Prometheus and Systemd Configuration

## Systemd Options

- **Restart**: Configures when the service should be restarted. For example, the service can be restarted if it crashes, is stopped, or times out.

- **RestartSec**: Specifies the time to wait before restarting a service. If the service stops, Systemd will wait this amount of time before attempting to start it again.

- **User and Group**: Defines the Linux user and group under which the Prometheus process should run. This is important for managing permissions and security.

## Prometheus Options

- **–config.file=/etc/prometheus/prometheus.yml**: Path to the main Prometheus configuration file. It tells Prometheus where to find its settings.

- **–storage.tsdb.path=/data**: Location where Prometheus should store its data on disk.

- **–web.listen-address=0.0.0.0:9090**: Configures Prometheus to listen for web requests on all network interfaces at port 9090. If using a proxy like nginx to redirect requests to Prometheus, you might set this to only listen on localhost.

- **–web.enable-lifecycle**: Enables management of Prometheus, such as reloading its configuration, without needing to restart the service.

then automatically start the Prometheus after reboot, run enable.

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```


Suppose you encounter any issues with Prometheus or are unable to start it. The easiest way to find the problem is to use the journalctl command and search for errors.

```bash
journalctl -u prometheus -f --no-pager
```
Now we can try to access it via the browser. I’m going to be using the IP address of the Ubuntu server. You need to append port 9090 to the IP.
```bash
<public-ip:9090>
```


![image](https://github.com/mayaworld13/prometheus_monitoring/assets/127987256/aca5267e-d3cc-413c-ad56-f0b76b5d7c35)

