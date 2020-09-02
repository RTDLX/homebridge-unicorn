# Unicorn Homebridge Installation Guide
___
## What you need for this project:

### Hardware: (using non affiliate links)
* [Raspberry Pi Zero W](https://www.adafruit.com/product/3400)
* SD-Card for the Raspberry Pi
* [Pimoroni Unicorn pHAT](https://shop.pimoroni.com/products/unicorn-phat)
* Soldering Iron for soldering the male headers to connect the Pi with the pHAT.

### Software:
* [balenaEtcher](https://www.balena.io/etcher/) ( flashing the OS)
* PuTTY (SSH and telnet client for communicating with the Pi)

___
## Step 1: Setting up Homebridge
At this point you have two possibilites for the Raspberry Pi OS: 
* You can either choose to install the standard [Raspbian OS](https://github.com/homebridge/homebridge/wiki/Install-Homebridge-on-Raspbian) and make all the necessary Homebridge-installations yourself or 
* you can make your life easy and choose the premade [official Homebridge Raspberry Pi Image](https://github.com/homebridge/homebridge-raspbian-image/wiki/Getting-Started)

In this case we'll go for the premade Homebridge image according to the detailled installation guide [here](https://github.com/homebridge/homebridge-raspbian-image/wiki/Getting-Started).

Nevertheless here is a quick installation guide:

### 1.1 Download

You can download the most recent Homebridge Raspberry Pi image [here](https://github.com/homebridge/homebridge-raspbian-image/releases).

### 1.2 Flash to SD card

Using balenaEtcher you can choose the image (Raspbery Pi image) and the drive (SD-card) you want to flash it onto. 
Click 'Flash!'.

![](https://user-images.githubusercontent.com/3979615/74733445-789cac00-52a0-11ea-9167-05b42d6383ad.gif)

### 1.3 Connect to Network

After successfully flashing the image to the SD-Card you can put the SD-Card into the Pi and power it on using a Micro-USB cable and follow the following steps:
1. After powering on the Pi wait 1-2 minutes
2. Use any device with WiFi (i.e. Smartphone, Tablet, Laptop) and scan for new WiFi networks. 
3. Connect with the WiFi 'Homebridge WiFi Setup'
4. A captive portal opens where you can enter your local WiFi network. (This is the network your Pi is using and in which you can controll the Lamp)

If you enter your WiFi credentials incorrectly (Step 4) the WiFi will reappear and you can try again.

![](https://user-images.githubusercontent.com/3979615/75397237-7e525b80-594a-11ea-9be0-4f064b6a4178.png)

### 1.4 Manage Homebridge

One of the advantages of the premade Official Raspberry Pi Image we chose is the Web UI.

If the Raspberry Pi successfully connected to your local WiFi the Web UI should be accessible via
* `http://homebridge.local` (only if you're using macOS or a mobile device) or
* `http://<ip adress of your pi>`

**Default user name is `admin` with password `admin`.**

If `http://homebridge.local` doesn't work you need to find out your Pi's IP-Adress for accessing the web page. There are several possibilites:
1. Login to your routerand find check the connected devices. The device name should be 'homebridge'.
2. Use for example an iPhone to access `http://<ip adress of your pi>`. Log in with username `admin` and password `admin`. The IP adress is under System Information.
3. Download the [Fing](https://www.fing.com/) app for your mobile device which scans your network and the connected devices.

![](https://user-images.githubusercontent.com/3979615/71886653-b16d3f80-3190-11ea-9ff8-49dc4ae4fff0.png)

### 1.5 Connect to HomeKit

After logging into your Homebridge Web UI you can connect the homebridge to your apple device.
1. Start your Home app  on your Apple device.
2. Click the (+) Button on the top right.
3. Click 'Add Accessory' and scan the QR Code on the top left of the Web UI. (If this doesn't work you can enter the code which is shown beneath the QR Code)

## Step 2: Setting up the Unicorn pHAT

___

In this step we need to install all the required libraries and software for the Unicorn pHAT. 

### 2.1 Connecting with your Raspberry Pi

Open up PuTTY and enter the Pi's IP-Adress in the field 'Host Name (or IP adress)', click 'Open'.
A terminal pops up and asks for the login credentials.

**Default user name is `pi` with password `raspberry`**

### 2.2 Update your Raspberry Pi

The first thing you should do is update the software to ensure that everything runs flawlessly.
Enter the following commands:

    sudo apt-get update
    sudo apt-get -y upgrade

> Hint: You can paste the commands in PuTTY by just using your mouse right-button.

This process can take several minutes.

### 2.3 Update Node.js

Enter the Homebridge Raspbian Configuration tool:

    sudo hb-config
    
Select `Upgrade Node.js`. 
This will ensure that your Raspberry Pi is running the latest LTS version of Node.js.

![](https://user-images.githubusercontent.com/3979615/74602581-4b17fd00-50fe-11ea-8684-c03eaa42d27d.png)

### 2.4 System Configuration

You can enter the configuration setup with
   
    sudo raspi-config
    
In this configuration Tool you can change things like the pi's password, network settings, etc. if you want.
Recommended changes:
* Under `4 Localisation Options` change `Locale` and `Time Zone` accordingly.
* Under `7 Adavanced Options` click `A1 Expand Filesystem`. This ensures that all of the SD card storage is available.     
* Reboot your Pi

    
### 2.5 System Users and Directories

Add the system users 'homebridge' and 'unicorn':

    sudo useradd --system homebridge
    sudo useradd --system unicorn

Append groups for unicorn:

    sudo usermod -aG sudo unicorn
    
Create their directories:

    sudo mkdir /var/homebridge
    sudo mkdir /var/unicorn
    
Change owners:

    sudo chown homebridge:homebridge /var/homebridge
    sudo chown unicorn:unicorn /var/unicorn
    
Allow to alter the home folder:

    sudo usermod -d /var/unicorn unicorn
    sudo usermod -d /var/homebridge homebridge
    
### 2.6 Install unicorn-homebridge-integration

Change directory to `/var/unicorn`:

    cd /var/unicorn

Install the integration software via Git:

    sudo su unicorn -c "git clone https://github.com/mkoistinen/unicorn-homebridge-integration.git unicorn"

Change to the just created directory:

    cd /var/unicorn/unicorn

Copy the files `homebridge` and `unicorn` from the `init.d` directory to `/etc/init.d` to the right place:

    sudo su root -c "cp ./init.d/homebridge /etc/init.d/"
    sudo su root -c "cp ./init.d/unicorn /etc/init.d/"
    
Register the daemons:

    sudo update-rc.d homebridge defaults
    sudo update-rc.d unicorn defaults
    
Reload the daemons:

    sudo systemctl daemon-reload

    
### 2.7 Install Homebridge Node Requirements

Install using:

    sudo npm install -g --unsafe-perm homebridge
    sudo npm install -g --unsafe-perm homebridge-better-http-rgb
    
Copy the contents of `config.json` file from the `homebridge` directory into `/var/homebridge`:

    sudo cp /var/unicorn/unicorn/homebridge/config.json /var/homebridge/
    
### 2.8 Install the Unicorn Service

Change directory to `/var/unicorn`:

    cd /var/unicorn
    
Install `virtualenv`:

    sudo -H pip install virtualenv
    
Create virtual environment:

    sudo su unicorn -c "virtualenv venv"
    
Install requirements:

    sudo su unicorn -c "source venv/bin/activate && pip install -e unicorn/."

>When trying to execute the command from above the following error message appears in the terminal: `ERROR: Package 'mock' requires a different Python: 2.7.16 not in '>=3.6'`
    
Restart your Pi.
    
## 3. Final Steps

> coming soon...

## 4. Sources

https://github.com/homebridge/homebridge-raspbian-image/wiki/Getting-Started

https://github.com/mkoistinen/unicorn-homebridge-integration

https://github.com/mkoistinen/unicorn-homebridge-integration/blob/master/pi_setup.md










