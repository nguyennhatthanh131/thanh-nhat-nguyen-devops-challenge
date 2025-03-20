## Task 3: Diagnose Me Doctor

### Troubleshooting a VM with 99% Storage Usage

#### Initial Investigation

1. **Get an overview of disk usage**
   ```bash
   df -h  # Check disk usage by filesystem
   du -sh /*  # Find largest directories
   ```

2. **Check running processes**
   ```bash
   ps aux --sort=-%mem  # Check if any process is consuming abnormal resources
   ```

3. **Examine NGINX logs**
   ```bash
   ls -lah /var/log/nginx/  # Check log sizes
   ```

#### Likely Root Causes and Solutions

1. **Unconfigured Log Rotation**
   - Signs: Enormous /var/log/nginx/access.log or error.log files
   - Impact: Continuous log growth until disk is full, leading to service disruption
   - Recovery:
     ```bash
     # Immediate fix
     sudo truncate -s 0 /var/log/nginx/access.log
     sudo truncate -s 0 /var/log/nginx/error.log
     
     # Long-term fix
     sudo nano /etc/logrotate.d/nginx
     ```
     Configure proper logrotate settings:
     ```
     /var/log/nginx/*.log {
         daily
         missingok
         rotate 14
         compress
         delaycompress
         notifempty
         create 0640 www-data adm
         sharedscripts
         postrotate
             [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
         endscript
     }
     ```

2. **Excessive Caching Files**
   - Signs: Large size of /var/cache/nginx/
   - Impact: Inefficient resource usage, potential service slowdown
   - Recovery:
     ```bash
     # Clear cache
     sudo rm -rf /var/cache/nginx/*
     
     # Update NGINX config
     sudo nano /etc/nginx/nginx.conf
     ```
     Add proper cache size limits:
     ```
     proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
     ```

3. **Core Dumps**
   - Signs: Large files in /var/crash/ or core dump files in working directory
   - Impact: Diagnostic files consuming space after application crashes
   - Recovery:
     ```bash
     # Remove existing dumps
     sudo find / -name "core*" -delete
     
     # Disable core dumps
     sudo nano /etc/security/limits.conf
     ```
     Add: `* soft core 0`

4. **Docker Images and Containers**
   - Signs: Large /var/lib/docker directory
   - Impact: Unused images/containers consuming space
   - Recovery:
     ```bash
     # Clean unused docker resources
     docker system prune -af --volumes
     ```

5. **Old Package Archives**
   - Signs: Large /var/cache/apt/ directory
   - Impact: Minimal performance impact, but wastes space
   - Recovery:
     ```bash
     sudo apt clean
     sudo apt autoremove
     ```