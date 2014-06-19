---
title: 'deploy &#038; debug ovirt in the intellij ide'
author: lofyer
layout: post
comments: true
duoshuo_thread_id:
  - 1234836220387786882
categories:
  - Virtualization
---
Before we start, we need intellij enterprise version and a register tool..  
[url]

[Summary]  
1. We should open the pom.xml file instead of the project directory.  
2. Modify the extension of the directory in the project properties.  
3. Modify the .m2 file in your personal directory to make jboss know where to deploy the project.  
4. We need seperate run/debug config to make it clearer.

[Steps]  
*OVIRT_HOME=/home/demo  
JBOSS_HOME=/usr/jboss-as  
JDK=/usr/java/jdk1.7.0_17  
$ cmd as demo  
\# cmd as root*  
**1.Compile the ovirt from src**  
in $OVIRT_HOME

<pre>$ mvn clean install -Pgwt-admin,gwt-user -DskipTests=true
</pre>

**2.Deploy ovirt**  
You need ~/.m2/settings.xml file in /root/ and /home/demo directory.

<pre>&lt;settings xmlns="http://maven.apache.org/POM/4.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">

<!--**************************** PROFILES ****************************-->

        &lt;activeProfiles>
                        &lt;activeProfile>oVirtEnvSettings&lt;/activeProfile>
        &lt;/activeProfiles>

        &lt;profiles>
                &lt;profile>
                        &lt;id>oVirtEnvSettings&lt;/id>
                        &lt;properties>
                           &lt;jbossHome>/usr/jboss-as&lt;/jbossHome>
                           &lt;JAVA_HOME>/usr/java/jdk1.7.0_17&lt;/JAVA_HOME>
                        &lt;/properties>
                &lt;/profile>
         &lt;/profiles>
&lt;/settings>
</pre>

in $OVIRT_HOME/ear

<pre># mvn clean install -Pdep,setup
</pre>

**3.Run jboss**  
You should export an environment variant and a runtime configuration file.

<pre>$ echo -e "ENGINE_USR=admin111nENGINE_ETC=/etc/ovirt-engine" > $OVIRT_HOME/backend/manager/conf/engine.conf.defaults
# export ENGINE_DEFAULTS=$OVIRT_HOME/backend/manager/conf/engine.conf.defaults
# /usr/jboss-as/bin/standalone.sh
</pre>

UPDATE:  
Now there is README.developer in the source dir, just follow it.