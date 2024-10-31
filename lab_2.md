
<br />
<div align="center">
  <h3 align="center">Iptables Lab 2</h3>
  <p align="center">Interact with SSH packet using iptables and tcpdump<br>
</div>


| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) |
| --- | --- | --- |
|1| 						 | Allow incoming connection from 192.168.1.10 on port 22 :<br> `-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT` <br>Block any other incoming connection: <br> `-A INPUT -m state --state NEW -m tcp -p tcp -j REJECT`
|2| `ssh -vvv 192.168.1.20` | `tcpdump -Snn -i any host 192.168.1.10` |
|3||Block incoming SSH connection from 192.168.1.10 : <br> `-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j REJECT`|
|4|Open two teminal : <br> Terminal 1 : `tcpdump -Snn -i any port 22 and host 192.168.1.20` <br> Terminal 2 : `ssh -vvv 192.168.1.20`|`tcpdump -Snn -i any port 22 and host 192.168.1.10`|
|5| 						 | configure iptables Log on ICMP packets <br>****Enable Logging in Iptables****<br>Add this expression in the end of iptables rule `-j LOG --log-level 4`<br><br>***Configure Syslog***<br>By default, iptables logs are sent to the kernel’s message buffer. To view these logs, you need to configure syslog to read the message buffer and write the logs to a file. <br>This can be done by editing the syslog configuration file, typically located at `/etc/syslog.conf` or `/etc/rsyslog.conf`<br>You will need to add the following line to the syslog configuration file to enable iptables logging:<br>`kern.warning /var/log/iptables.log`<br>***restart services : iptables and rsyslog***<br>`service iptables restart`<br>`service rsyslog restart`<br>***Add the following line to iptables config***<br>`-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j LOG --log-prefix "SSH_Log_Monitor" --log-level 4` |
|6| 						 | change the rules order : <br>`-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j REJECT` <br>`-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT` |
|7| `ssh -vvv 192.168.1.20` | `tcpdump -Snn -i any host 192.168.1.10` |
|8| 						 | change the rules order, remove this rule : <br>`-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j REJECT`<br> and restrict for any other incoming connection to the server, by adding this rule: <br> `-A INPUT -m state --state NEW -m tcp -p tcp -j REJECT`<br><br>Iptables config should be like this : <br> `-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j LOG --log-prefix "SSH_Log_Monitor" --log-level 4`<br>`-A INPUT -m state --state NEW -m tcp -p tcp -j REJECT`|
|9| `ssh -vvv 192.168.1.20` | Verify Logging: <br>`tail -f /var/log/iptables.log ` |
