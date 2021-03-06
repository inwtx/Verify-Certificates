# Verify-Certificates
Check certificate validity of remailers and pingers.

```
#!/bin/bash
#
# VerifyCerts.sh  1.2
#
# This script checks the certificate dates for chosen remailer and
# and pinger and signals if a certificate has expired.  It is mainly
# useful to the keepers of the pinger's allpinger sites.  It can be
# used to find remailers that don't have tls connection capabilities.
# Place the remailer and pinger domain name in the SYSarray array as
# needed.  If the script hangs on a displayed name, cancel it and
# remove that name from the SYSarray array.
#
# Execute as: /path/to/VerifyCerts.sh
# You can email results to an email address using a cronjob:
# 0 6 * * * /path/to/VerifyCerts.sh your@email.address    # run at 6 AM and send to attached email
#

filePath=${0%/*}  # current file path

emailaddr=$1

SYSarray=(
anonymitaet-im-inter.net
beaufusil.com
borked.net
cloaked.pw
deuxpi.ca
dizum.com
eu.org
fruiti.org
hoi-polloi.org
kraut.space
lambton.org
mixharbor.xyz
mixmin.net
mixport.xyz
paranoici.org
privacy.at
redjohn.net
sec3.net
slugish.net
tnetconsulting.net
uni-boeblingen.de
zip2.in
)

cat /dev/null > $filePath/varVC.txt
cat /dev/null > $filePath/varVC.txt2
cat /dev/null > $filePath/varVC.txt3


for i in "${SYSarray[@]}"; do
    echo "$i"
    echo "$i" >> $filePath/varVC.txt2
    varVC1=$(echo | openssl s_client -servername $i -connect $i:443 2>/dev/null | openssl x509 -noout -dates 2>/dev/null)

    if [[ ! $varVC1 == "" ]]; then               # put in cert expdt
       echo "$varVC1" > $filePath/varVC.txt
       sed -i '/notBefore=/d' $filePath/varVC.txt
       sed -i 's/notAfter=//g' $filePath/varVC.txt
       varVC2=$(< $filePath/varVC.txt)
       echo "Expdt: $varVC2" >> $filePath/varVC.txt2

       varVC3=$(awk '{ print $1"  "$2" "$4}' <<< $varVC2)  # Feb  8 12:19:20 2021 GMT to Feb  8 2021
       varVC4=$(date -d "$varVC3" +%s)
#t5
       echo "Days remaining: $(( ($varVC4 - $(date -u +%s))/(60*60*24) ))" >> $filePath/varVC.txt2
    else
       echo "Certificate not found." >> $filePath/varVC.txt2  # no cert for this remailer
    fi

    echo "" >> $filePath/varVC.txt2
done

sed -i '/CERTIFICATE/d' $filePath/varVC.txt2

echo ""
echo "--- Results ---"
echo ""


sendemail=0

while read line1; do
      if [[ $line1 =~ "Days remaining:" ]] && [[ $line1 =~ "-" ]]; then
         echo "$line1          <===EXPIRED!" >> $filePath/varVC.txt3
         sendemail=1
         else
         echo "$line1" >> $filePath/varVC.txt3
      fi
done< $filePath/varVC.txt2

if [[ $sendemail == "1" ]] && [[ ! $emailaddr == "" ]]; then              # send results in email if available
   mail -s "From VerifyCerts.sh" $emailaddr < $filePath/varVC.txt3
fi

cat $filePath/varVC.txt3

rm $filePath/varVC.txt
rm $filePath/varVC.txt2
rm $filePath/varVC.txt3

exit 0
```
