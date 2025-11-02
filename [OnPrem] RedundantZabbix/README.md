# Project Overview

![](https://github.com/JonmarCorpuz/projectsDump/blob/main/%5BOnPrem%5D%20RedundantZabbix/Images/topologie.png)

* Each service was installed on it's own virtual Ubuntu Server

<br>

# Technology Stack

| Technology | Purpose |
| --- | --- |
| Zabbix | For infrastructure and application monitoring, alerting, performance tracking, and visualization |
| MariaDB | The RDBMS that'll act as the backend database for storing all monitoring data, events, and configurations for Zabbix |
| Galera | For synchronous clustering of the MariaDB databases |
| Rsync | For fast file synchronization and copy tool to ensure that all the datases within the Galera cluster are synced |
| HAProxy | To serve as a reverse proxy and the public facing entry point for the Zabbix users |
| Keepalived | To provide a virtual IP address for the Zabbix frontend |
| Cisco Switches | For L2 forwarding of network packets |
| Cisco Router | For L3 routing of network packets and allow our internal network to reach the internet |
