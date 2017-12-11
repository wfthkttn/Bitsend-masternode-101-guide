# 「BitSend」Masternode 101 Guide



Guide designed for total linux/windows newbies.

We will use Ubuntu 16.04 Server hosted at Vultr and local machine (PC) with Win10 OS.

Other distributives than linux16.04 will require additional knowledge, we will not cover them.

Take guide seriously, especially parts with creating keys,  strong passwords which not used anywhere else and overall security.
You deal with money, don’t forget that.




**Sources:**
- [Bitsend GitHub](https://github.com/LIMXTEC/BitSend)
- [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- [Keepass](https://keepass.info/download.html)
- [Vultr hosting](https://www.Vultr.com)  ([referral](https://www.vultr.com/?ref=7146997))



**1. Server preparations.**

**1.1 Initialisation:**

Create vultr.com account, and i suggest you to add 2FA.

Deploy new server (+), choose region to host a server, ubuntu 16.04 with 1gb RAM($5/mo). 

Auto backups is up to you, i don’t use it, however it’s pretty cheap. Give server a name, deploy, and it will be ready after few minutes.

Press “Manage”, or  “...” -> “Server Details”.

Write down and save IP Address and password(will be changed), we will need it later. 

You can connect to the server directly from the browser, but we’d better go another way


**1.2. Securing:**

Run putty.exe, paste IP address and specify name “bitsend under “Saved sessions” and hit save, then “open”.

Accept unknown certificate -> yes.

Login as “root” -> “password”. (hint - paste from clipboard is shift+insert or RMB)

Change root’s password:       (learn to use keepass, it’s a lifesaver)

```
passwd
[NEW PASSWORD]
[NEW PASSWORD]
```

Add new user with sudo group:

```
adduser bitsend
[Another NEW PASSWORD]
[Another NEW PASSWORD]
```

skip others
```
usermod -aG sudo bitsend
```
This will add new user with name bitsend, reboot server to make sure passwords are correct.


**1.3. ACCESS SSH WITH KEY** (optional, but recommended)

Run PuTTY gen, check “RSA” type and generate the key. You can specify passphrase, then on login  it will be verified too.

save both - public and private keys on your PC (you can save it in keepass)

Open public key with text editor, login into server as bitsend and create folder for keys:
```
mkdir ~/.ssh
chmod 700 ~/.ssh
```
create file containing public key:
```
nano ~/.ssh/authorized_keys
add “ssh-rsa [copy public key from text editor and paste here]”
```
it will look like “ssh-rsa KEYRANDOMNESSjPAISJDOIAJSDOihadohAD==”, in one line

[ctrl+o, enter, ctrl+x to save&exit]
```
chmod 600 ~/.ssh/authorized_keys
exit
```
Now we need to add those keys to putty, run it, hit our saved session and press load.

Pick “data” from left menu, add “bitsend” under “auto-login”.

Find “auth” submenu under “SSH” directory in the left menu, click “browse..” and point it into private key.

Under first menu option - “session” specify new name for this connection. Like “bitsend@mn#” and hit save.

Now, when you open this session key and passphrase will be checked. It’s the time to disable default login+password authentication.
```
sudo nano /etc/ssh/sshd_config
[insert bitsend’s password]
```
Find PasswordAuthentication, uncomment it and change “yes” to “no”

Also uncomment strings and make it match following:
```
PermitRootLogin no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
```
restart SSH daemon:
```
sudo systemctl reload sshd
```
Now you can only login with rsa key, check if you are able to.

Gratz, now you can have simple userpassword to use with sudo, and strong passphrase to log-in with key.

It’s more secure than using common passwords, just make sure your private key is not leaked.


**1.4 Firewall configuration**

Straight to the commands:

```
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow 8886/tcp
sudo ufw default deny incoming 
sudo ufw default allow outgoing 
sudo ufw enable 
```

[y, Enter]

```
sudo reboot
```

Now login again and check if firewall works correctly:

```
sudo ufw status
```

should print ‘status: active’ and current rules.

Now you are ready to go with fully configured linux server. Weeee!


**2. Installing latest wallet on PC and gathering keys**

Download latest binaries for windows  https://github.com/LIMXTEC/BitSend/releases

Install it

Run the wallet(bitsend-qt.exe), after synchronization go to Help > Show Bitsend Conf file, add “masternode=0”

Open settings>options>wallet, check “Enable coin control features”

Open debug window under “help”, go to console

type “masternode genkey”, save generated key (MNPRIVKEY)

type “getaccountaddress mn1”, save the address (MN1 ADDRESS)

Encrypt your wallet (Settings -> Encrypt Wallet…) and save passphrase.

To check if encryption works fine, run wallet and type in console:

walletpassphrase [passphrase] 60

It should output “null”

Backup wallet.dat somewhere, attach to keepass, use flash card or load it to cloud, whatever.

Send 25,001 BSD from exchange to no label, main address(not mn1). Adresses listed under “file>receiving addresses”. 

You can create test-transaction by sending 1 bsd to wallet, usually it takes less than 15 minutes to fully confirm transaction but appears instantly.

After you receive bitsend on wallet, create transaction to address mn1 with exactly 25000 coins

Type in console:
```
masternode outputs
```
Save given TXID and INDEX.

open masternode.conf (Help>Show Masternode Conf File) and fill it. Syntax:
```
ALIAS ServerIP:8886 MNPRIVKEY TXID INDEX DONATIONADDRESS:AMOUNT
```
It should look like that:
```
mn1 192.168.0.1:8886 123qwerty321ytrewq 0 iQWERTY123z:100
```
Donationaddress is addres which will receive rewards, you can add bittrex address here, or split rewards

e.g. DONATIONADDRESS:50 will send half of reward to address. It is optional.

**3. Building masternode wallet on remote server.**

**3.1 Allocate swap**
```
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo mkswap /swapfile
sudo swapon /swapfile
sudo chmod 600 /swapfile
```
**3.2 Install requirements**
```
sudo apt-get update -y
sudo apt-get dist-upgrade -y
sudo apt-get install build-essential libtool autotools-dev autoconf automake pkg-config libssl-dev -y
sudo apt-get install libboost-all-dev git npm nodejs nodejs-legacy libminiupnpc-dev redis-server -y
sudo apt-get install software-properties-common -y
sudo apt-get install libevent-dev -y
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install libdb4.8-dev libdb4.8++-dev -y
```

**3.3 Clone and build BitSend daemon**
```
git clone https://github.com/LIMXTEC/BitSend
cd BitSend
./autogen.sh
./configure
make
cd src
strip bitsendd
sudo cp bitsendd /usr/local/bin
sudo cp bitsend-cli /usr/local/bin
sudo chmod 775 /usr/local/bin/bitsend*
```

**3.4 Install and configure masternode**
```
mkdir ~/.bitsend
nano ~/.bitsend/bitsend.conf
```
Modify and paste this into file:
```
rpcuser=[randomuser]
rpcpassword=[strong&random password]
rpcallowip=127.0.0.1
maxconnections=256
gen=0
listen=1
server=1
daemon=1
masternode=1
promode=1
masternodeprivkey=[MNPRIVKEY]
externalip=[ServerIP]:8886
```
[ctrl+o,Enter,ctrl+x]

**Use bootstrap file to load blockchain quicker. (without it will take several hours)**
```
wget https://www.mybitsend.com/bootstrap.tar.gz
tar -xvzf bootstrap.tar.gz
```
Now run bitsend, just type “bitsendd” and hit enter

You can enjoy watching blocks getting accepted
```
tail -f debug.log
```

**3.5 Bitsend as a service**
```
sudo mkdir /usr/lib/systemd/system
sudo nano /usr/lib/systemd/system/bitsendd.service
```
paste this:
```
[Unit]
Description=BitSend's distributed currency daemon
After=network.target

[Service]
User=bitsend
Group=bitsend
Type=forking
PIDFile=/home/bitsend/.bitsend/bitsendd.pid

ExecStart=/usr/local/bin/bitsendd -daemon -disablewallet -pid=/home/bitsend/.bitsend/bitsendd.pid \
          -conf=/home/bitsend/.bitsend/bitsend.conf -datadir=/home/bitsend/.bitsend/

ExecStop=-/usr/local/bin/bitsend-cli -conf=/home/bitsend/.bitsend/bitsend.conf \
         -datadir=/home/bitsend/.bitsend/ stop

Restart=always
PrivateTmp=true
TimeoutStopSec=60s
TimeoutStartSec=2s
StartLimitInterval=120s
StartLimitBurst=5

[Install]
WantedBy=multi-user.target
```
Save and enable it.
```
systemctl enable bitsendd.service
```
reboot your server and check if bitsendd running
```
sudo reboot
```
Reconnect and input “top”, bitsendd should be at the top somewhere.

clear bash logs, just in case:
```
cat /dev/null > ~/.bash_history && history -c && exit
```

**4. Activate masternode!**

Start local wallet (PC) and go to console.

Unlock wallet:
```
walletpassphrase [passphrase] 60
```
start masternode by alias:
```
masternode start-alias mn1
```
Output should be like that:
```
{
  "alias": "mn1",
  "result": "successful"
}
```
You can close your wallet, your masternode is running! Weeee!


Now you are free to drop your job and enjoy self-explorations for a lifetime!

Support BitSend and stay tuned.
