# umg: Universal Maillog Grepper

A tool that understands mail logs, so you can grep for *messages* rather than 
just lines out of a log.

So far, it only understands postfix and qmail. Example logs and/or patches 
welcome!

Currently, this is working but largely untested, if this turns out useful it's 
quite likely to change quite rapidly as I use it :)

## Installation

So far, where installed, this is just checked out into 

    /root/universal-maillog-grepper/

So use it there :) Its only non-core dependency is JSON.

## Use

In general, you'll cat a logfile in on STDIN, use the options to set filters
and output and then pass the output onto something else.

To count see who's sending all that mail to @domain.com:

   cat /var/log/maillog | umg --pattern to:@domain.com --fields from | sort | uniq -c

for example.
	
`--pattern` (or `-p`) may be specified several times, and is a field name and a 
simple Perl regex separated by a colon. By default a message is deemed a match 
if any of the patterns match, but `--and` may be set to require *all* of them to.


The output is generally in columns of plain text, and the fields are specified 
using --fields (or `-f`) which expects a comma-separated list:

    cat /var/log/maillog | umg -p from:@domain.com  -f to,smtp_status | sort | uniq -c


Since this is generally intended as a data source for other things, the `--json`
option causes a JSON document to be output instead, which you can pipe into `jq` 
if you want to do some more complex stuff with the data:

https://stedolan.github.io/jq/


## Data Structure

Internally, a hash is created representing each message, some of the keys used
are universal (in that both supported MTAs log them) others are specific to one
or the other. The 'id' field is based on the MTA's own internal ID, but no 
guarantee is made that it will be usefully-so.

Common:
* `from`:                Envelope 'from' address
* `remote_hostname`:     DNS name of host making inbound connection, if it's provided in the log
* `remote_host`:         IP address of host making inbound connection
* `remote_or_local`:     Whether the recipient is local or remote
* `size`:                Size of the message, in bytes
* `smtp_status`:         SMTP mail-sending status
* `smtp_status_message`: Message returned by the remote host on mail-sending
* `timestamp_start`:     Timestamp from the first log-line mentioning the message
* `timestamp_end`:       Timestamp from any 'delivered' or 'finished' log-line
* `to`:                  Recipient address of the emai

Qmail-only:
* `delivery_id`:         Qmail's delivery ID (prone to re-use)
* `message_id`:          Qmail's message ID (prone to re-use)
* `timestamp_delivery`:  Timestamp from the 'delivered' log-line.
* `uid`:                 UID of the proces submitting the message.

Postfix-only:
* `auth_username`:       SASL username (Postfix/SASL only)
* `queue_id`:            Postfix's queue ID
* `relay`:               Postfix's relay name
* `subject`:             Where logged, parsed out of cleanup's warnings

additionally, in the JSON, the `log lines` key contains an array of the lines from
the input that were deemed related to the message. These are stripped out of non-
JSON output, and displayed by themselves when --log-lines is used.


