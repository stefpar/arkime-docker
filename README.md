# arkime-docker
Simple Docker Compose setup to quickly deploy a full Arkime stack for network monitoring and security analysis.


## Directory structure

```
|-- docker-compose.yml
|-- raw
|-- etc
|   |-- config.ini
|   |-- arkime.rules
```


---

## 🚀 Running

### 1️⃣ Configure Arkime
Before running, edit `etc/config.ini` and update:
- `passwordSecret`
- `interface` (Optional for live capture)
- Any other settings appropriate for your environment.

---

### 2️⃣ (Optional) Configure and run MaxMind GeoIP update

If you want GeoIP data, uncomment the `geoipupdate` service in `docker-compose.yml`  
and add your **Account ID** and **License Key** It's free to create yours.

Run it to download the GeoIP databases:

```bash
sudo docker compose up -d geoipupdate
[+] Running 2/2
 ✔ Network arkime-docker-main_internal-net  Created                                                                                                                                                   0.0s 
 ✔ Container geoipupdate                    Started   
```

### 3️⃣ Initialize OpenSearch (Only once)

```
sudo docker compose up -d opensearch
echo "INIT" | sudo docker run --rm -i --network arkime-docker-main_internal-net -v ./etc:/opt/arkime/etc ghcr.io/arkime/arkime/arkime:v5-latest /opt/arkime/db/db.pl http://opensearch:9200 init
```
Let it run for 15 seconds or so, and continue...

### 4️⃣ Start the full Arkime stack

```
sudo docker compose up -d
[+] Running 4/4
 ✔ Container geoipupdate     Running                                                                                                                                                                  0.0s 
 ✔ Container opensearch      Running                                                                                                                                                                  0.0s 
 ✔ Container arkime-viewer   Started                                                                                                                                                                  0.2s 
 ✔ Container arkime-capture  Started       
```

### 5️⃣ Create a default viewer user

```
sudo docker exec arkime-viewer /opt/arkime/bin/arkime_add_user.sh admin "Admin User" YOUR_PASSWORD --admin
```

You can now log in to the Arkime Viewer at http://localhost:8005

## 🧪 Import the Demo PCAP

A sample PCAP from https://www.malware-traffic-analysis.net/2024/07/30/index.html  is included.
Import it into Arkime with:

```
sudo docker exec arkime-capture  /opt/arkime/bin/capture -r /opt/arkime/raw/2024-07-30-traffic-analysis-exercise.pcap -t demo
```

You’ll find related sessions dated 2024-07-30 in the Viewer.
You can also upload your own PCAPs via the GUI or by placing them in the raw/ folder and running the same command.

## 🧠 Notes

- The raw/ folder stores captured and imported PCAPs.

- etc/config.ini controls both capture and viewer configuration.



