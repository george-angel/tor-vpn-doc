# How to VPN without VPN
This aims to solve the problem of connecting to internal networks without exclusively locking your connection to that one network as well as not having to 2FA with your phone, dongle, rain stick or a magic charm. You shoould be able to open your laptop anytime, anywhere and have secure encrypted access to the network.

## Prerequesites
Things you will need on your journey:

  - Machine inside the network running *nix with:
      * [openssh](http://www.openssh.com/)
      * [tor](https://www.torproject.org/)
  - Locally you will need: 
      * [openssh](http://www.openssh.com/)
      * [tor](https://www.torproject.org/)
      * [ncat](https://nmap.org/ncat/)
      * [foxyproxy](http://getfoxyproxy.org/) (optional)
      * [autossh](http://www.harding.motd.ca/autossh/) (optional)

## Setup - Serverside
Enable (systemd) sshd and tor, or rc.d or whatever gets the daemons running on your machine. Configure tor run a hidden service forwarding to 22:

Create a user, leave their public key in `~/.ssh/authorized_keys`

Edit: `vim /etc/tor/torrc`

GOTO: `############### This section is just for location-hidden services ###`

Uncomment: `HiddenServicePort 22 127.0.0.1:22`

Uncomment: `HiddenServiceDir /var/lib/tor/hidden_service/`

Restart tor

Grab the hostname of the now running hidden service: `cat /var/lib/tor/hidden_service/hostname`

Done!

## Setup - Clientside
This varies with your needs. I have the following in `~/.xinitrc`:

`autossh -f -M 20000 -Nn -D 2424 -l {{user}} {{onion-address-of-your-server}}.onion`

Which will run a permanent tunnel on local 2424 to reconnect whenever disconnected (sleep, no wifi)

### SSH to internal network boxes
In your `~/.ssh/config`:

```sh
### Tor tunnel - route any ssh connection to .onion through tor
Host *.onion
ProxyCommand ncat --proxy-type socks5 --proxy 127.0.0.1:9050 %h %p

### When you ssh to any box inside the network, route that through ssh tunnel
Host *.{{network-domain}}.com
ProxyCommand ncat --proxy-type socks5 --proxy 127.0.0.1:2424 %h %p
```

### Browsing internal services on the browser 

Create a proxy in *foxyproxy* pointing to `localhost:2424` tick `sock proxy v5`.

In patters configure patters for your network:

```
^https?://.*\.{{domain}}\.com.*$
^https?:\/\/{{domain}}[a-z]{3,5}[0-9]{2,5}.*$
...
```
Select *use proxies based on their pre-defined patterns and priorities*

