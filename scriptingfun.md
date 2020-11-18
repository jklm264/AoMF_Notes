# Linux Scripting Fun

**Per Hal**

`$awk '/search_term/  {print $1}' <file>`



`sort | uniq -c | sort -n` # 1) sort by alphabetic 2) uniqify them 3) count each unique value

- unique only works on dups that are contiguous 



`sed 's/- -.*" "//;' ` # match between two patterns (with `.*`)

`grep php access.log | sed 's/^.*] //; s/[0-9][0-9][0-9] ".*$//' | wc -l` # greps for all mentions of php and cuts line from right after the time (`*]`) to (some of) byte count to end. We just want the URI.



` grep php access.log | sed  's/[0-9][0-9][0-9] ".*$//' | awk '{print $1}' | sort | uniq | xargs -n1 echo "banyou"
banyou 149.129.50.37` # 1) Finds all mention of php 2) cuts line off after URI 3) prints just the ip 4) sorts 5) formats for banyou







"GET /wp-login.php HTTP/1.1" 404 77888 "http://jmussman.net/wp-login.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0"