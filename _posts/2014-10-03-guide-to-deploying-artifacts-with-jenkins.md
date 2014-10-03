---
layout: post
name: quick-guide-to-deploying-artifacts-with-Jenkins
title: A Very Quick Guide to Deploying Artifacts With Jenkins
date: 2014-10-03
author: Toby Retallick
image:
    src: /assets/img/blog/jenkinsPost/jenkins.png
tags:
- continuous integration
- Jenkins
---

[Jenkins](http://jenkins-ci.org/) is an open-source continuous integration sever. At Codurance, we use Jenkins to build and test projects to make sure everything is in order before handing over to the client.

Both Amir and I have just recently started at Codurance as apprentices. 

This week, one of our tasks was to set up Jenkins on a new server then add a client’s project for building and testing. Thanks to an [excellent tutorial from Jeff Shantz](http://www.youtube.com/watch?v=zojMg2c6k3Q), it ended up being a relatively straight forward task. We then wanted to create a way for the client to be able to download their tested application. However we got more than a little stumped it came to deploying those assets to a new location on the server. Frantic Googling had us going round in circles.

Luckily, a pairing session with Codurance Craftsman Samir pointed us in the right direction and we thought we would set out our process below incase any other Jenkns newbies got stuck like we did. 

I'll assume you have a client project set up on Jenkins, and it is connected to your version control system. If you have not got this far then checking out the link to the tutorial earlier will see you right.


#### Step 1: Create a new Jenkins Item
Select 'New item' from the main menu and call it something like "[ENTER-PROJECT-NAME]-Output". Its purpose will be solely export the files from your client project to a folder of your choosing on the server. 


#### Step 2: Create a post-build action
Go to your client project and select configure. Create a post-build action and select ‘archive artififacts’ from the drop down menu. Add the type of files you want to archive (and eventually, copy and export). Next add another post-build action ‘Build other project’ and enter the name of the build item you created earlier. 

![Archive artifacts build step](/assets/img/blog/jenkinsPost/archiveArtifacts.png)


#### Step 3: Install the Copy Artifact plugin
Next you need to install the Copy Artifact plugin in the Manage Plugins section of Jenkins. Go to "[PROJECT-NAME]-Output" > configure and add a new build step. Because you have installed the Copy Artifact plugin you should see an option called ‘copy artifacts from another project’ in the drop down menu. Specify the types of folder/files you want copied and set your location. Notice that we set set our location "var/www/clients/…". This is the path to a new folder on the server (we were using an Apache server on an Amazon EC2 instance). Don't do what we did and set the path using an http address(!).

![Archive artifacts build step](/assets/img/blog/jenkinsPost/copyArtifacts.png)


#### Step 4: Test it Out
Trigger a build from your client-project. This should then trigger a new build in [PROJECT-NAME]-Output. Check the deployment folder you set up on the server. Hopefully you will see the newly-deployed files but...


#### Wait, the build failed! I got a FileException error... 

It may mean that Jenkins does not have the right permissions to write to the folder and cannot therefore copy the files.

SSH into your server and check the permissions of you output folder. We ran into this problem and ended up:

1. adding Jenkins to the www-data group 
2. changed ownership of the folder from ubuntu to www-data with the command <code>sudo chown -R :www-data clients</code>
3. enabling write access on the folder with the command <code>sudo chmod -R g+w clients</code>

Don’t forget to restart your server to finalise the changes!

![Archive artifacts build step](/assets/img/blog/jenkinsPost/console.png)