# dockerized-apps
The acmeair sample was run using the git repository at git@github.com:mpirvu/dockerized-apps.git. The test environment runs the liberty built with local jdk in one container, mongo db in another container and then run the JMeter tests in another third container. 


1. #### Build image with the local jdk build
cd /home/dev/openj9-openjdk-jdk11/build/linux-x86_64-normal-server-release/images \
rm openjdk.tar.gz \
tar -czvf openjdk.tar.gz jdk \

The doccker file for building a jdk image sample can be looked up from https://hub.docker.com/_/open-liberty. Open for ex. the tag `full-java11-openj9` and edit the file, add few lines and add a couple. Comment all the lines that downloads the semeru java and instead copies the local jdk to build the image with command `COPY openjdk.tar.gz /tmp/openjdk.tar.gz`

docker build -f ./DockerFile . -t myjdk:ccfix --no-cache | tee dockerBuild1.log \

<p>
docker images

	REPOSITORY        TAG                   IMAGE ID       CREATED         SIZE \
	myjdk             ccfix                 10d22601df23   2 minutes ago   1.95GB \
</p>

2. #### Build an image for open Liberty with the local jdk build

The liberty image sample can be looked from here: https://github.com/openliberty/ci.docker

Its better to use a full liberty image than the slim version. But if the slim version is being used then include the `RUN features.sh`.
This script will add the requested XML snippets to enable Liberty features and grow image to be fit-for-purpose using featureUtility. Only available in 'kernel-slim'. The 'full' tag already includes all features for convenience.

Modify the docker file to pick the jdk image built in the previous step using `FROM myjdk:ccfix`.

cd /home/dev/dockerized-apps/ci.docker/releases/23.0.0.5/full \
docker build -f ./Dockerfile.ubuntu.openjdk11_saj . -t open-liberty:myjdk-openj9 --no-cache | tee dockerLibertyBuild1.log \

3. #### Build the Mongo db image for the testing

cd /home/dev/dockerized-apps/AcmeAirEE8 \
cd MongoContext \
./buildMongo.sh \

<p>
docker images

	REPOSITORY          TAG                   IMAGE ID       CREATED          SIZE \
	open-liberty        myjdk-openj9          6f1b4cda4c7b   35 seconds ago   1.98GB \
	mongo-acmeair-ee8   5.0.15                3a7954332034   24 minutes ago   659MB \
	myjdk               ccfix                 10d22601df23   34 minutes ago   1.95GB 
</p>

4. #### Build the acmeair liberty sample using the liberty image we built

cd /home/dev/dockerized-apps/AcmeAirEE8 \
cd LibertyContext \
./build_acmeair_saj.sh \

5. #### Build the JMeter image
cd .. \
cd JMeterContext \

./build_jmeter.sh  \

<p>
	docker images

	REPOSITORY            TAG                   IMAGE ID       CREATED          SIZE \
	jmeter-acmeair        5.3                   f69defb1f51f   2 minutes ago    393MB \
	liberty-acmeair-ee8   J17                   4173cb4edfb0   6 minutes ago    1.99GB \
	open-liberty          myjdk-openj9          6f1b4cda4c7b   16 minutes ago   1.98GB \
	mongo-acmeair-ee8     5.0.15                3a7954332034   40 minutes ago   659MB \
	myjdk                 ccfix                 10d22601df23   50 minutes ago   1.95GB \
</p>

6. #### Start the Liberty AcmeAir sample

cd .. \
./startAcmeAir_saj.sh \

7. #### Start the JMeter tests in a container.

In another terminal \
cd /home/dev/dockerized-apps/AcmeAirEE8 \
./runJMeter.sh  \

In another terminal \

<p>
docker ps

    CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                              NAMES \
    51dcac8a2523   liberty-acmeair-ee8:J17    "/opt/ol/helpers/run…"   2 minutes ago   Up 2 minutes   0.0.0.0:9080->9080/tcp, 9443/tcp   acmeair \
    ae5830e4e001   mongo-acmeair-ee8:5.0.15   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   27017/tcp                          mongodb \
    27a34103cc80   registry:2                 "/entrypoint.sh /etc…"   3 weeks ago     Up 12 days     0.0.0.0:5000->5000/tcp             registry \
</p>



#### Exec inside the acmeair container
docker exec -it 51dcac8a2523 bash \
cd /opt/java/openjdk

#### Exec inside the mongodb container and confirm if the db is loaded correctly

Open a shell for the mongoldb container:
 
docker exec -it d9eee71fa442 bash

mongosh

test> show dbs \
acmeair  684.00 KiB \
admin     40.00 KiB \
config   108.00 KiB \
local     40.00 KiB \

test> use acmeair \
switched to db acmeair \

acmeair> show collections \
airportCodeMapping \
booking \
customer \
customerSession \
flight \
flightSegment \

acmeair> db.flight.count() \
DeprecationWarning: Collection.count() is deprecated. Use countDocuments or estimatedDocumentCount. \
2364 \
acmeair> db.customer.count() \
200 \
acmeair> db.airportCodeMapping.count() \
31 \
acmeair> db.flightSegment.count() \
394 \

#### Stress test for container 
----------------------------

Just a quick follow-up on pumba. Installation (for Ubuntu):

<p>
$ curl -SL https://github.com/alexei-led/pumba/releases/download/0.7.2/pumba_linux_amd64 -O

$ sudo mv pumba_linux_amd64 /usr/bin/pumba

$ sudo chmod +x /usr/bin/pumba

$ pumba --version
</p>

It uses stress-ng under the hood, so same commands apply. Examples:

$ pumba stress -d 1m container_name
-d 1m - duration of a stress test (1 minute)

container_name - your target container (from docker ps output)

$ pumba stress -d 30s --stressors "--vm 10 --vm-bytes 512M --vm-hang 20" container_name
-d 30 - duration 30 seconds

-stressors - parameters passed to stress-ng app

In this particular example we create 10 workers spinning on malloc()/free() allocating 512MB each and sleep 20 seconds before freeing the memory.

Default stressor as in first example is --cpu 4 --timeout 60s



https://www.markdownguide.org/basic-syntax/
