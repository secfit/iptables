<br />
<div align="center">
  <h3 align="center">Iptables Lab 3</h3>
  <p align="center">Interact with Brute Force Attack using iptables<br>
</div>


| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) |
| --- | --- | --- |
|1| 						 | Allow incoming connection from 192.168.1.10 on port 22 :<br> `-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT` <br>we create a smaller pipe for new SSH sessions.<br>This rule will block an IP if it attempts more than 3 connections per minute to SSH:<br>`-A RH-Firewall-1-INPUT -p tcp --dport 80 -m state --state NEW -m recent --set`<br>`-A RH-Firewall-1-INPUT -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP`<br>Notice that the state is set to NEW. This means only new connections not established ones are impacted. Established connections are the result of a successful SSH authentication, so users who authenticate properly will not be blocked.<br>Block any other incoming connection: <br> `-A INPUT -m state --state NEW -m tcp -p tcp -j REJECT`|
|2| `` | Verify Logging: <br>`tail -f /var/log/iptables.log ` |
|3||Let see what’s being done, we want to log these drops. We can do so by setting up a log rule and then using these rules instead:<br>`-A RH-Firewall-1-INPUT -p tcp --dport 80 -m state --state NEW -m recent --set`<br>`-A RH-Firewall-1-INPUT -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP`<br>|
|4| `ssh -vvv 192.168.1.20` | Verify Logging: <br>`tail -f /var/log/iptables.log ` |
