# Install EMQX Open Source on macOS

This page guides you on installing and starting the EMQX Open Source edition on macOS with a zip file.

Supported versions:

- macOS 14
- macOS 13

The instructions below will take macOS 13 as an example to illustrate how to download the latest version of EMQX. If you want to install a different version or in a different system, visit the [EMQX Deployment page](https://www.emqx.com/en/try?product=broker). 

## Install EMQX with Homebrew

[Homebrew](https://brew.sh/) is a free and open-source software package management system that simplifies software installation on macOS.

1. If you don't already have Homebrew installed on your Mac, you can install it by running the following command in the Terminal:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. Install EMQX:

   ```bash
   brew install emqx
   ```

## Install EMQX from Zip Package

1. Go to the [official site for EMQX](https://www.emqx.com/en/try?product=broker).

2. Select `@CE_VERSION@` for **Version** and `macOS 13` for **OS**, and click the **Download** button.

3. On the Downloads and Install page, select `zip` as the **Install Method** and select the proper **CPU Architecture** that matches your system. Click **Download Now**.

   You can also follow the command instructions on the page.

## Start and Stop EMQX

You can start EMQX in daemon mode, foreground mode, or interactive mode. Note that only one instance of EMQX can be running at any time with default configuration.

If you install EMQX with Homebrew, use `emqx` command as specified below. If you install EMQX from a zip package, use `bin/emqx` instead (assuming you are in the directory where you extract emqx files).

   ```bash
   # start as daemon
   emqx start

   # start in foreground
   emqx foreground

   # start in interactive mode, with Erlang shell
   emqx console
   ```

After a successful start, EMQX will output this message (if it is started in the foreground or interactive mode):

```bash
EMQX @CE_VERSION@ is running now!
```

You may also see some warning messages which are intended for operators of the production environment and can be ignored if EMQX is used in the local environment for tests, experiments, or client development:

```bash
ERROR: DB Backend is RLOG, but an incompatible OTP version has been detected. Falling back to using Mnesia DB backend.
WARNING: ulimit -n is 256; 1024 is the recommended minimum.
WARNING: Default (insecure) Erlang cookie is in use.
WARNING: Configure node.cookie in /opt/homebrew/Cellar/emqx/@CE_VERSION@/etc/emqx.conf or override from environment variable EMQX_NODE__COOKIE
WARNING: NOTE: Use the same cookie for all nodes in the cluster.
```

You can check the status of EMQX with this command:

```bash
emqx ctl status
```

Start your web browser and enter `http://localhost:18083/` (`localhost` can be substituted with your IP address) in the address bar to access the  [EMQX Dashboard](../dashboard/introduction.md), from where you can connect to your clients or check the running status.

The default user name and password are `admin` and `public`. You will be prompted to change the default password once logged in.

To stop EMQX:

* Use `emqx stop` or `bin/emqx stop` if it is started in daemon mode.
* Press Ctrl+C if it is started in foreground mode.
* Press Ctrl+C twice if it is started in interactive mode.
