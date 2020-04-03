# grAfterburner

<p align="center"><a href="https://grafana.com"><img src="https://i.imgur.com/YSRD3li.png" width="100"/></a> <img src="https://i.imgur.com/TeISZNZ.png" vspace=20 width="50"/> <a href="https://www.msi.com/page/afterburner"><img src="https://i.imgur.com/6dY1eUu.png" width="105"/></a> <br>Monitor your PC in style with Grafana and MSI Afterburner</p>

![Dashboard Screenshot](https://i.imgur.com/vTExX20.jpg)

### Why?

- A lot of people already use MSI Afterburner for it's GPU overclocking functionality, as well as coming with the excellent RTSS for frametime monitoring and on-screen-display. It also adds negligible load on your CPU.
- Since I have a single monitor set-up, I wanted something I could display on my [Raspberry Pi 3B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) which I had a small [3.5" Display](https://learn.adafruit.com/adafruit-pitft-3-dot-5-touch-screen-for-raspberry-pi/overview) for, and I did not want to run yet another monitoring software just for that. Also, very few software can monitor the frame-time during games.
- As MSI Afterburner has an additional Remote Server component, I wanted to find a way to get that displayed in a nice dashboard, which led me to this.
- [Interactive Snapshot Demo](https://snapshot.raintank.io/dashboard/snapshot/Z5mCjhrfHn8bTYr7Ld3YgbRoNAvbl6We) (not 100% accurate as some features are not supported)
- The Grafana dashboard can be found [here](https://grafana.com/grafana/dashboards/12021) to import into your existing instance.

### Components

- [MSI Afterburner](https://www.msi.com/page/afterburner): provides a detailed overview of your hardware, and also allows graphics card overclocking. Includes [RTSS](https://www.guru3d.com/files-details/rtss-rivatuner-statistics-server-download.html) which also provides an on-screen-display during games.
- [MSI Afterburner Remote Server](https://www.msi.com/page/afterburner): serves up an HTTP endpoint with data from MSI Afterburner in an XML format.
- [collectd](https://collectd.org/): a daemon which collects metrics periodically and provides mechanisms to store the values in a variety of ways. Used to perform HTTP requests to the MSI Afterburner endpoint and store the data in a database.
- [Graphite](https://graphiteapp.org/): a monitoring tool that stores numeric time-series data and provides an API to access it. Stores all the data from collectd.
- [Grafana](https://grafana.com/): allows you to query, visualize, alert on and understand your metrics and create your own dashboards. Visualizes the data stored in Graphite.
- [Docker](https://www.docker.com/): tool for building and running applications in containers. Allows you to start up collectd, Graphite and Grafana in one command without needing to think about dependencies or configurations.

### Requirements

- For the PC you want monitored, you will need these (Windows only)
  1. [MSI Afterburner](https://www.msi.com/page/afterburner) (it will ask you about RTSS as well during installation which you need for FPS / Frametime information)
  2. [MSI Afterburner Remote Server](https://www.msi.com/page/afterburner) (does not install but just has an executable)
- For the monitoring setup (can be on the same PC or a separate one, Windows / macOS / Linux)
  1. [Docker](https://docs.docker.com/install/) (and [Docker Compose](<[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)>))

### Installation

1.  Download and install the requirements listed in the above section and make sure that they are running.
2.  [Clone](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository) [this repository](https://github.com/RafhaanShah/grAfterburner.git) on the PC that you want the monitoring setup to run on (the one with Docker) or [download it as a zip file](https://github.com/RafhaanShah/grAfterburner/archive/master.zip) and extract it to a folder.

### Configuration

1. **MSI Afterburner:**
   - Open MSI Afterburner by clicking the tray icon in the bottom right of Windows
   - Go into Settings -> Monitoring
   - Press the check mark next to every metric that you want monitored (e.g.: GPU Usage)
   - Press OK
2. **MSI Afterburner Remote Server:**
   - This application only has a tray icon so right click it to view the options
   - Go into security and set a password of your choice
   - Go into HTTP Listener and change the port if you wish to change the default of 82
   - Also note the IP address it shows, you will need it in the next step
   - Press restart server when you are done
3. **Monitoring Configuration:**
   - Open the cloned repository folder
   - There is a _collectd.conf_ file inside the collectd folder, open this in a text editor
   - Edit line 14 to be the [local IP](https://lifehacker.com/how-to-find-your-local-and-external-ip-address-5833108) address and the port of your PC (the one with MSI Afterburner running) which you configured in the previous step, e.g.: if my PC's IP address is 192.168.1.10 and I set port 82, the line should be:
     ```
     <URL "http://192.168.1.10:82/mahm">
     ```
     (You can also use your PC's hostname instead of the IP if you have local hostname resolution working on your network)
   - Edit line 19 to have the password you set in step 3, e.g.: if my password is "abcd1234" the line should be:
     ```
     Password "abcd1234"
     ```
   - Save and close the file
   - Additional step for Raspberry Pi's / ARM based computers ONLY - edit line 21 of _docker-compose.yaml_ to be:
     ```
     image: "grafana/grafana-arm32v7-linux:latest"
     ```
4. **Optional Configurations:**
   - edit line 25 of _docker-compose.yaml_ to change the port Grafana runs on from 8025 to something else (leave the 3000)
   - remove line 29-32 of _docker-compose.yaml_ if you want to enable login with password for Grafana
   - If you do not want the services to start on the boot-up of your system remove the 3 lines in _docker-compose.yaml_ that say _'restart: always'_ (line 7, 22, 39)
   - If you wish to change how often MSI Afterburner polls your hardware for new data, open MSI Afterburner, Settings -> Monitoring -> Hardware Polling Period.
   - You will probably also want to change how often collectd fetches new data as well, so open _collectd.conf_ and edit line 11 and 15, further reading in the [collectd docs](https://collectd.org/documentation/manpages/collectd.conf.5.shtml).
   - If you changed the collectd fetch frequency, you should also edit _graphite/conf/storage-schemas.conf_ and edit line 34, more info in the [Graphite docs](https://graphite.readthedocs.io/en/latest/config-carbon.html#storage-schemas-conf).
   - Various Grafana configuration options can be changed through [environment variables](https://grafana.com/docs/grafana/latest/installation/configuration/#configure-with-environment-variables) in the docker-compose file.

### Running

1.  Open command prompt / powershell / any terminal in the repository folder
2.  Run this command to start up the monitoring services:
    ```
    docker-compose up -d
    ```
3.  To open the Grafana and view the dashboard, you will need to know the local IP address of the computer running the services, e.g.: if the computer I am running the services on has a local IP address of 192.168.1.20, then I will need to type into my web browser:
    ```
    http://192.168.1.20:8025
    ```
    (if you changed the port Grafana runs on, then you will need to replace 8025 with that port)
4.  The dashboard and data source will already be set up but you can modify all of it to suit your needs. Auto refresh and time range options are in the top right corner, and more information on dashboard can be found [here](https://grafana.com/docs/grafana/latest/features/dashboard/dashboards/).
5.  If you wish to stop the services use the following command:
    ```
    docker-compose stop
    ```

### Notes

- I'd recommend running the monitoring on a different PC to your main one, so as to not affect it's performance.
- The number of graphs and the refresh rate will affect the amount of CPU usage by the Graphite docker container.
- SD cards used in Raspberry Pi's are not really meant to be [constantly written](https://domoticproject.com/extending-life-raspberry-pi-sd-card/) to, and even though Graphite is very efficient with it's storage, you may want to look into booting off an SSD or setting the Graphite storage [volume](https://docs.docker.com/compose/compose-file/#volumes) (line 12 of _docker-compose.yaml_) to be on an external drive.
- If you are using a small screen (like a 3.5" Raspberry Pi Screen), you may need to adjust the zoom level of the dashboard in your browser to be able to fit more graphs at once.

### Troubleshooting

- If the docker commands do not work, double check that you are inside the repository folder (the one with the _docker-compose.yaml_ file in it) in your terminal.
- Verify MSI Afterburner Remote Server is working: go to _http://192.168.1.10:82/mahm_ (where 192.168.1.10 is the IP address of your PC running MSI Afterburner) in a web browser (try it on a browser that is not on the PC you are running MSI Afterburner on). You should get a pop-up asking for a username and password, use MSIAfterburner and the password you set, and verify that there is XML data visible, like [this](https://gist.github.com/joar/2a91ff54718e58011f86).
- If your PC does not have a static local IP, you should [set it](https://www.howtogeek.com/howto/19249/how-to-assign-a-static-ip-address-in-xp-vista-or-windows-7/) to have one so that the _collectd.conf_ will not point to the wrong IP every time your PC gets a new local IP address.
- Check if collectd is storing anything in Graphite: un-comment lines 8 and 9 of _docker-compose.yaml_ (double check your indentation [here](http://www.yamllint.com/)) and run _docker-compose up -d_ again. Visit _http://192.168.1.20:8026_ (where 192.168.1.20 is the local IP address of the PC you are running the monitoring services on). You should see the web interface for Graphite, on the left menu click metrics, you should see _collectd._ Clicking through it should show _pc_ -> _curl-xml-afterburner_ -> and the monitors you enabled.
- If there is no data, double check your configuration in _collectd.conf_, you can also try changing the _'udp'_ on line 40 to _'tcp'_.
- Verify the data source in Grafana: click the cog on the left for settings. On the data sources page there should be an entry for _Graphite_. Click on this and click save and test. It should say '_Data source is working_'. If not, then it is having a problem connecting to Graphite, check that the Graphite container is running and the URL is _http://grafterburner-graphite:80_.
- Check the monitored metrics you set up in MSI Afterburner during step 1, the default provided dashboard uses: GPU temperature, GPU usage, Memory usage, Core clock, Memory clock, GPU voltage, Fan speed, Fan speed 2, Fan tachometer, Fan tachometer 2, Temp limit, Power limit, Voltage limit, No load limit. CPU temperature, CPU usage, CPU clock, CPU power, RAM usage, Framerate, Frametime. You do not need to have all these, and you can make your own dashboards, but the one that is already set up uses these metrics.
- Check the [logs](https://docs.docker.com/config/containers/logging/) of the docker containers to see if there are any log messages that may help.

### Acknowledgments

- The awesome [gamergraf](https://github.com/ragesaq/gamergraf) inspired a lot of this, and a huge credit to the project for giving me a lot of ideas and some of the configuration for things like collectd's curl_xml plugin.
- [StackeEdit.io](https://stackedit.io/app#) for great MarkDown editing.
- [PNGGuru](https://www.pngguru.com/) for lovely free PNG icons.
- [Imgur](https://imgur.com/) for hosting the screenshots / images for free.

![Raspberry Pi Dashboard](https://i.imgur.com/UpnLKZh.jpg)
