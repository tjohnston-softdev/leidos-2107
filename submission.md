# Junior-Mid Software Developer

## Scenario

A company has an application running on a server that is critical and required to always be accessible. This is called a 'high availability requirement'. Developers are required to migrate changes twice per week on Tuesday and Thursday.

The issue is that a C: drive temp folder occasionality gets large logs. A verbose (large and detailed) log from a developer sometimes goes in during development testing and can cause starvation. How would you address this? It does not happen all the time, only sometimes and it is unpredictable as to when it will happen.

When this happens the machine can shut down ungracefully.

---

### Q1 - Can we check the machine is on? If yes, how?
The most primitive way to do this would be to send a ping to the machine's IP Address. This will send a series of test packets to the target server, which will then send reply packets back to our machine. This can be done using the system terminal, such as 'Command Prompt' for Windows. To ping a server using the Windows terminal, type the command `ping <ip-address>`. If this is successful, we can assume that the machine is turned on, connected to the internet, and able to send/receive data.

Another, fancier way of doing this would be to incorporate server monitoring or remote access software. The exact program you choose will depend on your use cases and personal preferences but the ones that I think best match this scenario from what I have found are [TeamViewer](https://www.teamviewer.com/en-us/remote-management/remotemonitoring/), [SysGauge](https://www.sysgauge.com/index.html), and [RemotePC](https://www.remotepc.com/features). Not only will these programs check whether a server is online, they also enable a variety of features such as real-time resource monitoring.

---

### Q2 - If machine is off, is there a way to remotely turn it on? If yes, how?
One way to do this would be using 'Wake-on-LAN' (WoL). When a device is switched off, it does not actually 'turn off' completely, it merely enters a low-power standby mode. Essentially, WoL works by sending a data packet to the asleep computer containing the network card's MAC Address. After this packet is received, it will be interpreted as an instruction to turn on the 'asleep' computer, hence, waking it up. The process for setting up WoL is too long and complicated to explain here, but there are helpful tutorials on [Appuals](https://appuals.com/how-to-turn-on-your-pc-remotely/) and [How-To-Geek](https://www.howtogeek.com/70374/how-to-geek-explains-what-is-wake-on-lan-and-how-do-i-enable-it/).

As for sending the WoL command remotely, there is dedicated software that you can use. In addition, some server monitoring tools such as TeamViewer include this as a feature.

---

### Q3 - Machine isn’t responding to ping, can we restart the machine? If yes, how?
To start, just because a machine does not respond to 'ping', it does not necesarially mean that it is offline. It could be offline but it is also possible that the machine is just too busy responding to other requests or its bandwidth has been exceeded. Assuming the computer is disconnected from the internet, remote access would then be very difficult, impractical, or just impossible. I have actually tried to research ways of restarting a computer remotely without an IP address but I have not found any conclusive methods. What I did find are ways to be able to detect the offline state as quickly as possible so that it can be fixed accordingly. Some of it relates to automation.

For example, you could set up a script to ping the server from a separate machine. You should allow some fault tolerance but if the ping request fails multiple times in a row, we would assume that the computer is offline and send a notification.

A similar idea is to write a script that runs locally on the server itself to check if internet access is available. One way to do this would be for the server to ping itself (127.0.0.1). If the online check fails beyond fault tolerance, you could take action such as troubleshoot the connection or restart the server entirely. This does not restart the machine remotely but it does at least enable an automated restart if the server falls offline, and send a notification to relevant IT staff.

Assuming that just the server is offline and not the whole local network, you can also try remotely accessing the router. If you are able to do this, you can then restart the router and hopefully restore internet access across the LAN. There are articles on how to do this on [Lifehacker](https://lifehacker.com/remotely-reboot-your-router-from-any-browser-5548323) and [Infravio](https://www.infravio.com/how-to-reset-a-router-from-a-computer/).

A more complex but still relevant method you might want to try is hooking the server up to a Power Distribution Unit (PDU). If the server is deemed offline, you can remotely access the PDU and restart the server from there. This is similar to the router method except that it restarts the whole machine and not the router. You could try either or both. I found an [article](https://www.dpstele.com/power-distribution-unit/pdu-snmp-server-room.php) that goes into more detail, and a [PDU](https://dlidirect.com/products/new-pro-switch) that can reboot an offline server automatically.

---

### Q4 - Machine now responds to ping, can we remotely access it? If yes, how? Is there anything we should check once the server is healthy?
(Following up from Question 3)

Now that the server responds to 'ping', you should be able to remotely access it as described in Question 1. As for checking the system health, even a successful ping and enabled remote access is an important health indication. It shows that the server is alive and is able to communicate. It is similar to "Hello World" in programming.

As for further checks, I would start by checking the system resource metrics to ensure that it is running as normal within a safe range. If you are not satisfied, you can even choose to perform benchmarks. Monitoring the response times of different functions and components can help you determine whether performance is satisfactory. If the performance is poor or inconsistent, it could be an early sign of a potential issue.

If this is after recovering from outage, you should check any relevant logs and crash dumps for more detail as to what went wrong and how much damage was caused. If there are any automated or background processes, you should check the corresponding dashboard on your monitoring software to ensure that they are running as expected. It is also worth considering checking for any new OS, driver, or security updates. In order to minimise downtime, this should be saved for low-peak time or a dedicated maintenance period.

If you have the available staff, you should also have someone check on the server physically. While the physical maintenance of cloud servers is outsourced to the provider, your physical servers need to be protected from environmental hazards, physical intrusion, and other forms of damage. If nothing else, be sure to monitor the power supply and temperature. It is even better if your chosen monitoring software is able to measure these variables remotely.

While it is important to check up on server health after recovering from downtime, healthy maintenance should be an ongoing process in general. For an example in terms of remote monitoring, you can not only view metrics at a glance but also set alerts that trigger when sustained resource consumption is at unsafe levels. One needs to be mindful of total system capacity and set  these thresholds well below the upper limit. High availability is a requirement and the biggest threat to any business is downtime. On that note, an overloaded server can be just as bad as an offline server as far as the average user is concerned. If the warning threshold is set conservatively, IT staff can address any potential issues before they become problematic.

For more comprehensive information related to server health, there are articles from [Rackaid](https://www.rackaid.com/blog/server-maintenance-checklist/), [WhatsUp Gold](https://www.whatsupgold.com/blog/assess-server-health-easy-way), and [Comparitech](https://www.comparitech.com/net-admin/server-monitoring-best-practices/#Server_performance_monitoring_tasks).

---

### Q5 - How do we remove logs?
The most obvious way to remove the logs is to delete the files themselves manually. Depending on how the log files are structured, you could write a script to automatically delete files that are older than a specified age. While those solutions would be functional for what they are, they might not be viable for a long-term solution.

Since these log files are generated during development testing, it is important to differentiate between production and testing environments. As a back-end software developer, I know first-hand that it is possible to program a script to function differently based on whether it is in a testing environment or a production environment. I won't go into specific implementation details but as an example, you could generate verbose logs in testing mode while generating simpler logs in production mode.

All the same, the general problem is that these log files accumulate overtime. This consumes an unnecessary amount disk space, eventually to the point of crashing the system entirely. As mentioned, deleting these files is the most obvious solution but we still want to be able to access logs as needed. The real question at this point is how to manage these log files so that they are active for a reasonable amount of time without using too much space.

One possible way to do this is 'Log rotation'. This involves the automatic compression and archival or log files based on certain variables such as the file's size and age. The overall purpose of rotation is to keep the log files small enough so that they can be safely opened for review, and to archive them once they are too old.

It would be up to you to decide what makes a log file too 'old' or too 'large'. You should also decide how much disk space would be allocated for logging. This can be a fixed amount such as 500MB, or a percentage of the total drive capacity. Most of the information available for log rotation relates to Unix-based systems, but you would be able to write a script yourself to perform log rotation as a cron job on Windows systems.

---

### Q6 - How do we then check periodically how full the drive is? Are there ways to manage an ungraceful shutdown of an application?
Through the use of server monitoring tools, you are able to view resource consumption metrics in real time such as CPU, RAM, and disk space. That seems like a simple answer but in practice, you would want a solution that automatically checks the available disk space and takes immediate action when it is deemed to be too low. As discussed in Question 4, it could send a notification when this happens so that a staff member can perform clean-up themselves. You can also write a script to remove common leftover files without user intervention, similar to [CCleaner](https://www.ccleaner.com/). Obviously there are no right or wrong ways to implement this. It will depend on the team and what software they use. In my research, I found that TeamViewer offers automated disk space monitoring, including remote script execution.

Refer to Question 5 for how to manage log size.

As for managing the ungraceful shutdown of an application, the first thing to do would be attempting to diagnose the problem. Identifying the root cause of the issue allows you to correct those errors and prevent them from causing crashes in the future. Diagnosing a software problem would be an entire case study in itself but a good starting point is to inspect any logs or output files generated by the script. The information contained within these files may hold clues as to what caused the crash. For example, if the log messages suggest Input/Output, the problem might be to do with reading and writing files. Hence, you will want to investigate and debug corresponding areas of the script.

When I debug code, I start by adding console output at key milestones. That way, when those milestones are reached, we know the script works up until that point. If a particular milestone isn't reached, that means the error likely happens between milestones X and Y. Then, I set a breakpoint after milestone X and walk through the code step-by-step until something unusual occurs. This could be a runtime error or an incorrect value but what you find at this step will help you determine the source of the error and how it might be fixed. For more information about this process, I have found articles related to [JavaScript](https://www.w3schools.com/js/js_debugging.asp) (W3Schools) and [Microsoft Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/debugging-absolute-beginners?view=vs-2019&tabs=csharp#step-through-your-code-in-debugging-mode-to-find-where-the-problem-occurred).

When I write code, I like to include as much error checking as possible. This could be in the form of [try-catch blocks](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch), or IF conditions that manage to filter out invalid values before they cause damage. That way, when something goes wrong, the errors are handled as gracefully as possible with a helpful error message instead of crashing straight away. For production applications, you will want to include these errors and other helpful information in the log files so that if there is an ungraceful shutdown, we will at least have some idea of what went wrong.

Another topic worth addressing is the loss of data. I believe that ungraceful shutdowns are ungraceful. There is no sure way to completely prevent nor recover from these events. However, what we must do is minimise the damage caused. One way to do this in my opinion is by not holding data in memory any longer than necessary. In other words, after you are done modifying data, save it to an external file. When an application crashes, all of the data kept in memory vanishes whereas once it is saved, it will remain intact if the program crashes (Assuming the drive isn't corrupted, of course). It is just like when writing text in a word processor. You should save as often as you feel necessary so that if something crashes, the loss will only be minimal.

If this application needs to be running constantly, ensure that it is running as a [daemon process](https://en.wikipedia.org/wiki/Daemon_(computing)). In summary, when the application itself crashes (Not the whole computer, mind you), it will start up again automatically. Of course, there might be some downtime depending on your infrastructure but at least the application will start itself again automatically.

---

## References

**Question 1**
* [How to Ping a Computer or a Website](https://www.lifewire.com/how-to-ping-computer-or-website-818405)  
-Bradley Mitchell (Lifewire)  
Updated 2 July 2021

* [TeamViewer - Remote Device Monitoring](https://www.teamviewer.com/en-us/remote-management/remotemonitoring/)

* [SysGauge - System Monitor](https://www.sysgauge.com/index.html)

* [RemotePC by IDrive](https://www.remotepc.com/features)

\
**Question 2**

* [How to Turn on your PC Remotely using Wake-on-Lan](https://appuals.com/how-to-turn-on-your-pc-remotely/)  
-Kevin Arrows (Appuals)  
9 April 2021

* [What Is Wake-on-LAN, and How Do I Enable It?](https://www.howtogeek.com/70374/how-to-geek-explains-what-is-wake-on-lan-and-how-do-i-enable-it/)  
-Yatri Trivedi and Michael Crider (How-To-Geek)  
28 July 2017, 6:40AM EDT

* [How To Remotely Wake Up Your Windows 10 PC](https://helpdeskgeek.com/windows-10/how-to-remotely-wake-up-your-windows-10-pc/)  
-Ben Stockton (Help Desk Geek)  
18 August 2020

\
**Question 3**

* [How to Monitor and Alert When Servers Go Offline](https://knowledge.broadcom.com/external/article/178920/how-to-monitor-and-alert-when-servers-go.html)  
-Broadcom Knowledge Center  
Updated on 17 February 2017

* [Rebooting a remote Windows 10 machine not responding via mstsc or shutdown](https://community.spiceworks.com/topic/2270006-rebooting-a-remote-windows-10-machine-not-responding-via-mstsc-or-shutdown)  
-eNMok (Spiceworks community forum)  
Thread opened 27 April 2020, 9:31PM

* [How to Reset a Router from a Computer](https://www.infravio.com/how-to-reset-a-router-from-a-computer/)  
-Andrew Namder (Infravio)  
25 May 2021

* [Remotely Reboot Your Router from Any Browser](https://lifehacker.com/remotely-reboot-your-router-from-any-browser-5548323)  
-Whitson Gordon (Lifehacker)  
26 May 2010, 2:00PM

* [Server Room PDU: Remote Reboot Units Using SNMP](https://www.dpstele.com/power-distribution-unit/pdu-snmp-server-room.php)  
-DPS Telcom

* [New! Pro Switch](https://dlidirect.com/products/new-pro-switch) (Yes that is the actual name)  
-Digital Loggers Inc.

\
**Question 4**

* [12 Point Server Maintenance Checklist](https://www.rackaid.com/blog/server-maintenance-checklist/)  
-Jeff Huckaby (Rackaid)  
10 March 2018

* [How to Assess Server Health the Easy Way](https://www.whatsupgold.com/blog/assess-server-health-easy-way)  
-Joe Hewitson (WhatsUp Gold)  
14 September 2020

* [Server Monitoring Best Practices – How To Monitor Server Health](https://www.comparitech.com/net-admin/server-monitoring-best-practices/#Server_performance_monitoring_tasks)  
-Stephen Cooper (Comparitech)

\
**Question 5**

* [Wikipedia - Log rotation](https://en.wikipedia.org/wiki/Log_rotation)

\
**Question 6**

* [4 Things You Can Do When Your Production App Crashes](https://technologypartners.net/blog/2017/10/4-things-can-production-app-crashes/)  
-Technology Partners  
4 October 2017

* [W3Schools - JavaScript Debugging](https://www.w3schools.com/js/js_debugging.asp)

* [Stepping Through Code](https://docs.microsoft.com/en-us/visualstudio/debugger/debugging-absolute-beginners?view=vs-2019&tabs=csharp#step-through-your-code-in-debugging-mode-to-find-where-the-problem-occurred)  
-Microsoft Visual Studio documentation  
14 February 2020

* [try-catch (C# Reference)](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch)  
-Microsoft Visual Studio documentation  
20 July 2015

* [Does your Web Application on Production Servers Crash Unexpectedly?](https://varunvns.wordpress.com/2011/09/13/does-your-web-appcrash-unexpectedly/)  
-Varun (Sitecore Endeavor)  
13 September 2011

* [Wikipedia - Daemon (computing)](https://en.wikipedia.org/wiki/Daemon_(computing))

* [Ccleaner](https://www.ccleaner.com/)

---
