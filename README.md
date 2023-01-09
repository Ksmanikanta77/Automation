# Automation

sudo apt-get update

#checking apache server

server= dpkg --get-selections apache2 | awk '{print $2}'

if [[ ${server} == install ]]
then
       echo "Apache 2 is already installed"
else
        sudo apt install apache2 -y
fi

#checking running status

status=sudo systemctl status apache2 | grep running | awk '{print $3}' | tr -d '()'

if [[ ${status} == running ]]
then
        echo "Server is running"
else
        sudo service apache2 start
fi

#logfiles

cd /var/log/apache2/

name="srinivas"
timestamp=$(date '+%d%m%Y-%H%M%S')

tar -zcvf /tmp/${name}-httpd-logs-${timestamp}.tar *.log
s3_bucket="upgrad-srinivas-manikanta"
aws s3\
        cp /tmp/${name}-httpd-logs-${timestamp}.tar \
        s3://${s3_bucket}/${name}-httpd-logs-${timestamp}.tar


cd /var/www/html/
record="/var/www/html"

if [[ ! -f ${record}/inventory.html ]]; then
echo -e 'Log Type\tTime Created\tType\tSize' | sudo tee $record/inventory.html > /dev/null
fi

if [[ ! -f ${record}/inventory.html ]]; then
size=$(du -h /tmp/${name}-httpd-logs-${timestamp}.tar | awk '{print $1}')
echo -e "httpd-logs\t${timestamp}\ttar\t${size}" | sudo tee -a $record/inventory.html > /dev/null
fi


if ! crontab -l | grep -q "/root/Automation_Project/automation.sh"; then
  echo "* * * * * root /root/Automation_Project/automation.sh" | sudo tee -a /etc/cron.d/automation > /dev/null
fi
