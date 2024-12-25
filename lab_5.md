<br />
<div align="center">
  <h3 align="center">Iptables Lab 5</h3>
  <p align="center">Create Advanced Reverse Proxy with Port obfuscation<br>
</div>

## Lab Objectives : <br>
-  Hide *Host A* to be visible from *Host C*<br>
-  Hide *Host C* to be visible from *Host A*<br>
-  Obfuscate source server *Host A*<br>
-  Obfuscate destination server *Host C*<br>
-  Obfuscate source port *2202*<br>
-  Obfuscate destination port *22*<br>
-  The connection from <b>Host A</b> to <b>Host B</b> via port <b>2202</b> will be redirected to <b>Host C</b> to port <b>22</b>

| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) | Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1|Enable IP Forwarding : <br>`echo 1 > /proc/sys/net/ipv4/ip_forward`|||
|2||Enable IP Forwarding : <br>`echo 1 > /proc/sys/net/ipv4/ip_forward`||
|3|Test before applying *POSTROUTING* rule:<br><br>On the first console : <br>`tcpdump -i any -Snn dst port 2022`<br><br>On the second console : <br>`telnet 192.168.1.20 2202`|||
|4|Change the source port of connections from port:<b>2202</b> to port:<b>1102</b>:<br>`iptables -t nat -A POSTROUTING -p tcp -m state --state NEW,ESTABLISHED,RELATED -m tcp --dport 2202 -j SNAT --to :1102` | | |
|5|Retest after applying *POSTROUTING* rule:<br><br>On the first console : <br>`tcpdump -i any -Snn dst port 2022`<br><br>On the second console : <br>`telnet 192.168.1.20 2202`|||
|6||Change the outgoing connections packet from ip:192.168.1.10 port:1102 to ip:192.168.1.20 port:22 : <br><br>`iptables -t nat -A POSTROUTING -p tcp -m state --state NEW,ESTABLISHED,RELATED -m tcp --sport 1102 -s 192.168.1.10 --dport 22 -j SNAT --to-source 192.168.1.20`||
|7||Forward incomming connection from ip:192.168.1.10 port:2202 to ip:192.168.1.30 port:22 : <br><br>`-A PREROUTING -d 192.168.1.20 -p tcp -m tcp --dport 2202 -j DNAT --to-destination 192.168.1.30:22`||
|8| | Allow incomming connection from ip:192.168.1.10 port:1102 <br>`iptables -A RH-Firewall-1-INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 1102 -s 192.168.1.10 -j ACCEPT` ||
|9| | Allow incomming connection from ip:192.168.1.10 port:22 <br>`iptables -A RH-Firewall-1-INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT` ||
|9| | Allow incomming connection from ip:192.168.1.10 port:2202 <br>`iptables -A RH-Firewall-1-INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 2202 -s 192.168.1.10 -j ACCEPT` ||
|9| | |Allow incomming connection from ip:192.168.1.20 port:22 <br>`-A RH-Firewall-1-INPUT -m state --state NEW,ESTABLISHED,RELATED -m tcp -p tcp --dport 22 -s 192.168.1.20 -j ACCEPT` |
|5|Test routing connection:<br><br>On the first console : <br>`tcpdump -i any -Snn host 192.168.1.20 or host 192.168.1.30`<br>|`tcpdump -i any -Snn host 192.168.1.10 or host 192.168.1.30`|`tcpdump -i any -Snn host 192.168.1.10 or host 192.168.1.20`|
