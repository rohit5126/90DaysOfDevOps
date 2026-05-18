## 50 commands for linux cheat sheet
## process management

### 1. Monitoring & Viewing Processes

'ps' -> it displays snapshot of current processes.
'ps -aux' -> it displays all process with detailed info
'ps -ef --forest' -> it displays and ASCII process tree to show parent child relationship.
'top' -> it displays real time view of process running and system resource usage.
'htop' -> it displays a colorful version of top with easier management.
'pstree' -> it displays a process tree for easy visualization of hierarchy.
'pgrep <name>' -> it displays the PID of the process.

### Terminating Processes
kill <PID> -> it is used to terminate a process.
kill -9 <PID> -> terminate a process forcefully. used as a last resort.
pkill <name> -> terminate a process using name.
killall <name> -> kill alll process matching a given name

### 3. Priority & Scheduling
nice -n <value> <command> -> Starts a new process with a specific priority (range: -20 for highest to 19 for lowest).
renice <value> -p <PID> -> to change priority of already running process.

### 4. Background & Foreground Management
'&' -> to run a command in backgroun
ctrl+z -> to pause a foreground process.
'nohup' -> to run a command in backgroud that will run even after you logout.
'jobs' -> to check jobs running in background.
fg %<job_id> -> to bring a background job to foreground.
bg %<job_id> -> to resume a background job.

### 5. System Services & Uptime

'systemctl' -> used to manage systemd services.
'free -h' -> to see details about the RAM
'uptime' -> hows how long systemm has been running.
'df -h' -> it shows details about the storage or space left.


