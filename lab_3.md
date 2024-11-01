<br />
<div align="center">
  <h3 align="center">Iptables Lab 3</h3>
  <p align="center">Interact with Brute Force Attack using iptables<br>
</div>


| Steps | Host A (192.168.1.10) | Host B (192.168.1.20) |
| --- | --- | --- |
|1| 						 | 1- We create a smaller pipe for new SSH sessions.<br>This rule will block an IP if it attempts more than 3 connections per minute to SSH:<br>`-A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set`<br>`-A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP`<br>Notice that the state is set to NEW. This means only new connections not established ones are impacted. Established connections are the result of a successful SSH authentication, so users who authenticate properly will not be blocked.<br><br>2- Allow incoming connection from 192.168.1.10 on port 22 :<br> `-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT` <br><br>3- Block any other incoming connection: <br> `-A INPUT -m state --state NEW -m tcp -p tcp -j REJECT`|
|2| run `sh attck_bf.sh` | |
|3||Let see what’s being done, we want to log these drops. We can do so by setting up a log rule and then using these rules instead:<br>`-A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set`<br>`-A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j LOG --log-prefix "SSH_Log_Monitor" --log-level 4`<br>|
|4| `ssh -vvv 192.168.1.20` | Verify Logging: <br>`tail -f /var/log/iptables.log ` |

<br>
<br>

### Simple brute force script : attck_bf.sh<br>
1 - Creat the following simple brute force attack script attck_bf.sh: 

      
          #/bin/sh
          IP=$1
          PORT=$2
          TIMEOUT=$3
          SLEEP=$4
          MAX_ATTEMPS=$5
        
          echo "Testing Brute Force connection to $IP on port $PORT..."
          
          for nbr in `seq $MAX_ATTEMPS`
          do
          	echo -e "\nAttemp number : [ $nbr ]"
              nc -z -v -w$TIMEOUT $IP $PORT 2>/dev/null
          	# Check the exit status to determine if the connection was successful
          	if [ $? -eq 0 ]
          	then
          		echo "\_ Connection to $IP on port $PORT successful"
          	else
          		echo "\_ Connection to $IP on port $PORT failed."
          	fi
          	sleep $SLEEP
          done
2- Make the script executable: `chmod +x attck_bf.sh`<br>

3- Run brute force scripts with followinf parameter : <br>
Syntax : `sh attck_bf.sh ip port timeout sleep max_attemps`<br>
Example : `sh attck_bf.sh 192.168.1.20 22 3 2 5`<br><br>
`ip` : This is the IP address to which the script will attempt to connect. (192.168.1.20)<br>
`port ` : This is the port number that the script will use for the connection attempt. (22)<br>
`timeout ` : This specifies the maximum time, in seconds, that each connection attempt will wait before timing out. (3s)<br>
`sleep ` : This defines the number of seconds the script will wait between each connection attempt. (2s)<br>
`max_attemps ` : indicating the maximum number of connection attempts. (5)<br>

This will print whether the connection to the specified IP and port is successful or not.<br>

      
