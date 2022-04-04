# smarthomeserver
A simple setup for your own smart home server (on a Raspberry Pi)

## Why you need your own smart home server
The smart home market is really fragmented. Using different gateways and Apps from different ecosystems can be annoying, expansive and might not work very well together. Instead I suggest using a custom smart home server with some open source software that can replace all your hubs and give you access to one system to control it all.

## The hardware
I am running this on a Raspberry Pi but as these are Docker containers you can run them on pretty much any platform.

## How to start
Create a folder to hold all your docker data. Then clone this repository and update the .env file. Change the password and IDs and update the path to the folder you just created.

Be sure to update the email related settings if you want to use notifications for automatic container updates. If you have done that uncomment the `WATCHTOWER_NOTIFICATIONS` related variables in the hosting.yml file.

If you want to use Loki/Grafana to see all logs in one place you need to also copy the loki-configuration.yaml file to `${DATADIR}/loki/config/loki-config.yaml`. You need to also install the Loki logging driver or remove the logging commands from the compose files. The installation can be done via
```
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
// or if you are running this on a Raspberry Pi
docker plugin install grafana/loki-docker-driver:arm-v7 --alias loki --grant-all-permissions
```
Alternatively if you are going to use promtail to ingest the logs you need to also copy the promtail-config.yaml file to `${DATADIR}/promtail/config/promtail-config.yaml` and remove the `logging:` parts in the compose yaml files.

Then you can start the containers via docker compose.
```
docker-compose -f hosting.yml up -d
docker-compose -f smarthome.yml up -d

// to see logs (some will only be available via Grafana, see below)
docker-compose -f ...yml logs -f

// to stop
docker-compose -f ...yml down
```

## Which services are included?

In the smarthome.yml:

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| Mosquitto  | 1883  | You need to copy the config file above. Can be accessed with a MQTT client like MQTT explorer |
| InfluxDB  | only internally available from other containers  | - |
| Grafana | 3000  | Setup can be done according to my [Grafana dashboard guide](https://thesmarthomejourney.com/2020/07/20/smart-home-dashboards-grafana/). You can use this to view logs according to [the Loki guide](https://thesmarthomejourney.com/2021/08/23/loki-grafana-log-aggregation/) |
| TasmoAdmin  | 3080  | just let it scan your network for devices |
| Zigbee2MQTT  | -  | Setup can be done according to my [Zigbee2MQTT guide](https://thesmarthomejourney.com/2020/05/11/guide-zigbee2mqtt/) |
| Zigbee2MQTTAssistant  | 8880  | - |
| HomeAssistant  | 8123  | Just go to the webpage and follow the setup wizard |

This assumes that you have a Zigbee to USB stick connected to /dev/ttyACM0. Otherwise you need to update one line in the Zigbee2MQTT part. You will also need to provide a config file for mosquitto in the `${DATADIR}/mosquitto/config` folder. There is an example config here in the repo.

In the hosting.yml:

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| Heimdall  | 9080  | - |
| PiHole  | 6080  | There is a nice in-depth guide [here](https://www.smarthomebeginner.com/pi-hole-setup-guide/) |
| Adguard Home | 3380 | You can follow my setup guide [here](https://thesmarthomejourney.com/2021/05/24/adguard-pihole-dns-ad-blocker/)|
| Unifi controller | 8080  | Just follow the setup wizard |
| Watchtower | - | This is set up according to my [Watchtower guide](https://thesmarthomejourney.com/2021/03/01/watchtower-docker-auto-updates/) |
| Loki | 3100 | This is set up according to my [Loki guide](https://thesmarthomejourney.com/2021/08/23/loki-grafana-log-aggregation/) |
| Duplicati | 8200 | This allows you to back up any data from your Docker containers. More details [here](https://thesmarthomejourney.com/2022/04/04/home-assistant-docker-backup/) |

You should only use one adblocker (Adguard Home or PiHole) at a time as they use the same ports.

## Logging
If something is not working, check the logs first! Some service logs can only be viewed directly via `docker logs containername` or `docker-compose -f yamlname.yaml logs`. The important services are pushing their logs to Loki which collects all of them. You can use Grafana to view them all. I describe this in more detail [here](https://thesmarthomejourney.com/2021/08/23/loki-grafana-log-aggregation/). You need to install the Loki logging driver (see installation part above) for this to work or slightly change the compose files by removing the custom logging sections.

## How does it look like? I need more details
You can find more images and a details in my [blog post](https://thesmarthomejourney.com/2021/01/09/custom-smart-home-server-hub/)
