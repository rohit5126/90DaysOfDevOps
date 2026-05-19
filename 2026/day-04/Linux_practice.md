Today’s goal is to practice Linux fundamentals with real commands.


# Difference between service and process in linux.

**every service is a process but every process is not a service.**

Any instance of running program is a process and specilized process running the backgroup is service. the main difference is how both are managed. process is managed manually using commands where as services are managed by service manager like systemd or systemctl.

# Check running process.

## to check running process.
```bash
ps -ef
```

## to get details about nginx process and ssh process.

```bash
ps -ef | grep nginx && ps -ef | grep ssh

## check the status of the services.

systemctl status nginx
```

## to check real time running process.

```bash
top or htop
```

## to get PID of process

```bash
pgrep nginx
```
## to kill the process

```bash
sudo kill -9 PID
```
## check the status again. if it shows failed restart the service.

```bash
sudo systemctl restart nginx
```

## inspect service logs.

```bash

journalctl  #to view service logs
journalctl -u ssh #view specific service

journalctl -f #to follow live logs

journalctl | tail -n 50 #to see last 50 logs
journalctl | head -n 50 #to see top 50 logs

journalctl -b #to see logs filter by boot

journalctl --list-boots #to see reboot details.

journalctl --since "yesterday" #to see specific time logs, use words like 1 hour ago, or specific dates

journalctl -p err #to see error logs only.

journalctl -g [keyword] # use specific keyword to search for logs, like kill or uid

```

## inspect service file.

```bash

systemctl cat [servicename]
systemctl cat ssh
```

# Mini Troublehsooting flow.
# issue nginx not working and unable to restart the service.

**step 1 - check the running process and find nginx is there or not.**
```bash
ps -aux | grep nginx
```
**step 2 - Inspect nginx service and ports**
```bash

systemctl status nginx
systemctl restart nginx # to see if restart is working or not.

nc -zv localhost 80 #to check if port is working.

journalctl -u nginx | tail -n 50  #check te latest logs.
jorunalctl -u nginx -g kill #to check if service was killed and not started correctly.
```

**step 3 - if you found restart was not successfull or unable to restart services**
```bash

pgrep nginx #to get PID

#kill all the nginx PID forcefully.
sudo kill -9 3992
sudo kill -9 3993
sudo kill -9 3992    

# restart nginx using systemctl
sudo systemctl restart nginx
```

**step4 - validate again**
```bash
systemctl status nginx
```

**the service started successfully**






