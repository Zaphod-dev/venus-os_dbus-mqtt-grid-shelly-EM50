# dbus-mqtt-grid-shelly-EM50 - Emulates a physical Grid/Genset/AC Load Meter from Shelly PRO EM50 CT sensors via MQTT data
This fork or Mr Manuel's repo (all credits to him for making this feasible!) essentially just remaps the Shelly out of the box MQQT payloads to the relevant Venus OS DBus data

### Disclaimer

I wrote this script for myself. I'm not responsible, if you damage something using my script.

### Purpose

The script emulates a physical Grid/Genset/AC Load Meter in Venus OS. It gets the Shelly Pro EM50 MQTT data from a subscribed topic and publishes the information on the dbus as the service `com.victronenergy.grid.mqtt_grid`, `com.victronenergy.genset.mqtt_genset` or `com.victronenergy.acload.mqtt_acload` with the VRM instance `31`.

### Config

Copy or rename the `config.sample.ini` to `config.ini` in the `dbus-mqtt-grid` folder and change it as you need it.

### JSON structure

<details><summary>Minimum required with just one CT clamp (L1)</summary>

```json
{
    "grid": {
        "power": 0.0,
        "L1": {
            "power": 0.0
        }
    }
}
```
</details>

Currently not implemented : import of the second CT clamp data (would be easy to do, but I have currently no use case for second clamp)

### Install

1. Login to your Venus OS device via SSH. See [Venus OS:Root Access](https://www.victronenergy.com/live/ccgx:root_access#root_access) for more details.

2. Execute this commands to download and extract the files:

    ```bash
    # change to temp folder
    cd /tmp

    # download driver
    wget -O /tmp/venus-os_dbus-mqtt-grid.zip https://github.com/mr-manuel/venus-os_dbus-mqtt-grid/archive/refs/heads/master.zip

    # If updating: cleanup old folder
    rm -rf /tmp/venus-os_dbus-mqtt-grid-master

    # unzip folder
    unzip venus-os_dbus-mqtt-grid.zip

    # If updating: backup existing config file
    mv /data/etc/dbus-mqtt-grid/config.ini /data/etc/dbus-mqtt-grid_config.ini

    # If updating: cleanup existing driver
    rm -rf /data/etc/dbus-mqtt-grid

    # copy files
    cp -R /tmp/venus-os_dbus-mqtt-grid-master/dbus-mqtt-grid/ /data/etc/

    # If updating: restore existing config file
    mv /data/etc/dbus-mqtt-grid_config.ini /data/etc/dbus-mqtt-grid/config.ini
    ```

3. Copy the sample config file, if you are installing the driver for the first time and edit it to your needs.

    ```bash
    # copy default config file
    cp /data/etc/dbus-mqtt-grid/config.sample.ini /data/etc/dbus-mqtt-grid/config.ini

    # edit the config file with nano
    nano /data/etc/dbus-mqtt-grid/config.ini
    ```

4. Run `bash /data/etc/dbus-mqtt-grid/install.sh` to install the driver as service.

   The daemon-tools should start this service automatically within seconds.

### Uninstall

Run `/data/etc/dbus-mqtt-grid/uninstall.sh`

### Restart

Run `/data/etc/dbus-mqtt-grid/restart.sh`

### Debugging

The logs can be checked with `tail -n 100 -F /data/log/dbus-mqtt-grid/current | tai64nlocal`

The service status can be checked with svstat `svstat /service/dbus-mqtt-grid`

This will output somethink like `/service/dbus-mqtt-grid: up (pid 5845) 185 seconds`

If the seconds are under 5 then the service crashes and gets restarted all the time. If you do not see anything in the logs you can increase the log level in `/data/etc/dbus-mqtt-grid/dbus-mqtt-grid.py` by changing `level=logging.WARNING` to `level=logging.INFO` or `level=logging.DEBUG`

If the script stops with the message `dbus.exceptions.NameExistsException: Bus name already exists: com.victronenergy.grid.mqtt_grid"` it means that the service is still running or another service is using that bus name.

### Multiple instances

It's possible to have multiple instances, but it's not automated. Follow these steps to achieve this:

1. Save the new name to a variable `driverclone=dbus-mqtt-grid-2`

2. Copy current folder `cp -r /data/etc/dbus-mqtt-grid/ /data/etc/$driverclone/`

3. Rename the main script `mv /data/etc/$driverclone/dbus-mqtt-grid.py /data/etc/$driverclone/$driverclone.py`

4. Fix the script references for service and log
    ```
    sed -i 's:dbus-mqtt-grid:'$driverclone':g' /data/etc/$driverclone/service/run
    sed -i 's:dbus-mqtt-grid:'$driverclone':g' /data/etc/$driverclone/service/log/run
    ```

5. Change the `device_name`, increase the `device_instance` and update the `topic` in the `config.ini`

Now you can install and run the cloned driver. Should you need another instance just increase the number in step 1 and repeat all steps.

### Compatibility

It was tested on Venus OS Large `v2.92` on the following devices:

* RaspberryPi 4b
* MultiPlus II (GX Version)

### Screenshots

<details><summary>Power and/or L1</summary>

![Grid power L1 - pages](/screenshots/grid_power_L1_pages.png)
![Grid power L1 - device list](/screenshots/grid_power_L1_device-list.png)
![Grid power L1 - device list - mqtt grid 1](/screenshots/grid_power_L1_device-list_mqtt-grid-1.png)
![Grid power L1 - device list - mqtt grid 2](/screenshots/grid_power_L1_device-list_mqtt-grid-2.png)

</details>

<details><summary>Power, L1 and L2</summary>

![Grid power L1, L2 - pages](/screenshots/grid_power_L2_L1_pages.png)
![Grid power L1, L2 - device list](/screenshots/grid_power_L2_L1_device-list.png)
![Grid power L1, L2 - device list - mqtt grid 1](/screenshots/grid_power_L2_L1_device-list_mqtt-grid-1.png)
![Grid power L1, L2 - device list - mqtt grid 2](/screenshots/grid_power_L2_L1_device-list_mqtt-grid-2.png)

</details>

<details><summary>Power, L1, L2 and L3</summary>

![Grid power L1, L2, L3 - pages](/screenshots/grid_power_L3_L2_L1_pages.png)
![Grid power L1, L2, L3 - device list](/screenshots/grid_power_L3_L2_L1_device-list.png)
![Grid power L1, L2, L3 - device list - mqtt grid 1](/screenshots/grid_power_L3_L2_L1_device-list_mqtt-grid-1.png)
![Grid power L1, L2, L3 - device list - mqtt grid 2](/screenshots/grid_power_L3_L2_L1_device-list_mqtt-grid-2.png)

</details>
