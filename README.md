# wmbusmeters importer into influxDb

> This are bash scripts to install an realtime importer for [wmbusmeters][1] data
> into [InfluxDB][2] time series database.

## 1 Features

* Importer for manual import in InfluxDB
* Script for watching wmbusmeters data directory and import new data in real time
* systemd service file for start watch daemon
* logrotate config file

## 2 Usage

### 2.1 Installation of the influxImport script

Copy the sript `influxImport` in a directory like `/usr/local/sbin`

Configure the variables in the top of the script.

Most probably you need to alter the `function formatInfluxData()`, for your needs if your wmbusmeters json data has another structure.
Also adjust the `echo` command at the end of this function for your influxDB write command. 
A desription of the syntax you can find [here][4]. 

Please read before carefully the advices for your InfluxDB [schema layout][3]. 

Test an import of a wmbusmeters file with the followning command

```
influxImport <filename>
```

### 2.2 Installation of the watchNewData script

Copy the sript in a directory like `/usr/local/sbin/` 

Create all the necessary directories under the working directory of wmbusmeters

	.
 	└── wmbusbeterdirectory				# The directory where wmbusmerters is writing the meter data
		├── imported
		├── process
		└── error  

Create a directory for the logs in `/var/log/influxImport/` 

Make all the directorys configured in this script writeable for wmbusmeters

```
chown wmbusmeters.wmbustertes <directory>
```

Configure the vairables in top of the script `watchNewData`.

Start the script first time with the variable `VERBOS=true` and let wmbusmeters write new data. Watch if the import works properly.

```
watchNewData <directory with the wmbusmeters data>
```

### 2.3 Install systemd service

Copy the `wmbusmeter-import.service` service file into the directory `/etc/systemd/system/`

Set `WorkingDirectory=` to the directory with the wmbusmeters data.

Write into `ExecStart=` the command with the watchNewData script. Append an ampersand `&` at the end of this command.

Restart systemd.

```
systemctl daemon-reload
```

Enable the service.

```
systemctl enable wmbusmeter-import
```

Start the service

```
systemctl start wmbusmeter-import
```

Look if everything works fine.

```
systemctl status wmbusmeter-import
```

### 2.3 configure logrotate

Copy and rename the file `influxImport.logrotate` to `/etc/logrotate.d/influxImport`

Enter the name of the directoy from the `LOG_FILE` variable in script `watchNewData` at the top of this file.

Test the configuration.

```
logrotate -f /etc/logrotate.d/influxImport
```


[1]: https://github.com/weetmuts/wmbusmeters
[2]: https://www.influxdata.com/
[3]: https://archive.docs.influxdata.com/influxdb/v1.2/concepts/schema_and_data_layout/
[4]: https://docs.influxdata.com/influxdb/v2.1/write-data/developer-tools/influx-cli/