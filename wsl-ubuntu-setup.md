# 🚀 WSL Ubuntu Setup Guide (Windows)

This guide helps you install and configure Ubuntu 22.04 using Windows
Subsystem for Linux (WSL).

------------------------------------------------------------------------

## 📌 Step 1 --- Open PowerShell as Administrator

1.  Press `Win + S`
2.  Search for **PowerShell**
3.  Right-click on it
4.  Select **Run as Administrator**

------------------------------------------------------------------------

## 📌 Step 2 --- Install Ubuntu 22.04

Run the following command:

``` powershell
wsl --install -d Ubuntu-22.04
```

Wait for the installation to complete (5--10 minutes).

------------------------------------------------------------------------

## 📌 Step 3 --- Restart Your System

Restart your laptop after installation.

------------------------------------------------------------------------

## 📌 Step 4 --- Create Linux User

After reboot, create:

-   Username: (your choice)
-   Password: (your choice)

------------------------------------------------------------------------

## 📌 Step 5 --- Verify Ubuntu Installation

``` bash
cat /etc/os-release
```

Expected output: Ubuntu 22.04

------------------------------------------------------------------------

## 📌 Step 6 --- Update System & Install Tools

``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl wget -y
```

------------------------------------------------------------------------

## ✅ Environment Ready

You can now start working with DevOps tools and Linux.
