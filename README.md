# Telegraf, InfluxDB, Grafana Labs

## Lab videos: [YouTube Playlist](https://www.youtube.com/playlist?list=PLGmrb_nBLueKbk3tV5gwI7FxmXwBpGak0)

## Lab 1: Install Docker

`sudo apt update`

`sudo apt -y upgrade`

`sudo apt install docker`

## Lab 2: Docker nonroot access and start on boot

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

`sudo systemctl enable docker`

*Logout completely to take effect.

## Lab 3: Install and run InfluxDB

`sudo useradd -rs /bin/false influxdb`

`sudo mkdir -p /etc/influxdb`

`sudo chown influxdb:influxdb /etc/influxdb/`

`docker run --rm influxdb influxd config | sudo tee /etc/influxdb/influxdb.conf > /dev/null`

`sudo vim /etc/influxdb/influxdb.conf`

In the [http] session, change "auth-enabled" to "true" (without quote).
Save and exit.

`docker network create influxdb`

`docker volume create influxdb-data`

`docker run -dit --restart always -p 8086:8086 --name=influxdb --net=influxdb -v influxdb-data:/var/lib/influxdb -v /etc/influxdb/influxdb.conf:/etc/influxdb/influxdb.conf influxdb`

`docker exec -it influxdb bash`

`influx`

`create user admin with password 'influxdb' with all privileges;`

`exit`

`exit`

`curl -G -u admin:influxdb http://localhost:8086/query --data-urlencode "q=SHOW DATABASES"`

You should see the successful response.

## Lab 4: Install Telegraf and fetch configuration

`curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -`

`source /etc/lsb-release`

`echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list`

`sudo apt update`

`sudo apt install telegraf`

`telegraf -sample-config -input-filter cpu:mem:disk:kernel:processes:swap:system:net:ping -output-filter influxdb > telegraf.conf`

## Lab 5: Copy Telegraf configuration file to /etc

`sudo cp telegraf.conf /etc/telegraf/`

`sudo cp *snmp.conf /etc/telegraf/telegraf.d`

`sudo cp vsphere.conf /etc/telegraf/telegraf.d`

## Lab 6: Edit Telegraf configuration file

`sudo vim /etc/telegraf/telegraf.conf`

Edit the following parameters:
1. interval

[[inputs.ping]]
1. urls
2. count
3. timeout
4. ....

[[outputs.influxdb]]
1. urls = ["http://localhost:8086"]
2. database
3. username
4. password
5. insecure_skip_verify = true
6. influx_unit_support = false

## Lab 7: Run Telegraf

`sudo systemctl start telegraf`

`sudo systemctl status telegraf`

`docker exec -it influxdb bash`

`influx -username admin -password influxdb`

`show databases;`

`use telegraf;`

`show measurements;`

Check if the data has been inserted into the database.

## Lab 8: Retreive Grafana configuration

`docker volume create grafana-data`

`docker run --rm --name=grafana grafana/grafana:6.4.4`

Start a new terminal

`mkdir grafana-conf`

`cd grafana-conf`

`docker cp grafana:etc/grafana/grafana.ini .`

`docker cp grafana:etc/grafana/ldap.toml .`

`docker cp grafana:etc/grafana/provisioning .`

`sudo useradd -rs /bin/false grafana`

`sudo mkdir -p /etc/grafana`

`sudo cp -R * /etc/grafana`

`sudo chown -R grafana:grafana /etc/grafana`

Press Ctrl-C to exit the "grafana" container.

## Lab 9: Run Grafana

`docker run -dit --restart always -p 3000:3000 --name=grafana --net=influxdb -v grafana-data:/var/lib/grafana -v /etc/grafana:/etc/grafana grafana/grafana:6.4.4`

Start web browser, go to "http://localhost:3000"

Use "admin/admin" as the first time username and password and then change the password.

## Lab 10: Add the first datasource, dashboard and panel in Grafana

Please see the video for the detail.

## Lab 11: Add another data source and panel

Please see the video for the detail.

## Lab 12: Set up alert channel of Grafana (using LINE)

Please see the video for the detail.

## Lab 13: Upgrade Grafana

`docker container stop grafana`

`docker container rename grafana grafana-6.4.4`

`docker run -dit --restart always -p 3000:3000 --name=grafana --net=influxdb -v grafana-data:/var/lib/grafana -v /etc/grafana:/etc/grafana grafana/grafana:x.x.x`

Only delete containers and images after making sure the new container works as expect.

`docker rmi grafana/grafana:6.4.4`

`docker rm grafana-6.4.4`

## Lab 14: Run Grafana image renderer

`docker run -dit -p 8081:8081 --name=grafana-image --net=influxdb grafana/grafana-image-renderer:latest`

## Lab 15: Create Google Storage Bucket

`sudo mkdir -p /etc/grafana/key`

`sudo cp *.json /etc/grafana/key`

## Lab 16: Edit Grafana.ini setting to support image rendering

`sudo vi /etc/grafana/grafana.ini`

```
        [external_image_storage]
        provider = gcs

        [external_image_storage.gcs]
        key_file = /etc/grafana/key/ringed-XXXXXXX.json
        bucket = your-bucket-name

        [rendering]
        server_url = http://grafana-image:8081/render
        callback_url = http://grafana:3000/
```

Restart containers:grafana, grafana-image

## Lab 17: Add Email notification channel
