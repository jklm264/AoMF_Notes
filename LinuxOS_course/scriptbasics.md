# Linux Scripting Basics

"when you get the command prompt when you log in, that's a bash shell."

Script ideas:

- backup files on a schedule
- Run AV
- updates
- check for created, modified, or deleted files

Binary programs are faster than scripts.

Bash script := a list of commads to execute



`set -o vi/emacs` # will set default file editor.

Shebang (#!) tells machine what shell this should run.

```bash
'<text>' # no variable expansion
"<text>" # variable expansion
`<text>` # allows results to be assigned to a variable
(()) # how to do math
[[]] # something to be evaluated
() # How to get cmd return code
Source=“/var/log /etc/root /boot /opt /home” # list

if [ -z "$1"]; then
  echo "There aren't any args"
else
  echo "There are args being passed!"
fi

[ -d Documents ] && echo "found docs" # If documents found, then echo 'found docs'
[ -d Documents ] || echo "no docs" # If documents NOT found, then echo 'no docs'
```



**$Find**

```bash
find /etc -mtime -1 -print # Will find all modified within the past 24 hours
FILES_OLD=`find /etc/custom -type f -mtime+30 -print ` # will store the files that are older than 30 days
```



## Linux Regular Expressions (REGEX)

AIDE at beginning of line ***** 6/13/2013 ::`^AIDE*2013$`

Look for IP address:: `^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$`

`[]` match any set for a single char (unless followed by `()`)



## Custom Scripts

```bash
# This script updates the rkhunterinstallation and scans for rootkits
(
yum -y install rkhunter
/usr/local/bin/rkhunter --cronjob --report-warnings-only
) | /bin/mail -s "Daily rkhunter from $HOSTNAME" root
```



Logrotate is a tool to automatically rotate logs.

- like DNS request logs or DHCP bindings



#### Service and System Monitoring

Nagios or AIDE or custom script:

```bash
# Checks if splunk is up
if ps ax | grep splunkd> /dev/null
then
	echo "Splunk is running"
else
  echo "Splunk is down!"
fi
```



```bash
# Checks if drive is almost full
if (df -h | grep 9[89]%) then # Any values that are 98 or 99%
	df -h | grep 9.% | mail -s "Drive almost full on $HOSTNAME" serveruser@address.com
fi
```



#### **Cron**

`$crontab -e # to edit`

- will show in `/var/log/cron` and sometimes in `/var/log/messages`
- Can only see your own crontabs. HOWEVER, you could run `$ps` to check for cronjobs; will have to run *ps* at the right time though.

Syntax: `minute hour day month weekday program`

- Ex: `01 10 * * 0 checkrootkit.sh` # Will run at 10:01 on sundays



