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
sudo rtl_433 -R 40 -F "influx://localhost:9999/api/v2/write?org=<org>&bucket=<bucket>,token=<authtoken>"
```
