This repository is based on the [Intrinsec/nftables_exporter project: Expose nftables rules labeled by protocol, table, and chain](https://github.com/Intrinsec/nftables_exporter) and is compiled for ARM v8.

This is the dashboard created with this exporter in Grafana:

![grf](https://github.com/user-attachments/assets/56287b70-6741-4e29-8f56-774118878367)


To install it on an OpenWRT device (ARMv8), download the binary from the repository: [nftables_exporter](https://github.com/giuliomagnifico/openwrt-nftables_exporter/blob/main/nftables-exporter)

>[!NOTE]
>You can also compile it from the original source if you're using another architecture:
> `git clone https://github.com/Intrinsec/nftables_exporter.git && cd nftables_exporter` and compile with your flags. Example for arm v7: `GOOS=linux GOARCH=arm GOARM=7 go build -o nftables_exporter_arm ./cmd/nftables_exporter`
 
Move it to the `/usr/bin/` folder and make it executable with `chmod +x /usr/bin/nftables-exporter`

Add the service at startup by creating the file `/etc/init.d/nftables-exporter` with the following content:

 ```
#!/bin/sh /etc/rc.common

START=99
STOP=10

NAME=nftables_exporter
BIN=/usr/bin/nftables-exporter
CONFIG=/etc/nftables_exporter.yaml
PIDFILE=/var/run/$NAME.pid

start() {
    echo "Starting $NAME..."
    start-stop-daemon -S -b -m -p "$PIDFILE" --exec "$BIN" -- -config="$CONFIG" >> /var/log/$NAME.log 2>&1
}

stop() {
    echo "Stopping $NAME..."
    start-stop-daemon -K -p "$PIDFILE"
}
```


Make it executable with `chmod +x /etc/init.d/nftables-exporter`. Start it using `/etc/init.d/nftables-exporter start` and verify if is running:

```
root@R4S:~# pgrep -fa nftables-exporter
3868 /usr/bin/nftables-exporter
```

If it's running fine, you can enable it at login with `/etc/init.d/nftables_exporter enable`

### Prometheus

To add the scraping point to Prometheus:

```
- job_name: "nftables-exporter"
  static_configs:
    - targets: ["openwrt_ip:9630"]
      labels:
        name: "OpeWrt nftables exporter"
```

If all works you will see the metrics at `http://openwrt_ip:9630/metrics`

### Grafana

The JSON of my Grafana dashboard can be found here: [dashboard.json](https://github.com/giuliomagnifico/openwrt-nftables_exporter/blob/main/dashboard.json)
