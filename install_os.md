# Installation

1. Select the language og the OS

2. Select the Keyboard layout

3. Check the Type of installation. Check either "Ubuntu Server" or "Ubuntu Server (minimized)". Also, check "Search for third-party drivers".

## Network configuration

Follow the instructions below to configure your network and when you are done, select "Done" to continue.

### Connect to Wifi

This is optional if you have an Ethernet connection.

To connect to your Wifi, follow these steps:
1. Click on a Wifi interface with "TYPE" "wlan".
2. Click on "Edit Wifi".
3. Click on "Choose a visible network", find and select your Wifi name, enter your Wifi password, and click on "Save".

### Configure a static IP address

Select an interface that you want to configure. This interface can have any "TYPE" value. Note that the "TYPE" value "eth" is an Ethernet interface and "wlan" is a Wifi interface.

Select "Edit IPv4"

Select "Manual" for the field "IPv4 Method"

In "Subnet", enter your IP range of your network. It would be "192.168.0.0/24" in most home networks.

In "Address", enter your server static IP address like "192.168.0.200".

In "Gateway", enter your router's IP address. You can run `ipconfig` and find it in the output field "Default Gateway". In most case, it would be "192.168.0.1".

In "Name servers", enter "8.8.8.8, 8.8.4.4"

You can leave "Search domains" blank.

Select "Save".

For more info about how to configure a static IPv4 address watch this [video](https://youtu.be/2Btkx9toufg?si=5bEYdWzPwK_JYH1_&t=342)


## Proxy Configuration

You can leave blank for now and continue by selecting "Done".

## Ubuntu archive mirror configuration

Leave the default one and select done to continue.

## Storage Configuration




