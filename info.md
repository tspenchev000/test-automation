## Cunsul information

### Access consul at <a href="https://ld5tmlvault02.tfxcorp.com" target="_blank">link</a>

### Install Consul as backend state for Terraform installation
Machine is LD5TMLVAULT02

```
wget https://releases.hashicorp.com/consul/1.20.1/consul_1.20.1_linux_amd64.zip
unzip consul_1.20.1_linux_amd64.zip
mv consul /usr/bin
mkdir -p /etc/consul.d/
useradd consul -d /opt/consul
```

Define consul as a service via 
nano /usr/lib/systemd/system/consul.service
```
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/consul.d/consul.hcl

[Service]
Type=notify
EnvironmentFile=-/etc/consul.d/consul.env
User=consul
Group=consul
ExecStart=/usr/bin/consul agent -config-dir=/etc/consul.d/
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGTERM
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
```
systemctl enable consul.service
systemctl start consul.service
```

Consul will listen on port 8500. You need to access it on this port or to configure nginx as reverse proxy for specifc FQDN


