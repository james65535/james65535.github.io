---
layout: article
title: Eclipse Container Integration with VMware AppCatalyst
modified:
categories: Infrastructure
excerpt:
tags: [VMware, IntelliJ, IDE, Docker, Containers, AppCatalyst, PhotonOS]
image:
  feature: "bar.jpg"
  teaser: "bar.jpg"
  thumb:
---
# Introduction
The past year has seen tremendous growth and support for Containerization within the Development space.  Containers provide a fast and easy way to fire up a multitude of different development, test, and even production environments.

Unfortunately Docker does not run natively on OSX, an abstraction layer is needed to create a Container Runtime Host for the Docker service to run on.  Enter VMware AppCatalyst and Project Photon:

* **Project Photon:** A lightweight purpose built Linux Container Runtime Host currently released as an Open Source Tech Preview project by VMware
* **AppCatalyst:** A CLI/RESTful API driven lightweight hypervisor for OSX which provides a perfect environment for running Photon instances hosting Docker services

The goal of this article is to walk the user through the initial configuration steps required to integrate VMware AppCatalyst with IntelliJ IDEA and Eclipse IDE.  This integration will provide the user a seemles method for automating the delivery of containers and deploying their artifacts right on their laptop without having to perform any manual steps outside of the IDE.  Just tell your IDE to build your App and you're ready to go!

## Target Audience
MAC OSX users who have at least a basic knowledge of Docker and reasonable knowledge of the Eclipse IDE.

## System Requirements
* AppCatalyst: OSX 10.9.4 or newer
* Eclipse IDE: JBoss Developer Studio 9.0, Mars, or newer

# Pre-requisites
1. Install VMware AppCatalyst:
    2. Follow the directions from the [Getting Started guide](https://communities.vmware.com/docs/DOC-29885)
    3. You should now be able to fire up a PhotonOS VM with AppCatalyst and SSH into the Photon shell prompt
    4. Leave your PhotonOS VM powered on


1. It is recommended to secure your Docker Daemon for listening on a secure TCP Port:
    2. Follow [the Protect the Docker Daemon Guide](https://docs.docker.com/articles/https/) for creating the relevant certificates, you do not need to perform the "Secure by Default" section
    3. You should now have all the certificates and keys required to setup secure access to Docker from your host machine

# Prep-work
1. From your host machine get the IP address of your Photon instance:

    {:.bash}
        appcatalyst guest getip <replace with vm name>
    
2. Copy the client certificates from the Photon instance to your preferred folder on your host machine, in this case/home/DockerCerts/:

    {:.bash}
        scp -i /opt/vmware/appcatalyst/etc/appcatalyst_insecure_ssh_key photon@192.168.135.128:"ca.pem cert.pem ca.pem key.pem" ~/DockerCerts/
    
3. SSH to your Photon instance from your host machine:

    {:.bash}
        ssh -i /opt/vmware/appcatalyst/etc/appcatalyst_insecure_ssh_key photon@192.168.135.128

2. Optional: Move your server side certificates to a more appropriate folder:  

    {:.bash}
        sudo cp ~/{ca.pem,server-cert.pem,server-key.pem} /etc/ssl/certs/
    
2. The Docker service in Photon is configured through SystemD.  From your Photon SSH connection, modify your service file to change from Unix Socket to a secure TCP connection:  

    {:.bash}
        sudo nano /lib/systemd/system/docker.service
    
1. Find the following line:  

    {:.bash}
        ExecStart=/bin/docker -d -s overlay
    
2. Replace it with the following line:

    {:.bash}
        ExecStart=/bin/docker -d -s overlay --tlsverify --tlscacert=/etc/ssl/certs/ca.pem --tlscert=/etc/ssl/certs/server-cert.pem --tlskey=/etc/ssl/certs/server-key.pem -H=0.0.0.0:2376
    
3. Reboot your Photon instance or reload and restart Docker using systemctl:

    {:.bash}
        sudo shutdown -r now
    
    Or
    
    {:.bash}
        sudo systemctl daemon-reload
        sudo systemctl restart docker

# Eclipse IDE Configuration
These steps will walk you through setting up your IDE for Docker integration.  It is presumed you already have a project to use, if not create a simple Java EE project with a WAR artifact labeled: JavaEEHelloWorld_war.war.

1. Setup the Linux Tools Docker Plugin within your Eclipse IDE
    2. Navigate to `Help` -> `New Software`
    3. In the `Work with` text box enter the following address:
            
        {:.bash}
            http://download.eclipse.org/linuxtools/updates-docker-nightly/

    4. Expand `Linux Tools`, select `Docker Client` and `Docker Tooling`, click `Next` twice and `Finish`, and restart
    5. Add the three `Docker Tools` views by navigating to `Windows` -> `Show View` -> `Other` -> `Docker`, and select each view
    6. In the `Docker Explorer` view click `No connection to a Docker daemon is available.  Click this link to create a new connection`
    5. Specify the Docker API URL:  ```https://192.168.135.128:2376```
    6. Specifiy the Certificates Folder where you previously copied your client certificates to using SCP and hit `OK`
    7. Verify you can see the new Docker server within the `Docker Explorer` view.  Any images or containers should be visible within the appropriate folders
    8. In the `Docker Images` view you can create a new image by clicking the `Build Image` icon
    9. Specify a name of the image, browse to a folder where you would like to place the Dockerfile, in this case create a folder called `Docker-out` within your project folder
    10. Click `Edit Dockerfile` and specify the following:

        {:.bash}
                FROM jboss/wildfly
                ADD JavaEEHelloWorld_war.war /opt/jboss/wildfly/standalone/deployments/
    
    11. Place your `JavaEEHelloWorld_war.war` artifact in the `Docker-out` folder
    12. Now select the new image and click the 'Run Image' icon
    13. You should get a Wizard dialog to input the desired settings for the container but this does not currently seem to appear.  Clicking on this button produces no noticable result
    
# Eclipse IDE Post Deployment Test
1. To test a successful build and deploy from Eclipse IDE to your Docker container you can try one of the following steps:
    2. In your Web browser, navigate to 'http://192.168.135.128:18080/JavaEEHelloWorld_war/' to see your WebApp successfully running
    3. SSH to your Photon Instance and issue the Docker command to list running containers.  You should see 'My-WebApp' under the `Names` column:

        {:.bash}
            ssh -i /opt/vmware/appcatalyst/etc/appcatalyst_insecure_ssh_key photon@<insert guest ip here>
            sudo docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=<insert guest ip here>:2376 ps

# What's Next
So now you've configured a basic project within your IDE to work with VMware AppCatalyst, Photon, and Docker containers.  You can now look to implement these settings within your own project work and deploy more complex environments with the click of a button!

# Appendix

## Useful Links
* [VMware Cloud Native Apps Splash Page](http://www.vmware.com/cloudnative)
* [VMware AppCatalyst Splash Page](https://communities.vmware.com/community/vmtn/devops/vmware-appcatalyst)
* [VMware AppCatalyst Getting Started Guide](https://communities.vmware.com/docs/DOC-298850)
* [VMware Project Photon Github](https://github.com/vmware/photon)
* [Docker Docs for getting started with the Dockerfile](https://docs.docker.com/reference/builder/)
* [Docker Docs article for securing Docker Service with HTTPS](https://docs.docker.com/articles/https/)
* [Eclipse Docker Tooling Newsletter](http://www.eclipse.org/community/eclipse_newsletter/2015/june/article3.php)

## Known issues:
* AppCatalyst cannot be used while VMware Fusion is running
* Certificate configuration uses 192.168.135.128 address, if the target Photon instance is not the first instance powered on it will have a different IP address.  Ensure the target Photon instance is the first instance powered on to maintain a consistent IP address