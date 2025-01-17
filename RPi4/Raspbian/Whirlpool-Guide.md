# Welcome to the Whirlpool Installation Guide
This guide will allow you to set up the Whirlpool-client-cli, serving as a backend for continous mixing, even when you GUI is not running.

## Prepare your Pi
   1. First install openjdk8

```
sudo apt-get install openjdk-8-jre
```

   2. Create a whirlpool directory and pull the latest whirlpool-client-cli file

```
cd $HOME
mkdir whirlpool
cd whirlpool
wget https://github.com/Samourai-Wallet/whirlpool-runtimes/releases/download/cli-0.9.1/whirlpool-client-cli-0.9.1-run.jar
```

   3. Install Tor outside docker if you haven't already.
      - NOTE: it is recommended to create a new user for tor for security purposes

```
sudo adduser -m toruser
## Create password and remember it
sudo usermod -aG toruser
```

Raspbian requires us to update our apt sources list to properly install the most up-to-date Tor release

```
sudo nano /etc/apt/sources.list
--------------
###  Add the following:
deb https://deb.torproject.org/torproject.org stretch main
deb-src https://deb.torproject.org/torproject.org stretch main
```

**Save and exit with: Ctrl+X, y, Enter**

In order to verify the integrity of the Tor files, download and add the signing keys of the torproject using the network certificate management service (dirmngr)

```
sudo apt install dirmngr apt-transport-https
curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -
```

Now switch to toruser and install Tor

```
su toruser
sudo apt update
sudo apt install tor
```

Edit the torrc file

```
sudo nano /etc/tor/torrc
-------------
# uncomment:
ControlPort 9051
CookieAuthentication 1

# add:
CookieAuthFileGroupReadable 1
```

**Save and exit: Ctrl+X, y, Enter**

```
sudo systemctl restart tor
su pi ### Or whatever your main username is
```

   4. Install tmux to allow us to close out the window but keep whirlpool running

```
sudo apt-get install tmux
````

## Mobile Pairing Code

Open up your Samourai Mobile Wallet
Press 3 dots on top right -> Settings -> Transactions-> Pair to Whirlpool GUI. Copy the code to clipboard
Send the code to yourself, how you feel is most secure.

## Back to Pi

Make sure we are in the correct directory and initiate the whirlpool client

```
cd whirlpool
tmux new -s whirlpool
java -jar whirlpool-client-cli-0.9.1-run.jar --init --tor
```
   - This will ask for your pairing code, Paste it when prompted with Ctrl+Shift+V
   - When it displays the API key. Copy that key and send to your main computer however is most secure for you. 
      - You will need this to pair with GUI

The whirlpool client with then close and ask to run again.

This time we run a different command:

```
java -jar whirlpool-client-cli-0.9.1-run.jar --server=mainnet --tor --auto-mix --authenticate --mixs-target=0 --listen
### NOTE: If you want the mixing target to be a different number other than infinity change the mixs-target to desired number
```

This time it will ask for your Mobile wallet passpharase to authenticate the wallet
   - Enter it when prompted

It should load all your potential mixs. and be running.

Close out the tmux window

**Ctl+B , D**

**NOTE: To get back in later simply enter the following command:**

```
tmux a -t 'whirlpool'
```

## Pairing your with the Whirlpool GUI

Let's install the Whirlpool GUI on a your main computer for a beautiful interface.

Go to https://github.com/Samourai-Wallet/whirlpool-gui/releases and grab the appropriate file for your system. For this guide: I'll just use the App.Image

```
wget https://github.com/Samourai-Wallet/whirlpool-gui/releases/download/0.9.1/whirlpool-gui.0.9.1.AppImage
sudo chmod +x whirlpool-gui.0.9.1.AppImage
```

And now let's launch it:

```
./whirlpool-gui.0.9.1.AppImage
```

Ok so you'll be shown with two options: Standalone CLI or Remote CLI. Select remote CLI (our pi)

Next you'll need to enter your Pi's IP address: 192.168.X.XXX 

**NOTE: If you do not know your Pi's IP address, go into terminal and type:**

```
ip a
### if you are connected via ethernet, look for eth0 
### if you are connected via wifi, look for wlan0 
```

API Key you saved earlier, copy the code, click on the api box and hit Ctrl+V. 
_Leave the Port as is_

Now connect. May take a minute or so, but you'll see the wallet ask for your passphrase again. This is your Mobile wallet BIP39 
passphrase you entered in the CLI. 

Once it registers you'll be connected for self-sovereign continuous mixing!
