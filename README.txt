
maillog2json - converts various forms of maillog into a standard json structure.
Currently, 'various' means 'qmail and postfix'.

USAGE:

   maillog2json [options] <logfile>
   cat /var/log/maillog | maillog2json [options]
   cat qmail_log | tai64nfraction | mailllog2json [options]

OPTIONS:

 --log-type <type>, -t <type>
         override logfile type detection, 'type' may be one of 'qmail' or 'postfix'

 --pretty
         produce pretty-printed JSON rather than a single line

 --date-format <format>; --num-date-elements <num>
         Specify a date format for figuring out the timestamp of log lines 
	 containing human-friendly dates. <format> should be a pattern suitable
	 for passing to strptime(), and <num> should be the number of leading 
         whitespace-separated elements of the log line to take as the date 
	 string. Defaults:
	   date-format: %b %d %T
	   num-date-elements: 3

Either cat the file on stdin or supply it as an argument. For Qmail logfiles using the
tai64n timestamp, pipe them through tai64nfraction first.

Soon there will be something with which to read the JSON, until then use jq:

https://stedolan.github.io/jq/

STRUCTURE:

A JSON array of hashes is returned, keys of these hashes are:

Common:
* from:                Envelope 'from' address
* remote_hostname:     DNS name of host making inbound connection
* remote_host:         IP address of host making inbound connection
* remote_or_local:     Whether the recipient is local or remote
* size:                Size of the message, in bytes
* smtp_status:         SMTP mail-sending status
* smtp_status_message: Message returned by the remote host on mail-sending
* timestamp_start:     Timestamp from the first log-line mentioning the message
* timestamp_end:       Timestamp from any 'delivered' or 'finished' log-line
* to:                  Recipient address of the emai

Qmail-only:
* delivery_id:         Qmail's delivery ID (prone to re-use)
* message_id:          Qmail's message ID (prone to re-use)
* timestamp_delivery:  Timestamp from the 'delivered' log-line. Qmail only.
* uid:                 UID of the proces submitting the message; qmail-only

Postfix-only:
* auth_username:       SASL username (Postfix/SASL only)
* message_id           Postfix's message ID
* relay:               Postfix's relay name
* subject:             Where logged, parsed out of cleanup's warnings


