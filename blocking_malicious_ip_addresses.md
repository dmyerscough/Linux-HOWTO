Block Malicious IP Addresses
============================

A while back, I wrote a script that grabs the latest SPAM list from SPAMHAUS in order to block malicious IP addresses from connecting to the server.  The script shown below can be added to a cron job that runs on a daily basis. The script will maintain an IP chain “Spamhaus“, which will have the list of malicious IP addresses.

```bash
#!/bin/bash
#
# Author: Damian Myerscough
# Description: Downloads the latest known spam hosts and blocks their access
#
#
 
# Path to iptables
IPTABLES="/sbin/iptables";
 
# List of known spammers
SPAM="www.spamhaus.org/drop/drop.lasso";
SPAM_TABLE="Spamhaus";
SPAM_LIST="/tmp/spam.lst";
 
# Check to see if the chain Spamhaus exsits
$IPTABLES -L $SPAM_TABLE
 
# if the Spamhaus tables does not exsit create it
if [ $? -eq 0 ]; then
    # Flush the old rules
    $IPTABLES -F $SPAM_TABLE    
else
    # Create a new chain set
    $IPTABLES -N $SPAM_TABLE
fi;
 
# Get a copy of the spam list
wget -qc $SPAM -O $SPAM_LIST
 
# Iterate through all known spamming hosts
for IP in $( cat $SPAM_LIST | grep -iE "^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | awk -F\; '{ print $1 }' ); do
    # Add the IP address to the SPAM table
    $IPTABLES -A $SPAM_TABLE -p 0 -s $IP -j DROP
done
 
# Remove the spam list
unlink $SPAM_LIST
```
