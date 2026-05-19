Capture quick health snapshot.

## environment basics
**uname -a  or lsb_release -a**

<img width="1211" height="48" alt="image" src="https://github.com/user-attachments/assets/8142e3b2-e8b5-4491-b872-8016e52b892a" />

## cpu memory

**top**
<img width="1263" height="568" alt="image" src="https://github.com/user-attachments/assets/6324bdaa-8c35-4c4a-80f2-aec3af4c2599" />

**free -h**

<img width="937" height="87" alt="image" src="https://github.com/user-attachments/assets/ae18710c-4e16-4d57-8abe-034ca991b8c8" />

## Disk/IO

**df -h**

<img width="941" height="331" alt="image" src="https://github.com/user-attachments/assets/7c406ba3-c0d8-49e2-91d7-0b4eebd33476" />

**du -sh /var/log**

<img width="752" height="197" alt="image" src="https://github.com/user-attachments/assets/418a1198-07a9-4376-9d87-18f659725cb1" />

## Network

**ss -tulnp or netstat -tulnp**

<img width="1642" height="295" alt="image" src="https://github.com/user-attachments/assets/11167750-c225-4adb-956d-9b5545eb5394" />

## logs

**journalctl -n 50**

<img width="1822" height="265" alt="image" src="https://github.com/user-attachments/assets/623176c1-f4fa-4f53-b680-a349cf87c9f5" />

_________________________________________________________________________________________________________________________________________


# runbook on specific target service.(nginx)

## systemctl status nginx
**in output I observed that the service is inactive for the last 10 mins.**

## ps -aux | grep nginx
**the CPU and Memory usage look normal.**

## netstat -tulnp | grep nginx
**the service seems to be listening at port 80**

## journalctl -u nginx -n 20
**in logs we can see someone stopped the service and not started it**

## systemctl list-units --type=service --state=running
**checked active service, could not found nginx.**

## systemctl start nginx
**unable to start the service. getting error job for service failed. service not started successfully.**

## systemctl -n 50
**checked system logs and found UID=1000 stopped nginx and started the nginx service with hig priority using command - "nice -n -18 nginx"**

## pgrep -a nginx
**checked process details seems ok**

## top -p [pid]
**CPU and memory looks normal**

## sudo systemctl restart nginx

**if the above restart command does not work follow the below steps.**

**next step would be to kill the service and restart the service again using systemctl.**

## pgrep nginx
**get all the PID for nginx. kill each PID forcefullu if required.**

## sudo kill -9 [pid]
## systemctl restart nginx
**restart service using systemctl**

**validate the service again, check the status.**

**check the CPU and memory usgae after restart.**
## ps -aux | grep nginx

## after restart CPU and memory looks normal. no spike after restart. Logs for the service shows service restarted successfully.
