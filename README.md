This will hopefully help anyone go from having a single box ELK setup to visualizing their AD DNS server requests.
 
#I include all the sources I have saved but a majority of the "work" I put into this was piecing together what multiple others shared. Feel free to share this. 

#On Windows AD DNS server. (mine happens to be 2012)
#All ELK components are version 5.4

Grab version 5.4 from https://www.elastic.co/downloads/beats/packetbeat
5.5 is out now but this was done with 5.4 and although I havent tested, their updates break config often


Unzip and move the folder to exactly C:\Program Files\packetbeat
Follow one of many guides to create an SSL cert on your ELK server and copy it to Windows
Install the certificate to "Local Machine" and browse to "Trusted Root Certificate Authorites"

#The way I'm doing it stores a copy of the foo.syslog.cer in C:\Program Files\packetbeat\foo_syslog.cer
#Do not use tabs in the packetbeat.yml file.  Spaces only or it will not work at all.
#The packetbeat.yml config file is set to capture and forward all packets on port 53 for DNS only for AD DNS servers although I think very similar if not identical configs work on other DNS servers.

Open powershell as admin and "set-executionpolicy unrestricted"
	CD to C:\Program Files\packetbeat\
	./install_whateverthenameofthescriptis
	Wait until the rest of these steps are complete to run "start-service packetbeat"
	

#On Logstash server
put the 05-packetbeat.conf file in /etc/logstash/conf.d/
chown logstash:root /etc/logstash/conf.d/

#Feel free to delete 
filter {
   if "packetbeat" in [tags] {
	dns {
		reverse => [ "client_ip" ]
		action => "replace"
		hit_cache_size => "100"
		hit_cache_ttl => "180"
	}
   }
}
#I think it requires setting a cache in /tmp in your /etc/logstash/logstash.yml file. That part of the config shows the name of the employees machine that made the request rather than the IP.
#The email alert at the bottom is untested but I thought I'd leave it around.  I'm going to work on cross referencing the DNS lookups with live threat feeds for shit like TOR exit nodes and malicious domains.  First for alerting and testing, hoping eventually to take action automatically.
#An amazing blog I found extremely helpful is here. Thank you for this.  It answered so many questions when I was a complete noob.
http://syspanda.com/index.php/2017/02/07/incorporating-virustotal-data-to-elasticsearch/

#Amazing idea using only powershell and export-json. I haven't gotten it to work 100% yet. https://hazzy.techanarchy.net/winadmin/windows/windows-powershell-elk-log-wash/

#Useful article for winrm reference if you want to take that further. https://geeksilver.wordpress.com/2016/01/15/create-eventlog-forwarding-and-winrm/

#Use this link for a little information on Windows Event log forwarding. 
http://edbaker.weebly.com/blog/windows-and-logstash-quick-n-dirty

#I'm honestly not sure how my packetbeat.template.json that returns when I run "curl localhost:9200/_template/packetbeat*?pretty" is different than the one in your packetbeat folder on Windows. I may have uploaded a custom one that I don't have a link for.  The mapping parts are very confusing to me.  If you understand better please hit me up and explain.
#Try the included packetbeat.template.json file and if it doesn't work try the one included already.
"curl -XPUT 'localhost:9200/_template/packetbeat-*' -d @packetbeat.template.json"

#Make sure its loaded in logstash
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/05-packetbeat.conf   #or just "systemctl restart logstash"
sudo fuser tcp/5169
	#look for logstash PID
"Start-service packetbeat" on Active Directory DNS server
Go to Kibana under the management tab and add a new index. packetbeat-*



#######################################Extra notes###############################################
#In an environment of 38 servers and 21 users this adds up to 1.5gb worth of dns logs per day.  Its 
amazing how little power an elasticsearch server needs to quickly search through this data.

#I usually run this command to stop previous months logs from consuming elasticsearch's resources.
	curl -XPOST localhost:9200/winlogbeat-2017.06*/_close
		#To turn the indices back on just do curl -XPOST localhost:9200/winlogbeat-2017.06*/_open


#If you have any trouble I recommend adding the output filter to output to stdout with the rubydebug codec 
#Check the c:\program files\packetbeat\logs\ directory to see whats happening on the windows side after 



Check the pictures for my simple visualization and saved search.


