BIND Resetting SOA Serial Number
================================

The serial number defined in the SOA (Start of Authority) resource record is a 32 bit integer value (232 – 1) which is used by slave servers to determine if a zone has changed and requires a zone transfer. The serial number needs to be incremented in order for slave servers to know a change has occurred and initiate a zone transfer, the most common format for the serial number is YYYYMMDDVV (where VV is the edit version).

If you accidentally incremented the serial number to an undesired value, you can reset the serial number by adding 231 (2147483647) to the serial number, and then updating the serial number in the zone file and letting the changes propagate.

The following command can be used to query the slave server for the SOA resource record, which will show you the current serial number. If the serial number is the same as the master, propagation has occurred.

	$ dig @[SLAVE SERVER] [DOMAIN NAME] soa

Once the serial number has propagated, you can update the zone file with the desired serial number and let the new serial number propagate.
