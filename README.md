#!/bin/sh
: > output.txt
: > mail.html
subject="`date +"%Y%m%d %H:%M"` memory check-critical"
echo "From: kookai_924@yahoo.com" >> mail.html
echo "To: email@mine.com" >> mail.html
echo "Subject: $subject" >> mail.html
echo "Mime-Version: 1.0" >> mail.html
echo "Content-Type: text/html; charset=ISO-8859-1" >> mail.html
echo "Content-Transfer-Encoding: 7bit" >> mail.html
echo "<html><style type=text/css>
body {font-family:Segoe UI; padding: 3px; font-size: 14px;} TD {font-family:Segoe UI; padding: 3px; font-size: 12px; text-align: center;} TH{font-family:Segoe UI; padding: 3px; font-weight: bold; font-size: 12px;} </style>
<TABLE BORDER=1>

<TR>
<TH rowspan='1' bgcolor='black'><font color=white>User</TH>
<TH colspan='1' bgcolor='black'><font color=white>PID</TH>
<TH colspan='1' bgcolor='black'><font color=white>%CPU</TH>
<TH colspan='1' bgcolor='black'><font color=white>%MEM</TH>
<TH colspan='1' bgcolor='black'><font color=white>VSZ</TH>
<TH colspan='1' bgcolor='black'><font color=white>TTY</TH>
<TH colspan='1' bgcolor='black'><font color=white>STAT</TH>
<TH colspan='1' bgcolor='black'><font color=white>START TIME</TH>
</TR>" >> mail.html
if [ $# -ne 6 ] ; then
        echo ""
        echo "[Error] Incomplete Parameter.. "
        echo ""
        echo "NOTE: Parameters should be:
              -c : Critical Threshold
              -w : Warning Threshold
              -e : Email address"
        echo "(i.e. ./TOTAL_MEMORY.sh-w 40 -c 80 -e kookai_924@yahoo.com )"
        echo "***Critical Threshold should be GREATER than Warning Threshold"
        echo ""
        echo ""
exit 0;

fi
percent1=`free | grep Mem | awk '{print 100 - $7/$2 * 100.0}'`
percent=`printf "%.0f" $percent1`

case $1 in
        "-c") critical=$2
                if  [ "$2" -eq "0" ] 2>/dev/null ; then
                echo ""
                elif [ "$2" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-w") warning=$2
                if  [ "$2" -eq "0" ] 2>/dev/null ; then
                echo ""
                elif [ "$2" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-e") email=$2;;
        "*") echo "[Error] Invalid parameter.." ; exit 0;;
esac

case $3 in
        "-c") critical=$4
                if  [ "$4" -eq "0" ] 2>/dev/null; then
                echo ""
                elif [ "$4" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-w") warning=$4
                if  [ "$4" -eq "0" ] 2>/dev/null ; then
                echo ""
                elif [ "$4" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-e") email=$4;;
        "*") echo "[Error] Invalid parameter.." ; exit 0 ;;
esac

case $5 in
        "-c") critical=$6
                if  [ "$6" -eq "0" ] 2>/dev/null ; then
                echo ""
                elif [ "$6" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-w") warning=$6
                if  [ "$6" -eq "0" ] 2>/dev/null ; then
                echo ""
                elif [ "$6" -ne "0" ] 2>/dev/null ; then
                echo ""
                else
                echo "[Error] Invalid integer.." ; exit 0
                fi;;
        "-e") email=$6;;
        "*") echo "[Error] Invalid parameter.." ;exit 0 ;;
esac

if [ $warning -gt $critical ] ; then
        echo "[Error] Critical Threshold should be greater than Warning Threshold."
        echo ""
        exit 0
fi


if [ $percent -ge $critical ] ; then
#if [ 50 -ge 40 ] ; then

        echo "Used Memory is greater than or equal to Critical Threshold"

        ps aux --sort -rss | head -11 | awk '{print $1,$2,$3,$4,$5,$6,$7,$8,$9,$10}' > output.txt
        sed -n -e '2,11p' output.txt > final.txt

        cat final.txt | awk -F" " '{print $1}' > user.txt
        cat final.txt | awk -F" " '{print $2}' > pid.txt
        cat final.txt | awk -F" " '{print $3}' > cpu.txt
        cat final.txt | awk -F" " '{print $4}' > mem.txt
        cat final.txt | awk -F" " '{print $5}' > vsz.txt
        cat final.txt | awk -F" " '{print $6}' > tty.txt
        cat final.txt | awk -F" " '{print $7}' > stat.txt
        cat final.txt | awk -F" " '{print $8}' > starttime.txt

        paste user.txt pid.txt cpu.txt mem.txt vsz.txt tty.txt stat.txt starttime.txt | awk '{print "<td>" $1 "</td><td>" $2 "</td><td>" $3 "</td><td>" $4 "</td><td>" $5 "</td><td>" $6 "</td><td>" $7 "</td><td>" $8 "</td><tr>"}' >> mail.html

        echo "</TABLE><br><br>
</body></html>" >> mail.html

#       subject="`date +"%Y%m%d %H:%M"` memory check-critical"
#       recipient="ktuser2@no-reply.com"
#       mail -s "$subject" $recepient < mail.html

        cat mail.html | sendmail -t

elif [ $percent -ge $warning ] && [ $percent -lt $critical ] ; then

        echo "Used Memory is greater than or equal to Warning Threshold but less than the Critical Threshold"

else

        echo "Used Memory is less than Warning Threshold"
        echo ""
        echo ""

fi
