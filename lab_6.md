<br />
<div align="center">
  <h3 align="center">Iptables Lab 6</h3>
  <p align="center">Create Http Load Balancing using Iptables<br>
</div>


## Lab Objectives : <br>
-  Build a sophisticated TCP router and load balancer on Port 80<br>
-  Route TCP packets to the correct destication<br>
-  Load balance connections across multiple servers<br>

### Prerequisite 
#### Securing Front Host 
| Steps | Front _ Host A (192.168.1.10) | Back _ Host B (192.168.1.20) | Back _ Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1|Enable IP Forwarding : <br>`echo 1 > /proc/sys/net/ipv4/ip_forward`<br><br>or by *sysctl*:<br>`sysctl net.ipv4.ip_forward=1`|||
|2|Drop all FORWARD traffic:<br>`iptables -t filter -P FORWARD DROP`|||
|3|Allow FORWARD traffic for Backend Hosts <br>`iptables -t filter -A FORWARD -d 192.168.1.20 -j ACCEPT`<br>`iptables -t filter -A FORWARD -d 192.168.1.30 -j ACCEPT` | | |
|4||Allow incomming connection from ip:192.168.1.10 port:80 <br>`iptables -A INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 80 -s 192.168.1.10 -j ACCEPT`||
|5|||Allow incomming connection from ip:192.168.1.10 port:80 <br>`iptables -A INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 80 -s 192.168.1.10 -j ACCEPT`|

#### Case 1 : Random balancing
| Steps | Front _ Host A (192.168.1.10) | Back _ Host B (192.168.1.20) | Back _ Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1|for Back Host B :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.20:80`<br>|||
|2|for Back Host C :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.30:80`<br>|||
|3|Explanation : <br>With a probability of 0.5, the first rule will be executed 50% of the time and skipped 50% of the time.<br>You can calculte probability using this formula:<br><br>`p = 1/(n-i+1)`<br>where:<br>p = Probality<br> n = Number of nodes (in this example n = 2)<br> i = index, mean if the first node then i = 1, for second i = 2|||
