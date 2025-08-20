# Fully managed Home Energy Management System (HEMS)

This system was designed and built by Carbon Co-op to provide energy communities with a secure and private in-home device for data gathering and high-load device control. The project is primarily aimed at the following usecases: 

* **Building performance/retrofit evaluation**
While smart meter data is useful, the addition of local sensor data allows for the separation of the impact of fabric, HVAC and control measures and can account for changes in user behaviour, solar generation and electric vehicles charging. A fully managed HEMS allows us to provide monitoring to meet householder or project requirements, even with non-techical users via fleet deployment and remote support.


* **Demand Side Flexiblity**
The HEMS forms the endpoint for community 'demand side response' schemes, such as our [PowerShaper Flex service](https://flex.powershaper.io/), turning high load domestic devices up and down in order to respond to needs of a renewable rich power grid.

This version of the HEMS has been built from experience of previous systems. In particular the original [HEMS](https://gitlab.com/carboncoop/hems) developed by Carbon Co-op, the [Cofybox](https://gitlab.com/rescoopvpp/open-cofybox) developed by Carbon Co-op alongside other REScoop co-ops during the VPP project and also taking inspiration from the Balena labs [Home Assistant](https://github.com/balena-io-experimental/balena-homeassistant) project.

# HEMS System, Services and functionality
![Diagram showing each service running on the Balena OS and a description of its functionality.](https://cc-site-media.s3.amazonaws.com/uploads/2024/05/HEMS-services-diagram.png)

# Home Assistant and supporting containers.

* [**Home Assistant**](https://www.home-assistant.io/) - tracks the final version of the previous months release for stability.
* [**MQTT**](https://mosquitto.org/) - Mosquitto MQTT broker.
* [**InfluxDB**](https://www.influxdata.com/) - a comprehensive time-series database. We are using the 2.x branch in order to utilise the edge replication functionality to influxcloud, which brings the advantage of automated gathering of device load information, feed loss tracking and local caching (for later replication). 
* [**NGINX reverse proxy**](https://www.nginx.com/) - responsible for routing traffic when connecting remotely
* [**Hass configurator**](https://github.com/danielperna84/hass-configurator) - a tool for editing Home Assistant configuration
* [**Wifi repeater**](https://github.com/balena-labs-projects/wifi-repeater) - a tool for creating a wifi hotspot for device to connect to
* [**System manager**](https://gitlab.com/carboncoop/hems-2/-/tree/main/services/system-manager?ref_type=heads) - a container to handle system configuration and monitoring

## Software setup

Running this project is as simple as deploying it to a balenaCloud application, then downloading the OS image from the dashboard and flashing your SD card.

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/carboncoop/hems-test)

We recommend this button as the de-facto method for deploying new apps on balenaCloud, especially if you are just getting started or want to test out the project. However, if you want to modify the docker-compose or tinker with the code, you'll need to clone this repo and use the [balenaCLI](https://github.com/balena-io/balena-cli) to push to your devices. This can be done later if you initially deploy using the button above. [Read more](https://www.balena.io/docs/learn/deploy/deployment/).

### First login
To access your new Home Assistant instance, ensure you are on the same network as the device and browse to the IP address of your device (which you can find on your balenaCloud dashboard). Go through the Home Assistant setup process and establish a username and password. To obtain a secure public URL for your Home Assistant instance, simply click the "Public Device URL" switch on your balenaCloud dashboard. You'll then see a link to access your unique device URL.

### MQTT Broker use
The MQTT container provides a local open-acess broker (which is distinct from any Home Assistant integration). You can use this broker to connect local devices and add it to HA.

The access point is `mqtt://$your-local-ip:1883`. There is no credential needed.

### InfluxDB use
The InfluxDB container provides a dedicated timeseries database. It is not yet configured automatically, however this is important for caching of data, because Home Assistant keeps limited history. Here are a few steps to setup the local influx storage:
1. From the balena console, run this command on the InfluxDB container (with freely chosen new credentials): 
```
influx setup \
  --org hems \
  --bucket home_assistant \
  --username <USERNAME> \
  --password <PASSWORD> \
  --force
```
2. Find the bucket_id of the bucket you just created, with `influx bucket list`
3. Create authentication for HomeAssistant to write to the local influx, with
 `influx auth create --write-bucket <BUCKET_ID> --write-remotes --write-replications`
with the <BUCKET_ID> you just read. 
4. This command will give you an _API token_. Set it in a new environment variable (in the balena console) with the name : INFLUX_TOKEN.
5. Righ upon reload, Home Assistant will start reporting to the local influx database :slight_smile:


To test if you have data in influx, you can use this command in the influxdb container:
```
influx query 'from(bucket: "home_assistant") |> range(start: -1h)'
```
## File Locations
The project's _docker-compose_ file creates persistent volumes on your disk/SD card for storing configuration and data. In particular, the entire Home Assistant configuration is persistently stored in a volume mapped to location `/config`.

## Configuring Home Assistant
Generally we recommend UI configuration from the Home Assistant standard UI, but if you have particular requirements that mean you need to edit `yaml` files, a text editor called _Hass-Configurator_ is available locally on port `3218`. (To access this while on the same network as your device go to http://%your-ip-address%:3218).

 Using this editor, you can make changes to the Home Assistant configuration file `configuration.yaml` which is the in the default folder `hass-config` for Hass-Configurator. (`hass-config` is mapped to `/config`)

In order to access the configurator remotely, navigate to `https://<BALENA_URL>/configurator`. Note that this page is password protected. In order to set the user and password, create Balena variables `CONFIG_USER` and `CONFIG_PASSWORD`.


# Managing Home Assistant Community Custom Components

* **Community components** - The Home Assistant Community Store comes pre-installed on the HEMS 2 and provides much simpler installation, and most importantly management of updates for custom components. Moving away from fleet-wide deployment of custom components will free us up in terms of what devices we can support. We have already installed HACS on the ABS test fleet boxes and it's working very well.

# Wifi hotspot

The Wifi Repeater service sets up a wireless access point on the device when it boots. This can be used to connect devices in the home to the HEMS without relying on the home network. 

:warning: Note: this hotspot comes with default credentials

| Env var | Description | Default |
| ------------- | ------------- | ------------- |
| AP_SSID | Access point network name. | `WiFi Repeater` |
| AP_PASSWORD | Access point network password. | `charlietheunicorn` |

and should be considered insecure in this state.

To change these credentials, set the environment variables `AP_SSID` and `AP_PASSWORD` in your Balena configuration.


# Configuration considerations

## Hardware required

* Raspberry Pi 4B - 2GB or more
* 64GB Micro-SD Card (with wear levelling)
* Power supply and cable
* Optional: For connecting Zigbee devices - [SONOFF 3.0 dongle plus](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/) or [HA Skyconnect](https://www.home-assistant.io/skyconnect/) - for future MATTER compatibility (note: multi-protocol support is only available currently in HA OS version).
