# AcuRiteScraper
A python-based set of scripts to capture weather station data and dump it into a database.

## RTL-433

### Building and Installing the Tools
This set of scripts depends on the rtl_433 package to be installed, more information about it can be found on the GitHub
page located [here](https://github.com/merbanan/rtl_433).

If using the RTLSDR v4 dongle and the host system is Debian 12, both rtl-sdr and rtl-433 must be installed from source
as of the time of this writing, only version 22.01 is available in the package repositories and the dependency on
rtl-sdr version (0.6.0) doesn't work with the v4 dongle.

Instructions for installing rtl-sdr from source can be found [here](https://github.com/rtlsdrblog/rtl-sdr-blog/).

Instructions for install rtl-433 from source can be found [here](https://github.com/merbanan/rtl_433/blob/master/docs/BUILDING.md).

When building rtl-433 from source, the first cmake step should explicitly state the following:

```text
-- Found LibRTLSDR: /usr/local/lib/librtlsdr.so (found version "0.6.0-128-g240b")
```

If it only shows version "0.6.0" the generic version of rtl-sdr which is available in the Debian 12 package repositories
is being used.

### Using the Tools
The rtl-433 tool can be easily used with the following command, limiting it to the AcuRite 5n1 device-specific protocol:

```bash
sudo rtl_433 -R 40
```

It is possible to store the captured data directly into an InfluxDB with the command:

```bash
sudo rtl_433 -R 40 -F "influx://<hostname>:8086/api/v2/write?org=<org>&bucket=<bucket>,token=<authtoken>"
```

### Configuring a Service
A really great article explaining how to create a systemd service for keeping rtl_433 running can be found [here](https://www.apalrd.net/posts/2021/rtl433/).

The gist is to create a file using these commands:

```bash
sudo touch /etc/systemd/system/rtl433.service
sudo chmod 664 etc/systemd/system/rtl433.service
sudo nano /etc/systemd/system/rtl433.service
```

Then add this to the file:

```ini
[Unit]
Description=rtl_433 SDR Receiver Daemon for AcuRite
After=network-online.target

[Service]
ExecStart=/usr/local/bin/rtl_433 -R 40 -F "influx://<hostname>:8086/api/v2/write?org=<org>&bucket=<bucket>,token=<authtoken>"
Restart=always

[Install]
WantedBy=multi-user.target
```

Finally, reload systemd and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start rtl433
sudo systemctl enable rtl433
```

### Grafana Flux Language
After the source is defined in Grafana and a new dashboard is created with a panel that contains the influxdb source,
the following snippets need to be added for each type of metric:

```influxql
-- Temperature
from(bucket: "AcuRite")
    |> range(start: -24h)
    |> filter(fn: (r) => r._field == "temperature_F")

-- Humidity
from(bucket: "AcuRite")
    |> range(start: -24h)
    |> filter(fn: (r) => r._field == "humidity")

-- Wind Speed
from(bucket: "AcuRite")
    |> range(start: -24h)
    |> filter(fn: (r) => r._field == "wind_avg_km_h")

-- Wind Direction
from(bucket: "AcuRite")
    |> range(start: -24h)
    |> filter(fn: (r) => r._field == "wind_dir_reg")

-- Rain Amount
from(bucket: "AcuRite")
    |> range(start: -24h)
    |> filter(fn: (r) => r._field == "rain_in")
```
