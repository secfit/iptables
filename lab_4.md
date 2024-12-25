<br />
<div align="center">
  <h3 align="center">Iptables Lab 4</h3>
  <p align="center">Create DMZ using NAT table<br>
</div>


| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) | Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1||Enable IP Forwarding : <br>`echo 1 > /proc/sys/net/ipv4/ip_forward`||
|2| | Allow incoming connection from 192.168.1.10 on port 80: <br>`iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -s 192.168.1.10 -j ACCEPT`||
|3| | Apply PREROUTING and POSTROUTING rules in NAT table:<br>`iptables -t nat -A PREROUTING -p tcp -d 192.168.1.20 --dport 80 -j DNAT --to-destination 192.168.1.30:80`<br><br>`iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.30 --dport 80 -j SNAT --to-source 192.168.1.20`| |
|2| | | Allow incoming connection from 192.168.1.20 on port 80 <br>`iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -s 192.168.1.20 -j ACCEPT` |
|3| Testing : <br>`telnet 192.168.1.20 80` | | |
|4| | verify the connection : <br>`tcpdump -i any -Snn port 80`  | verify the connection : <br>`tcpdump -i any -Snn port 80` |
      
