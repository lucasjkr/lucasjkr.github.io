# TO LINODE:

This is incomplete - just a guide for installing elasticsearch on Ubuntu Server 16.04.

If you like it, I would actually prefer to do a series of articles, including:

1 - Installing the ELK stack on Ubuntu (Elasticsearch, Kibana, Logstash)

2 - Feeding data into Logstash from a second server

3 - Securing your installation.

That said, here is the first part of the first article for your review. Please let me know your thoughts, edits, etc:

## Introduction

During the course of this guide, I'll use a few placeholder variables that you should update with ones specific to your system, including:

**my-first-cluster** - The name you will assign to your elasticstack installation

**111.222.333.444** - External IP Address

**yourhost**        - Your systems hostname ('*elastic*')

**domain.com**      - Your domain name ('*domain.com*')

**server.fqdn.com**        - Fully qualified domain name ('*elastic.domain.com*')

> The steps in this guide require root privileges. Be sure to run the steps below as `root` or with the `sudo` prefix. For more information on privileges, see our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Before You Begin:
1.  Familiarize yourself with our [Getting Started](/docs/getting-started) guide and complete the steps for setting your Linode's hostname and timezone.

2.  This guide will use `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server) to create a standard user account, harden SSH access and remove unnecessary network services. Do **not** follow the Configure a Firewall section yet--this guide includes firewall rules specifically for an OpenVPN server.

3.  Update your system:

        sudo apt-get update && \
          sudo apt-get upgrade && \
          sudo-apt-get dist-upgrade

4.  Install the prerequisites:

Most systems will have the following tools installed by default, but just in case, lets' make sure they're installed on your system:

        sudo apt-get install wget \
          unzip \
          git \
          apt-transport-https \
          nano

## Lets get started!

### Step One: Install Java

The first step is to install Java8. You can choose either Oracle's offical implementation 
(Java8) or OpenJDK, the open-source alternative:

####If you choose Oracle Java

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer

**Verify your installation:**

    java -version

You should see output like this:

    java version "1.8.0_131"
    Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

####To install OpenJDK

    sudo apt-get install openjdk-8-jre

**Verify your installation**

    java -version

You should see output like this:

    openjdk version "1.8.0_131"
    OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-2ubuntu1.16.04.2-b11)
    OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)

### Prepare for installing Elasticstack

First, add elastic's GPG key to your keyring:

    wget - https://packages.elastic.co/GPG-KEY-elasticsearch 

Verify the key you downloaded:

    gpg --with-fingerprint GPG-KEY-elasticsearch

According to [Elastic's website](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html), their key fingerprint is:

    4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4

If that is the case, you should add to your sources keyring:

    sudo apt-key add GPG-KEY-elasticsearch

Second, add Elastics official repository to your systems sources:

    sudo echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | \
      sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
 
Update your repository cache: 

    sudo apt-get update

### Install Elastic Search

From here on out, it's pretty smooth sailing. First you need to install elasticsearch itself:

    sudo apt-get -y install elasticsearch

You'll need to make some edits to elasticsearch's configuration file, but before doing so lets back up the original (default) file:

    sudo cp /etc/elasticsearch/elasticsearch.yml \
      /etc/elasticsearch/elasticsearch.yml.original

Since (in a later guide), we'll be feeding data into elasticsearch through Logstash, which will run on the same server, we'll want to bind elasticsearch to the systems local adapter. Open /etc/elasticsearch/elasticsearch.yml:

    sudo nano /etc/elasticsearch/elasticsearch.yml

 and search for the line that reads:

    #network.host: 192.168.0.1

And change it to read as follows:

    network.host: localhost

You will also want to give your cluster a name, so find the line that reads:

    #cluster.name: my-application

And replace it with a name of your choosing:

    cluster.name: my-first-cluster


### Optional - reduce elasticsearch's memory allocation:

Elasticsearch loves to gobble up memory. Elastic recommends against allocating more than 32GB to the Java VM that runs ES, and by default ES is given 2GB.  If you want to reduce that footprint for testing purposes, you can do so by editing /etc/elasticsearch/jvm.options.

Remembe that before making any edits, you should make a backup copy of the file

    sudo cp /etc/elasticsearch/jvm.options \
      /etc/elasticsearch/jvm.options.original

Open it for editting:

    sudo nano /etc/elasticsearch/jvm.options

Look for the lines that read:

    -Xms2g
    -Xmx2g

And replace them as follows:

    -Xms512m
    -Xmx512m

You can now set elasticsearch the elasticsearch service to start automatically:

    sudo update-rc.d elasticsearch defaults 95 10

Lets start the elasticsearch service for the first time:

    sudo service elasticsearch start

It can take a minute for the service to start. You can check on it by running:

    sudo service elasticsearch status

And finally, you can verify that it's running and accepting local connections by running the following command:

    curl localhost:9200

You should see a response that looks similar to this:

    {
      "name" : "fSRGO4N",
      "cluster_name" : "my-first-cluster",
      "cluster_uuid" : "uLSmbNfrTz6MHnERoPcDxQ",
      "version" : {
        "number" : "5.5.1",
        "build_hash" : "19c13d0",
        "build_date" : "2017-07-18T20:44:24.823Z",
        "build_snapshot" : false,
        "lucene_version" : "6.6.0"
      },
      "tagline" : "You Know, for Search"
    }


## Action Summary:

Here is an non-annotated version of the steps you've run so far:

    #update repositories
    sudo apt-get update && \
      sudo apt-get upgrade && \
      sudo-apt-get dist-upgrade

    #Install utilities
    sudo apt-get install wget \
      unzip \
      git \
      apt-transport-https \
      nano

    #Install Java
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer

    #Optional, check java version
    java -version

    #Add elastic signing key
    wget - https://packages.elastic.co/GPG-KEY-elasticsearch 
    gpg --with-fingerprint GPG-KEY-elasticsearch
    sudo apt-key add GPG-KEY-elasticsearch

    #Add elastic package source
    sudo echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | \
      sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
    
    #Update Repositories
    sudo apt-get update

    #Install elasticsearch
    sudo apt-get -y install elasticsearch

    #Backup and edit elasticsearch configuration file
    sudo cp /etc/elasticsearch/elasticsearch.yml \
      /etc/elasticsearch/elasticsearch.yml.original

    sed -i 's/#network.host: 192.168.0.1/network.host: localhost/g' \
      /etc/elasticsearch/elasticsearch.yml
    
    #Important: Choose your own cluster name:
    sed -i 's/#cluster.name: my-application/cluster.name: my-first-cluster/g' \
      /etc/elasticsearch/elasticsearch.yml

    #Reduce JVM memory usage for this test phase
    sudo cp /etc/elasticsearch/jvm.options \
      /etc/elasticsearch/jvm.options.original

    sudo sed -i 's/-Xms2g/-Xms512m/g' \
      /etc/elasticsearch/jvm.options
    sudo sed -i 's/-Xmx2g/-Xmx512m/g' \
      /etc/elasticsearch/jvm.options

    #Set elasticsearch service to start automatically
    sudo update-rc.d elasticsearch defaults 95 10

    #start service manually
    sudo service elasticsearch start

Wait a minute or so, and then:

    curl localhost:9200
