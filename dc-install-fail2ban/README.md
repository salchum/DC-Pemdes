# Guide to Install and Manage fail2ban in Ubuntu Server 24.04

## a. Install fail2ban
1. Update repository and install ``fail2ban``
```
sudo apt update
sudo apt install fail2ban -y
```
2. Enabling ``fail2ban``
```
sudo systemctl enable --now fail2ban
```
3. Check whether ``fail2ban`` is running or not
```
sudo systemctl status fail2ban
```
4. Create file ``jail.local`` in ``/etc/fail2ban/`` by copying the default fail2ban configuration
```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
5. Modify ``jail.local`` 
```
sudo nano /etc/fail2ban/jail.local
```
6. Change some parameters

|Parameter| Description | Value |
|----------|--|--|
|ignoreip| Whitelist IP for private or public IP |127.0.0.1/8 192.168.1.0/24 10.10.1.1/24 10.10.2.1/24 150.150.0.1/22 24.24.1.1/20 |
|bantime|how long an IP address will be banned for, negative number means permanent ban|60m|
| findtime | defines the timeframe within which the failed login attempts must occur to trigger a ban | 10m |
| maxretry |  the number of failed login attempts allowed from a single IP address within the findtime timeframe before a ban is imposed | 5 |
| port |  specifies the port number that fail2ban should monitor for the service you're protecting | 22,2200,2201 |
7. Restart ``fail2ban`` to apply changes
```
sudo systemctl restart fail2ban
```

## b. Monitoring fail2ban
1. Ensure ``fail2ban`` is running
```
sudo fail2ban-client ping
```
> it should reply, "**Server replied: pong**"
2. Check status of specific rules, i.e. ``sshd``
```
sudo fail2ban-client status sshd
```
3. Check status of specific rules and parameter
```
sudo fail2ban-client get sshd bantime
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd actions
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd ignoreip
```

## c. Ban and Unban IP Address
1. Ban Specific IP Address
```
sudo fail2ban-client set sshd banip IP-ADDRESS
```
2. Unban Specific IP Address
```
Sudo fail2ban-client set sshd unbanip IP-ADDRESS
```
3. Check list banned IP Address 
```
sudo fail2ban-client status sshd
```