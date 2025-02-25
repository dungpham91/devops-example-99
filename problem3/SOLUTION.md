# Here are the steps I usually take to find the problem and fix it

## 1. Use monitoring system (if available)

In case we have a monitoring system like Prometheus - Grafana, New Relic, Datadog and log systems like ELK:

- Use these systems to check the real-time resources on the server, from there determine the abnormal time period.

- Check the number of connections that Nginx has, match the abnormal time period above, is it a DoS/DDoS attack?

- In case it is suspected that it is a DoS/DDoS attack, check the log from the log system to determine the IPs that are sending requests to Nginx that have increased suddenly, filter the list to prepare to put it on the firewall block list.

- Add CDN solutions like Cloudflare, fail2ban and set max_connection and rate_limit on Nginx to reduce connections immediately.

## 2. Use SIEM system (if available)

- In case we have a SIEM system, it will help in determining whether the server is being attacked or not? Or is the server infected with malware or not?

- If the server is not being attacked by DoS/DDoS, there is a second high possibility that the server is infected with malware that leads to always full resources (for example, coin mining malware).

- At this point, we need to use tools such as ClamAV or Security Endpoints (commercial) to scan for where the malware is and eliminate it.

- With the help of SIEM, the problem can be identified more quickly, otherwise we will have to do everything manually and spend more time.

- If the second case happens, that is, the server is infected with malware, it will be very troublesome. We need to immediately isolate it from the entire system by blocking its connection to other servers/systems. At the same time, build a new Nginx LB node to replace it while isolating.

- Next, we can investigate step by step how the malware was injected into the server, how long it has been happening. Is it infecting any other systems? This needs to be done urgently.

## 3. Manual investigation

This is the case where we do not have a monitoring system and a SIEM system.

- We can use Linux commands like `top`, `htop`, `df`, `free` to check the server resources. Use tools like `netstat` to check how the connection to the server is. Combine with checking the `nginx log` to see unusual requests from which IP. From there, determine whether there is a DoS/DDoS attack or not? If so, we will apply the measures I have outlined above.

- In the second case, malware, they need to check the `ps` command to see if it is Nginx alone that is taking up resources or if there are any other hidden processes running (for example, coin mining malware). Simultaneously perform a virus scan to find the malware and handle it as I said above in `section 2.`

- If it is not the above 2 cases, next, we need to check the internal service. We need to determine whether the upstream services are working properly or not? Because, there may be a case where the upstream error causes the connection in Nginx LB to be held for a long time, leading to a large amount of resource consumption. If there is an error from upstream, it must be handled immediately or the temporary solution is to restart the Nginx service.

- The next possible case is that Nginx configuration is wrong, we need to check if Nginx is misconfigured in any part related to `worker_processes`, `worker_connections`, `keepalive_timeout` or not? If so, we need to adjust these parameters immediately and reload nginx. We can also look at nginx's `access_log` and `error_log` to determine more clearly about this.

- The next case is to check the disk capacity and the existing Nginx cache files. It is very possible that the disk is almost full or the Nginx cache files have problems when stored on the disk, or the server's swap is prioritized to run more than RAM, leading to connections to Nginx being hung for a long time, thereby nginx consuming a large amount of resources (memory). You need to clean the cache or free the disk if necessary (by deleting unnecessary logs) and then restart nginx.

- After all the above cases, the last possible case is a hardware failure. I have encountered a few times in the past when the physical server had a RAID card failure, which caused the VPS's read/write speed to slow down dozens of times. This can cause Nginx to process requests very slowly and consume a lot of memory. Or the server's internet/LAN connection has a problem (for example, the cable head is loose, the copper core is broken, ...). These physical problems are usually checked last and also fixed last because it is very time-consuming, requiring a system backup, migration to another physical node and replacement of equipment. If there is a monitoring system here, it will help to quickly identify the problem with the network as well. But here I am assuming we are investigating in phase 3, which is manual investigation. And in addition, we are using cloud vps so we will contact the supplier to handle this problem.
