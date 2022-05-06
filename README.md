This is a fork of OSRM backend with project specific customizations for the smart transit project. Please refer to [OSRM-README.md](OSRM-README.md) for details about the OSRM project and the instructions for general compilation. 

Follow these instructions for the use of the router with smart transit project.

# Step 1: Setup

Clone `smarttransit-osrm-backend` and use `paratransit_software` branch.

Make sure you have docker and a tool that will crop OSM data [osm cropping tools](http://docs.opentripplanner.org/en/latest/Preparing-OSM/).
`osmconvert` is much faster than `osmosis` however `osmosis` is cross-platform and I could only get
`osmconvert` to work on linux.

To install osmosis with mac: `brew install osmosis`.

# Step 2: OSM Data

Download all US south data:

```bash
wget http://download.geofabrik.de/north-america/us-south-latest.osm.pbf
```

Crop the OSM to be a bounding box around the extended Tennessee region.

```bash
# if using osmconvert
osmconvert us-south-latest.osm.pbf -b=-90.9733,34.5341,-81.106,37.081 --complete-ways -o=tennessee-latest.osm.pbf
# or using osmosis
osmosis --rb us-south-latest.osm.pbf --bounding-box left=-90.9733 right=-81.106 bottom=34.5341 top=37.081 --wb tennessee-latest.osm.pbf
```

Move `tennessee-latest.osm.pbf` to `tn/` folder and delete `us-south-latest.osm.pdf`.

```bash
mkdir tn && mv tennessee-latest.osm.pbf tn/
rm us-south-latest.osm.pbf
```

# Step 3: Build an image and push to docker hub

```bash
# TAG can be something like tennessee-extended-20220506
docker build -f docker/smarttransitDockerfile -t mpwilbur/smarttransit-osrm-server:<TAG> . 

# push to docker hub
# Easiest way is to find the image in docker desktop and push from there
# TODO: if I have time, find the command line command to run
```

# Step 4: Pull and Run

```bash
docker pull mpwilbur/smarttransit-osrm-server:<TAG>
docker run -m=12g -p 8080:5000 -d --restart unless-stopped mpwilbur/smarttransit-osrm-server:<TAG>
```

After that we will have a docker container exposed on port 8080.

# Step 5: Test

```bash
# <remote-server> is the url/IP of the server you are running the OSRM instance (step 4)
curl "http://<remote-server>:8080/route/v1/driving/-86.79426670074463,36.12473806954196;-86.7641830444336,36.13808266878191"
curl "http://<remote-server>:8080/table/v1/driving/-86.79426670074463,36.12473806954196;-86.7641830444336,36.13808266878191"
```
