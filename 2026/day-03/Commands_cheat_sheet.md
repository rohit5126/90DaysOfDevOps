## 50 commands for linux cheat sheet
## process management

### 1. Monitoring & Viewing Processes

1. 'ps' -> it displays snapshot of current processes.
2. 'ps -aux' -> it displays all process with detailed info
3. 'ps -ef --forest' -> it displays and ASCII process tree to show parent child relationship.
4. 'top' -> it displays real time view of process running and system resource usage.
5. 'htop' -> it displays a colorful version of top with easier management.
6. 'pstree' -> it displays a process tree for easy visualization of hierarchy.
7. 'pgrep <name>' -> it displays the PID of the process.

### Terminating Processes
1. kill <PID> -> it is used to terminate a process.
2. kill -9 <PID> -> terminate a process forcefully. used as a last resort.
3. pkill <name> -> terminate a process using name.
4. killall <name> -> kill alll process matching a given name

### 3. Priority & Scheduling
1. nice -n <value> <command> -> Starts a new process with a specific priority (range: -20 for highest to 19 for lowest).
2. renice <value> -p <PID> -> to change priority of already running process.

### 4. Background & Foreground Management
1. '&' -> to run a command in backgroun
2. ctrl+z -> to pause a foreground process.
3. 'nohup' -> to run a command in backgroud that will run even after you logout.'
4. 'jobs' -> to check jobs running in background.
5. 'fg %<job_id>' -> to bring a background job to foreground.
6. 'bg %<job_id>' -> to resume a background job.

### 5. System Services & Uptime

1. 'systemctl' -> used to manage systemd services.
2. 'free -h' -> to see details about the RAM
3. 'uptime' -> hows how long systemm has been running.
4. 'df -h' -> it shows details about the storage or space left.


