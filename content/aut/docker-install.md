---
title: Hands on With The Archives Unleashed Toolkit
date: 2018-09-25T11:51:41-04:00
---

This quick guide will walk through how to install Docker for Mac users. Much of this is also applicable to Windows users, although the screenshots are from a Mac.

## Resources
* [Docker Community Edition for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
* [Docker Community Edition for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)

## Install Docker 
To install Docker for a Mac OS, use this [link](https://store.docker.com/editions/community/docker-ce-desktop-mac). You can also use these instructions to install Docker on a Windows 10 Professional installation, by using [this link](https://store.docker.com/editions/community/docker-ce-desktop-windows).

You'll need to create a free Docker ID to get started. It's a fairly quick signup process:

![Creating an account](/images/docker-create-account.png)

Once you have an account, log in to Docker and select the Docker Community for Mac (or Windows) program.

![Download](/images/docker-download2.png)

From the downloads folder, double click to start installation (.dmg file)

![Download](/images/docker-download3.png)

Once file has been verified you can drag and drop into your applications folder, then launch application.

![install](/images/docker-install.png)

<i>Please note</i>, depending on your security preferences a notice may show up warning you, you are about to open an application from the web. Select open to continue. 

The Docker icon should appear in your menu bar along the top of your screen. When you click on the icon, there should be a green dot indicating docker is running

![install-6](/images/docker-running.png)

Now that the Docker application is installed you can log in with your Docker ID that you created earlier.

## Run Docker and Install AUT

Open your terminal and try out some Docker commands. On a Mac, you can find a terminal window by going to your Applications Folder, selecting "Utilities," and then opening up "Terminal."

On Windows, you should open up a "command-line terminal like PowerShell." You can find some instructions for that [here](https://programminghistorian.org/en/lessons/intro-to-bash).

Try running the following two commands:

| Command        | Purpose           |
| ------------- |:-------------:|
| `docker version` | to check that you have the latest release installed |
| `docker run hello-world` | to verify that Docker is pulling images and running as expected |

Now you're ready to install AUT. Type the following command:

```
docker run --rm -it archivesunleashed/docker-aut:0.17.0
```

This will take a while to download and install. Eventually, you'll hopefully see:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.3.2
      /_/

Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_151)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

You're now ready for the [workshop](/aut/lesson).
