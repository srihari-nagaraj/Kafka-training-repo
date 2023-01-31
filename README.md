# Kafka-training-repo

Training Slide
https://docs.google.com/presentation/d/1YemAxWr8McpUDnOPziWTKKU-a2yQX9WBJRN6Mh-7Klk/edit?usp=sharing


# Kafka Local Installation 
# step 1 : Downloading and Extracting the Kafka Binaries

        curl "https://downloads.apache.org/kafka/3.3.2/kafka_2.12-3.3.2.tgz" -o  /home/path/kafka.tgz


Create a directory called kafka and change to this directory. This will be the base directory of the Kafka installation:

        mkdir ~/kafka && cd ~/kafka

Extract the archive you downloaded using the tar command:

        tar -xvzf /home/path/kafka.tgz --strip 1

We specify the --strip 1 flag to ensure that the archive’s contents are extracted in ~/kafka/ itself and not in another directory (such as              ~/kafka/kafka_2.12-3.3.2/) inside of it.

Now that we’ve downloaded and extracted the binaries successfully, we can start configuring our Kafka server.

# Step 2 — Configuring the Kafka Server
Kafka’s default behavior will not allow you to delete a topic. A Kafka topic is the category, group, or feed name to which messages can be published. To modify this, you must edit the configuration file.

Kafka’s configuration options are specified in server.properties. Open this file with nano or your favorite editor:

        nano ~/kafka/config/server.properties
        
First, add a setting that will allow us to delete Kafka topics. Add the following to the bottom of the file:
        
        delete.topic.enable = true

Second, change the directory where the Kafka logs are stored by modifying the logs.dir property:

        log.dirs=/home/path/kafka/logs
        
 Save and close the file. Now that you’ve configured Kafka, your next step is to create systemd unit files for running and enabling the Kafka server on startup.
 
# Step 3 — Creating Systemd Unit Files and Starting the Kafka Server
In this section, you will create systemd unit files for the Kafka service. This will help you perform common service actions such as starting, stopping, and restarting Kafka in a manner consistent with other Linux services.

Zookeeper is a service that Kafka uses to manage its cluster state and configurations. It is used in many distributed systems. If you would like to know more about it, visit the official Zookeeper docs.

Create the unit file for zookeeper:

        sudo nano /etc/systemd/system/zookeeper.service
        
Enter the following unit definition into the file:

        [Unit]
        Requires=network.target remote-fs.target
        After=network.target remote-fs.target

        [Service]
        Type=simple
        User=kafka
        ExecStart=/home/path/kafka/bin/zookeeper-server-start.sh /home/path/kafka/config/zookeeper.properties
        ExecStop=/home/path/kafka/bin/zookeeper-server-stop.sh
        Restart=on-abnormal

        [Install]
        WantedBy=multi-user.target

The [Unit] section specifies that Zookeeper requires networking and the filesystem to be ready before it can start.

The [Service] section specifies that systemd should use the zookeeper-server-start.sh and zookeeper-server-stop.sh shell files for starting and stopping the service. It also specifies that Zookeeper should be restarted if it exits abnormally.

After adding this content, save and close the file.

Next, create the systemd service file for kafka:

        sudo nano /etc/systemd/system/kafka.service
        
 Enter the following unit definition into the file:

        [Unit]
        Requires=zookeeper.service
        After=zookeeper.service

        [Service]
        Type=simple
        User=kafka
        ExecStart=/bin/sh -c '/home/path/kafka/bin/kafka-server-start.sh /home/path/kafka/config/server.properties > /home/path/kafka/kafka.log 2>&1'
        ExecStop=/home/path/kafka/bin/kafka-server-stop.sh
        Restart=on-abnormal

        [Install]
        WantedBy=multi-user.target
        
The [Unit] section specifies that this unit file depends on zookeeper.service. This will ensure that zookeeper gets started automatically when the kafka service starts.

The [Service] section specifies that systemd should use the kafka-server-start.sh and kafka-server-stop.sh shell files for starting and stopping the service. It also specifies that Kafka should be restarted if it exits abnormally.

Now that you have defined the units, start Kafka with the following command:

        sudo systemctl start kafka
        
To ensure that the server has started successfully, check the journal logs for the kafka unit:

        sudo systemctl status kafka
        
        output:
        kafka.service
        Loaded: loaded (/etc/systemd/system/kafka.service; disabled; vendor preset: enabled)
        Active: active (running) since Tue 2023-01-31 10:32:24 IST; 1s ago
        Main PID: 821986 (sh)
        Tasks: 34 (limit: 18526)
        Memory: 298.7M
        CPU: 3.150s
        CGroup: /system.slice/kafka.service
             ├─821986 /bin/sh -c "/home/sriharimn/kafka/bin/kafka-server-start.sh /home/sriharimn/kafka/config/server.properties >              /home/sriharimn/kafka/kafka.log 2>
             └─821987 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:Max>
        lines 1-10/10 (END)
        
You now have a Kafka server listening on port 9092.

You have started the kafka service. But if you rebooted your server, Kafka would not restart automatically. To enable the kafka service on server boot, run the following commands:
        
        sudo systemctl enable zookeeper
        sudo systemctl enable kafka
        
In this step, you started and enabled the kafka and zookeeper services. In the next step, you will check the Kafka installation.


# Docker compose run Kafka

1. Please make sure that Docker is already installed in the system,
If not, install from https://www.docker.com/products/docker-desktop

2. If on Windows O/S, open a Powershell window.
If on Mac OS or Linux, open a Terminal window.

find the file --> kafka-single-node.yml file

4. Execute the following command from this directory

        docker-compose -f kafka-single-node.yml up -d

5. Check if the containers are up and running

        docker ps


6. To shutdown and remove the setup, execute this command in the same directory

        docker-compose -f kafka-single-node.yml down



#Kafka commands:

## Logging into the Kafka Container

        docker exec -it kafka-broker /bin/bash

## Navigate to the Kafka Scripts directory

        cd /opt/bitnami/kafka/bin

## Creating new Topics

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --create \
            --topic kafka.learning.tweets \
            --partitions 1 \
            --replication-factor 1

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --create \
            --topic kafka.learning.alerts \
            --partitions 1 \
            --replication-factor 1

## Listing Topics

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --list

## Getting details about a Topic

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --describe


## Publishing Messages to Topics

        ./kafka-console-producer.sh \
            --bootstrap-server localhost:29092 \
            --topic kafka.learning.tweets

## Consuming Messages from Topics

        ./kafka-console-consumer.sh \
            --bootstrap-server localhost:29092 \
            --topic kafka.learning.tweets \
            --from-beginning

## Deleting Topics

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --delete \
            --topic kafka.learning.alerts



## Create a Topic with multiple partitions

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --create \
            --topic kafka.learning.orders \
            --partitions 3 \
            --replication-factor 1


## Check topic partitioning

        ./kafka-topics.sh \
            --zookeeper zookeeper:2181 \
            --topic kafka.learning.orders \
            --describe

## Publishing Messages to Topics with keys

        ./kafka-console-producer.sh \
            --bootstrap-server localhost:29092 \
            --property "parse.key=true" \
            --property "key.separator=:" \
            --topic kafka.learning.orders

## Consume messages using a consumer group

        ./kafka-console-consumer.sh \
            --bootstrap-server localhost:29092 \
            --topic kafka.learning.orders \
            --group test-consumer-group \
            --property print.key=true \
            --property key.separator=" = " \
            --from-beginning

## Check current status of offsets

        ./kafka-consumer-groups.sh \
            --bootstrap-server localhost:29092 \
            --describe \
            --all-groups




