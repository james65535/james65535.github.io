---
layout: article
title: Docker Integration with IntelliJ IDEA
modified:
categories: Development
excerpt: 
tags: [VMware, Go, Golang, IDE, Docker, Containers, IntelliJ, IDEA]
image:
  feature: 
  teaser: "bar.jpg"
  thumb:
author: James
---

# Introduction
Often times you may have a need to spin up multiple machines to test out an application you are developing.  Frequently this can lead you to install all manner of software and frameworks on your local laptop.  Overtime this may lead to application conflicts or just a plain mess.  With IntelliJ IDEA and Docker, it's now quite easy to deploy your application into a Docker container or containers directly from your IDE with the click of a button.

## System Requirements
* IntelliJ IDEA Ultimate Edition: 14.1 or newer

# Pre-requisites
1. A Linux OS with Docker Engine.  This may be running remotely, within VMware Fusion/Workstation, Docker Machine, etc...

1. It is recommended to secure your Docker Daemon for listening on a secure TCP Port:
    2. Follow [the Protect the Docker Daemon Guide](https://docs.docker.com/articles/https/) for creating the relevant certificates, you do not need to perform the "Secure by Default" section
    3. You should now have all the certificates and keys required to setup secure access to Docker from your host machine


# Prep-work
1. From your host machine get the IP address of your Photon instance:

    {:.bash}
        appcatalyst guest getip <replace with vm name>
    
2. Copy the client certificates from your Docker Host to your preferred folder on your development machine, in this case: /home/DockerCerts/:

    {:.bash}
        scp username@192.168.135.128:"ca.pem cert.pem ca.pem key.pem" ~/DockerCerts/
    
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

# IntelliJ IDEA Configuration
For this example, a simple Java Enterprise Project is used with the `Web Application` framework deployed as a Web Application Archive.  Please feel free to create a new project, use an existing one, or follow the steps in the tutorial from JetBrains: https://www.jetbrains.com/idea/help/developing-a-java-ee-application.html

1. Setup the Docker plug-in:
    2. Navigate to `IntelliJ IDEA` -> `Preferences` -> `Build, Execution, Deployment` -> `Clouds`
    3. Select the `+` symbol and add `Docker`
    4. Specificy name as `Docker`
    5. Specify the Docker API URL:  ```https://192.168.135.128:2376```
    6. Specifiy the `Certificates Folder` where you copied your client certificates to with SCP and hit `OK`

        ![IntelliJ IDEA Cloud Pref](https://james65535.github.io/images/posts/intellijcloudpref.png)

2. Configure your project specific settings:
    3. Create a folder under your project root folder named `Docker-out`
    3. You will need to create a Dockerfile to contain the commands which are issued to Docker during image build.  Within the `Docker-out` directory, create a new file called 'Dockerfile' and enter the following commands.  The first line specificies the base image and the second specifices the artifact you wish to add to the new image: 

        {:.bash}
            FROM jboss/wildfly
            ADD JavaEEHelloWorld_war.war /opt/jboss/wildfly/standalone/deployments/
    
    4. Your deployment artifacts will need to be placed in the same folder as your `Dockerfile`.  Navigate to `File` -> `Project Structure`
    10. Make sure your `JavaEEHelloWorld_war.war` artifact is selected and set your output directory to the `Docker-out` directory and hit `OK`
    
        ![IntelliJ IDEA Project Folders](https://james65535.github.io/images/posts/folderlayout.png)

3. Now the project `Run Configuration` needs to be configured for the deployment:
    11. Navigate to `Run` -> `Edit Configurations`, select `+` and add a new `Docker Deployment`
    12. Specify the name as `Docker Deployment`
    13. Ensure Deployment specifies `Docker-Out/Dockerfile`
    13. Choose a name for the container name, e.g. `My-WebApp`
    14. Next to `Container Settings` click the Green Arrow symbol and create a folder within your project folder labeled `Docker-Settings`
    15. In the file-name box 'container-settings.json' should already be specified, hit `OK`
    16. Next to 'Before Launch' click the `+` symbol and select `Build Artifacts`
   
        ![IntelliJ IDEA Project Run Config](https://james65535.github.io/images/posts/runconfig.png)

    17. Choose your artifact and hit `OK` to both windows
    18. Finally navigate to `Run` -> `Run Docker Deploy`

IntelliJ IDEA will now build your artifact, communicate to your Photon hosted Docker service, start up a new Container, and deploy your artifact to the container.  If this is the first time you have pulled down the Docker base image then it may take 5-15 minutes to download depending on your internet connection.

# IntelliJ IDEA Post Deployment Test
1. To test a successful build and deploy from IntelliJ IDEA to your Docker container you can try one of the following steps:
    2. In your Web browser, navigate to 'http://192.168.135.128:18080/JavaEEHelloWorld_war/' to see your WebApp successfully running
    3. SSH to your Photon Instance and issue the Docker command to list running containers.  You should see 'My-WebApp' under the `Names` column:

        {:.bash}
            ssh -i /opt/vmware/appcatalyst/etc/appcatalyst_insecure_ssh_key photon@<insert guest ip here>
            sudo docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=<insert guest ip here>:2376 ps

# What's Next
So now you've configured a basic project within your IDE to work with VMware AppCatalyst, Photon, and Docker containers.  You can now look to implement these settings within your own project work and deploy more complex environments with the click of a button!

# Appendix

## Useful Links
* [JetBrains guide for creating a Java EE Application](https://www.jetbrains.com/idea/help/developing-a-java-ee-application.html)
* [VMware Cloud Native Apps Splash Page](http://www.vmware.com/cloudnative)
* [VMware Project Photon Github](https://github.com/vmware/photon)
* [Docker Docs for getting started with the Dockerfile](https://docs.docker.com/reference/builder/)
* [Docker Docs article for securing Docker Service with HTTPS](https://docs.docker.com/articles/https/)
* [Jetbrains blog for configuring IntelliJ IDEA Docker Plug-in](http://blog.jetbrains.com/idea/2015/03/docker-support-in-intellij-idea-14-1/)
