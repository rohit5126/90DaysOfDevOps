Today’s goal is to practice Linux fundamentals with real commands.

You will create a short practice note by actually running basic commands and capturing what you see:

Check running processes
Inspect one systemd service
Capture a small troubleshooting flow
This is hands-on. Keep it simple and focused on fundamentals.
# Difference between service and process in linux.

** every service is a process but every process is not a service.**
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
