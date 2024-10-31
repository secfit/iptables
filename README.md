<br />
<div align="center">
  <h3 align="center">Iptables Lab</h3>
  <p align="center">Converts linux box into a packet filter<br>
</div>

This project aims to investigate the features of the Linux firewall, iptables, in a basic network environment. We will exclusively utilize Linux virtual machines to implement the network configuration in order to make it easier to deploy outside of the lab. We will just talk about VirtualBox exploitation in this guide.

### Lab_1
#### Interact with linux ping packet using iptables and tcpdump
| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) |
| --- | --- | --- |
|1| 						 | Restrict any incoming ICMP connection <br> `-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -j REJECT`
|2| `ping -c 4 192.168.1.20` | `tcpdump -icmp -n -i any host 192.168.1.10` |
|3| 						 | Allow incoming ICMP packets from **192.168.1.10** <br>`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -s 192.168.1.10 -d 192.168.1.20 -j ACCEPT` <br> and Restrict any other incoming ICMP connection <br> `-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -j REJECT` |
|4| `ping -c 4 192.168.1.20` | `tcpdump -icmp -n -i any host 192.168.1.10` |
|5| 						 | change the rules order : <br>`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -j REJECT` <br>`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -s 192.168.1.10 -d 192.168.1.20 -j ACCEPT` |
|6| `ping -c 4 192.168.1.20` | `tcpdump -icmp -n -i any host 192.168.1.10` |
|7| 						 | change the rules order : <br>`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -s 192.168.1.10 -d 192.168.1.20 -j ACCEPT` <br>`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -j REJECT` |
|8| 						 | configure iptables Log on ICMP packets <br>
****Enable Logging in Iptables****<br>
Add this expression in the end of iptables rule `-j LOG --log-level 4`<br>
<br>
***Configure Syslog***<br>
By default, iptables logs are sent to the kernel’s message buffer. To view these logs, you need to configure syslog to read the message buffer and write the logs to a file. <br>
This can be done by editing the syslog configuration file, typically located at `/etc/syslog.conf` or `/etc/rsyslog.conf`<br>
You will need to add the following line to the syslog configuration file to enable iptables logging:<br>
`kern.warning /var/log/iptables.log`<br>
***restart services : iptables and rsyslog***<br>
`service iptables restart`<br>
`service rsyslog restart`<br>
***Add the following line to iptables config***<br>
`-A INPUT -m state --state NEW,ESTABLISHED,RELATED -p icmp --icmp-type any -s 192.168.1.10 -d 192.168.1.20 -j LOG --log-prefix "ICMP_Log_Monitor" --log-level 4` |
|9| `ping -c 4 192.168.1.20` | Verify Logging: <br>`tail -f /var/log/iptables.log ` |
