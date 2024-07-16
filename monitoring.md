
<h1> Monitoring Setup</h1>

**[ritu@pal ~]$ podman run -d -p 3001:3000 --name=grafana docker.io/grafana/grafana-enterprise**

Trying to pull docker.io/grafana/grafana-enterprise:latest

Getting image source signatures

Copying blob 72db4ea52f68 done   
|
Copying blob 7f3c05061126 done  
 |
Copying blob 9213063c7a06 done   |`

Copying blob f5d9b74cdbda done   |

Copying blob 4abcf2066143 done   |

Copying blob 5d7c6f7faf4f done   |

Copying blob 1304d6cda511 done   |

Copying blob ec80b701a02f done   |

Copying blob d55f36d32694 done   |

Copying blob 9535f245c480 done   |

Copying config fc3384f5f9 done   |

Writing manifest to image destination

7005b14586c6f81a8c41fa8e3fd3b3e96fd92083bcc042e4813a3f880969c8f9
```
[ritu@pal ~]$ podman ps
CONTAINER ID  IMAGE                                    	COMMAND 	CREATED     	STATUS     	PORTS               	NAMES
7005b14586c6  docker.io/grafana/grafana-enterprise:latest          	17 seconds ago  Up 16 seconds  0.0.0.0:3001->3000/tcp  grafana
```

```
[ritu@pal ~]$ mkdir prometheus
[ritu@pal ~]$ cd prometheus/
[ritu@pal prometheus]$ ls
[ritu@pal prometheus]$  vim prometheus.yml
```

global:

  scrape_interval: 10s

rule_files:

  - './rules.yml'

alerting:

  alertmanagers:

	- static_configs:

    	- targets: ['192.168.1.24:9093']

scrape_configs:

  - job_name: 'prometheus'

	scrape_interval: 5s

	static_configs:

  	- targets: ['192.168.1.24:9090']

  # Blackbox exporter for HTTP checks
  - job_name: 'blackbox'

	metrics_path: /probe

	params:

  	module: [http_2xx]

	static_configs:

  	- targets:
    	- http://keen.fosteringlinux.com
    	- https://www.google.com

	relabel_configs:
  	- source_labels: [__address__]

    	target_label: __param_target

  	- source_labels: [__param_target]

    	target_label: instance

  	- target_label: __address__

    	replacement: 192.168.1.24:9115  # Provide your server's private IP here

  # Node exporter for system metrics
  - job_name: 'node_exporter'

	static_configs:

  	- targets: ['172.16.0.253:9100', '172.16.0.180:9100', '192.168.1.24:9100']

  # PostgreSQL exporter for database metrics
  - job_name: 'pg_exporter'

	static_configs:

  	- targets: ['192.168.1.24:9187']

Create rules.yml

groups:

  - name: InstanceDown

	rules:

  	- alert: InstanceDown

    	expr: up == 0

    	for: 1m

    	labels:

      	severity: "critical"

    	annotations:

      	summary: "Endpoint {{ $labels.instance }} down"

      	description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."






```
[ritu@pal prometheus]$   podman run -d --name prometheus -p 9090:9090 -v /home/ritu/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/ritu/monitoring/rules.yml:/etc/prometheus/rules.yml docker.io/prom/prometheus
```

Trying to pull docker.io/prom/prometheus:latest...

Getting image source signatures

Copying blob fbd390d3bd00 done
   |
Copying blob 9fa9226be034 done
   |
Copying blob 3ecda1bfd07b done  
 |
Copying blob ac9f4de4b762 done 
  |
Copying blob ea63b2e6315f done 
  |
Copying blob 1617e25568b2 done 
  |
Copying blob 9b1ac15ef728 done  
 |
Copying blob 8682f304eb80 done 
  |
Copying blob 5fbafe078afc done  
 |
Copying blob 7fb53fd2ae10 done  
 |
Copying blob 592798bd3683 done 
  |
Copying blob 473fdc983780 done 
  |
Copying config b74abbcc4e done 
  |
Writing manifest to image destination

094b1e9804ac985904e9eaed2029ce0d282d985d0937e6bdaaa2db17feb83ace
[ritu@pal prometheus]$

```[
    ritu@demo monitoring]$ podman run -d --name blackbox_exporter -p 9115:9115 docker.io/prom/blackbox-exporter
3e019b752f49760b0e06a823e03f5289cfdf7f30fc32dc936f88a0d0d4582aa5
```


**Grafana dashboard**

![alt text](<Screenshot from 2024-07-16 12-50-39.png>)

![alt text](<Screenshot from 2024-07-16 12-51-12.png>)

**Prometheus dashboard**
![alt text](<Screenshot from 2024-07-16 12-58-24.png>)

![alt text](<Screenshot from 2024-07-16 13-01-16.png>)


Node exporter 
![alt text](<Screenshot from 2024-07-16 13-03-09.png>)


Blackbox exporter
![alt text](<Screenshot from 2024-07-16 13-03-38.png>)



SETUP NODE EXPORTER FOR BOTH VMS

Vm-1


Check metrics
 curl 172.16.0.180:9100/metrices



 sudo firewall-cmd --permanent --add-port=9100/tcp

**Check this from base machine (192.168.1.24)**
```
[ritu@pal monitoring]$ telnet 172.16.0.180 9100
Trying 172.16.0.180...
Connected to 172.16.0.180.
Escape character is '^]'.
```

Vm-2

[dr-network@pg-1 ~]$ sudo firewall-cmd --reload

success

[dr-network@pg-1 ~]$ sudo firewall-cmd --list-all

```public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp1s0
  sources:
  services: cockpit dhcpv6-client http https postgresql ssh
  ports: 3000/tcp 5432/tcp 80/tcp 22/tcp 443/tcp 82/tcp 5433/tcp 9999/tcp 9898/tcp 9000/tcp 9694/udp 9100/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
  ```






Node exporter dashboard  (1860) 



Alertmanager dashboard

![alt text](<Screenshot from 2024-07-16 13-05-44.png>)

 Podman run -d -p 9093:9093 --name alertmanager -v /home/ritu/monitoring/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml docker.io/prom/alertmanager

Alertmanager.yml

[ritu@pal alertmanager]$ cat alertmanager.yml

global:

  resolve_timeout: 10s

route:

  receiver: 'gmail-notifications'

  routes:

	- match:

    	severity: 'critical'

  	receiver: 'gmail-notifications'

receivers:

  - name: 'gmail-notifications'

	email_configs:

  	- to: 'ritupal105398@gmail.com'

    	from: 'palritu2001@gmail.com'

    	smarthost: 'smtp.gmail.com:587'

    	auth_username: 'palritu2001@gmail.com'

    	auth_identity: 'palritu2001@gmail.com'

    	auth_password: 'jfjnzqrsjcamdhzm'

    	send_resolved: true






