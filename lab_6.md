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

<br><br>
#### Case 1 : Random balancing
| Steps | Front _ Host A (192.168.1.10) | Back _ Host B (192.168.1.20) | Back _ Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1|for Back Host B :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.20:80`<br><br>`iptables -A POSTROUTING -p tcp -d 192.168.1.20 --dport 80 -j SNAT --to-source 192.168.1.10`|||
|2|for Back Host C :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to-destination 192.168.1.30:80`<br><br>`iptables -A POSTROUTING -p tcp -d 192.168.1.30 --dport 80 -j SNAT --to-source 192.168.1.10`|||
|3|Explanation : <br>With a probability of 0.5, the first rule will be executed 50% of the time and skipped 50% of the time.<br>You can calculte probability using this formula:<br><br>`p = 1/(n-i+1)`<br>where:<br>p = Probality<br> n = Number of nodes (in this example n = 2)<br> i = index, mean if the first node then i = 1, for second i = 2|||

<br><br>
#### Case 2 : Round Robin
| Steps | Front _ Host A (192.168.1.10) | Back _ Host B (192.168.1.20) | Back _ Host C (192.168.1.30) |
| --- | --- | --- | --- |
|1|for Back Host B :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.20:80`<br><br>`iptables -A POSTROUTING -p tcp -d 192.168.1.20 --dport 80 -j SNAT --to-source 192.168.1.10`|||
|2|for Back Host C :<br>`iptables -A PREROUTING -t nat -p tcp -d 192.168.1.10 --dport 80 -m statistic --mode nth --every 2 --packet 1 -j DNAT --to-destination 192.168.1.30:80`<br><br>`iptables -A POSTROUTING -p tcp -d 192.168.1.30 --dport 80 -j SNAT --to-source 192.168.1.10`|||



<b>Explanation</b> : <br>We are using *nth* algorithm, this algorithm implements a round robin algorithm.<br>This algorithm takes two different parameters: every `n` and packet `p`.<br><br>Matching the Rule:<br>The first packet matches `--packet 0`, so it’s redirected to Backend Host 2.<br>The second packet matches `--packet 1`, so it’s redirected to Backend Host 3.
<br><br>
<b>How `--every` Works:</b> <br>

The `--every` option determines the frequency of packets that match. The --packet option specifies the starting point in the sequence.<br>
*Rule 1:*<br>
    `--every 2`: Matches every 2nd packet.<br>
    `--packet 0`: Matches the first packet in every two.<br>
    Action: Redirects to 192.168.1.20:80.<br><br>

*Rule 2:*<br>
    `--every 2`: Matches every 2nd packet.<br>
    `--packet 1`: Matches the second packet in every two.<br>
    Action: Redirects to 192.168.1.30:80.<br><br>

*Expected Behavior:*<br>
    Requests alternate between the two backend servers:<br>
       - First packet → 192.168.1.20:80.<br>
       - Second packet → 192.168.1.30:80.<br>
       - Third packet → 192.168.1.20:80.<br>
       - Fourth packet → 192.168.1.30:80.<br>

| `--every 2` | `--packet` | Destination server |
| --- | --- | --- | 
|1st Packet |0|192.168.1.20|
|2nd Packet |1|192.168.1.30|
|3rd Packet |0|192.168.1.20|
|4th Packet |1|192.168.1.30|
|5th Packet |0|192.168.1.20|
|6th Packet |1|192.168.1.30|

<table style="height: 119px; width: 502px;">
<thead>
<tr style="height: 18px;">
<th style="height: 18px; width: 122.3px;">Phase</th>
<th style="height: 18px; width: 98.8833px;">`--every 2`</th>
<th style="height: 18px; width: 116.9px;">`--packet`</th>
<th style="height: 18px; width: 135.917px;">Destination Host</th>
</tr>
</thead>
<tbody>
<tr style="height: 18px;">
<td style="height: 36px; width: 122.3px;" rowspan="2">every 2 packets</td>
<td style="height: 18px; width: 98.8833px;">P_1</td>
<td style="height: 18px; width: 116.9px;">0</td>
<td style="height: 18px; width: 135.917px;">192.168.1.20</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 98.8833px;">P_2</td>
<td style="height: 18px; width: 116.9px;">1</td>
<td style="height: 18px; width: 135.917px;">192.168.1.30</td>
</tr>
<tr style="height: 18px;">
<td style="height: 29px; width: 122.3px;" rowspan="2">every 2 packets</td>
<td style="height: 18px; width: 98.8833px;">P_3</td>
<td style="height: 18px; width: 116.9px;">0</td>
<td style="height: 18px; width: 135.917px;">192.168.1.20</td>
</tr>
<tr style="height: 18px;">
<td style="height: 11px; width: 98.8833px;">P_4</td>
<td style="height: 11px; width: 116.9px;">1</td>
<td style="height: 11px; width: 135.917px;">192.168.1.30</td>
</tr>
<tr style="height: 18px;">
<td style="height: 36px; width: 122.3px;" rowspan="2">every 2 packets</td>
<td style="height: 18px; width: 98.8833px;">P_1</td>
<td style="height: 18px; width: 116.9px;">0</td>
<td style="height: 18px; width: 135.917px;">192.168.1.20</td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; width: 98.8833px;">P_2</td>
<td style="height: 18px; width: 116.9px;">1</td>
<td style="height: 18px; width: 135.917px;">192.168.1.30</td>
</tr>
</tbody>
</table>
