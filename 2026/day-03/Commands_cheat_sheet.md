## 50 commands for linux cheat sheet
## process management

### 1. Monitoring & Viewing Processes

1. 'ps'-> it displays snapshot of current processes.
2. 'ps -aux' -> it displays all process with detailed info
3. 'ps -ef --forest' -> it displays and ASCII process tree to show parent child relationship.
4. 'top' -> it displays real time view of process running and system resource usage.
5. 'htop' -> it displays a colorful version of top with easier management.
6. 'pstree' -> it displays a process tree for easy visualization of hierarchy.
7. 'pgrep [name]' -> it displays the PID of the process.

### Terminating Processes
1. kill [PID] -> it is used to terminate a process.
2. kill -9 [PID] -> terminate a process forcefully. used as a last resort.
3. pkill [name] -> terminate a process using name.
4. killall [name] -> kill alll process matching a given name

### 3. Priority & Scheduling
1. nice -n [value] [command] -> Starts a new process with a specific priority (range: -20 for highest to 19 for lowest).
2. renice [value] -p [PID] -> to change priority of already running process.

### 4. Background & Foreground Management
1. '&' -> to run a command in backgroun
2. ctrl+z -> to pause a foreground process.
3. 'nohup' -> to run a command in backgroud that will run even after you logout.'
4. 'jobs' -> to check jobs running in background.
5. 'fg %[job_id]' -> to bring a background job to foreground.
6. 'bg %[job_id]' -> to resume a background job.

### 5. System Services & Uptime

1. 'systemctl' -> used to manage systemd services.
2. 'free -h' -> to see details about the RAM
3. 'uptime' -> hows how long systemm has been running.

## File systems

### 1. Navigation and Directory Info

1. 'ls' -> listing files and directory.
2. 'ls -l' -> show detailed info.
3. 'ls -a' -> show alll files, including hidden files.
4. 'ls -lh' -> display fie size iin human readable format.
   
### 2. File and Directory Operations

1. 'touch' -> create a new file.
2. 'mkdir -p [dir]' -> to ignore already exist error.
3. 'mkdir -p [path]' -> to create a parent dir.
4. 'cp [source_file] [dest_path]' -> to copy a file.
5. 'cp -r [source_dir] [dest_path]' -> to copy dir with its content.
6. 'mv [source_fileordir] [dest_path]' -> to move file or dir or renaming a file.
7. 'rm [file]' -> remove a file.
8. 'rm -r [dir]' -> to remove a dir.
9. 'rm -rvf [dir]' -> forcefullu remove a dir.

### 3. Viewing and Searching Files

1. 'cat [file]' -> to view content of a file
2. 'less [file]' ->  Opens a file in a scrollable view (press q to exit).
3. 'head -n 5 [file]' -> to see first 5 lines of the file
4. 'tail -n 5 [file]' -> to see last 5 lines of the file
5. 'find [path] -name "[file]"' -> to find a file in that particular path
6. 'grep [patternortext] [file]' -> to find a text in a file. it is case sensitive.

### 4. Disk and system info

1. 'df -h' -> it shows details about the storage or space left.
2. 'du -sh [dir]' -> to display the size of specific dir
3. 'mount' -> ists all currently mounted file systems.

## Networking troubleshooting

### Basic Connectivity & IP Configuration

1. 'ip a' or 'ip addr show' -> to see network details and ip assigned.
2. 'ping [IP]' -> to send ICMP packets to test reachability.
3. 'ip route' -> View or modify the routing table to see how traffic leaves the system
4. ethtool [interface] -> Query or modify network interface hardware and driver settings(ex- ethtool eth0)

### DNS & Name Resolution

1. 'dig [domain_name]' -> query DNS servre and get detailed records.
2. nslookup [domain_name] -> query DNS server to find Ip add.
3. cat /etc/resolv.conf -> Verify which DNS name servers the system is actively using.

### Port Connectivity & Services

1. 'nc -zv [host] [port]' -> netcat test to check if port is open on remote host.
2. 'ss -tulpn' -> View all listening network ports and the processes associated with them.
3. 'curl -I [URL]' -> Fetch HTTP headers to check web server responses

### Route Tracking & Packet Path

1. 'traceroute [host/ip]' -> Trace the path packets take to reach a target server.
2. 'mtr [host/ip] -> a diagnostic tool that combines traceroute and ping to continously show packets loss and latency.

### 5. Network Traffic & Packet Capture

1. 'sudo tcpdump -i [interface] -n -v' -> Capture and inspect live network packets on a specific interface.
2. 'ip -s link' -> display network interface statistics
3. 'sudo iftop' -> onitor real-time bandwidth usage by individual connections.

### 6. Active Connections & Sockets

1. 'ss -tunp' -> list all active TCP and UDP connection in real time.
2. 'lsof -i -P -n | grep LISTEN' -> List all programs with network files currently open in listening mode. 

