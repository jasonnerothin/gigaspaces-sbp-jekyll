---
layout: post
title:  Primary-Backup Zone Controller
categories: SBP
parent: production.html
weight: 900
---

{% compositionsetup %}
{% summary page %} {% endsummary %}
{% tip %}
**Summary:** {% excerpt %}Primary-Backup Zone Controller, allows you to specify a specific zone for primary instances and a different zone for backup instances.{% endexcerpt %}<br/>
**Author**: Shay Hassidim, Deputy CTO, GigaSpaces<br/>
**Recently tested with GigaSpaces version**: XAP 8.0<br/>
**Last Update:** April 2011<br/>

{% toc minLevel=1|maxLevel=1|type=flat|separator=pipe %}

{% endtip %}

# Overview

When a deploying a data grid, primary and backup instances will be provisioned in an arbitrary manner across the available machines running GSA/GSCs. You don't have a control where these will be physically located as the primary election mechanism determines the primary and backup instances location at the deploy time (first instance per partition elected as the primary).

In some cases you would like to determine the primary and backup instances location in an explicit manner. A simple approach would be to use zones, having one specific zone to host the primary instances and another zone to host the backup instances. These zones do not determine specific physical machines to host the primary/backup instances, but logical group of GSCs associated with a specific zone once started. Usually, the zone might reflect machines located in specific different racks or different data centers that are nearby having very fast and reliable connectivity in between.

{% indent %}
![PrimaryBackupZoneController.jpg](/attachment_files/sbp/PrimaryBackupZoneController.jpg)
{% endindent %}

## When the Primary-Backup Zone Controller should be used?
Having primary and backup instances on different remote sites that are far away from each other is not a recommended approach with read/write applications. The Primary-Backup Zone Controller approach intended to be used with read mostly scenarios (80% read) where the latency between the sites is extremely low (below one-two milliseconds) with high bandwidth capacity. Primary and backup instances should be located within the same LAN with high speed connectivity and high capacity bandwidth to allow the primary replicate data as fast as it can to the backup to minimize the replication overhead on the application behavior.

When there is a requirement to leverage remote sites for disaster recovery or remote clients access - most systems will have their data grid primary and backup instances (using synchronous mode) within the same LAN (Master data grid) with another grid and its data-grid (not sharing the same lookup service and GSM with the master data grid) running as a slave data grid where the [WAN Replication Gateway](./wan-replication-gateway.html) used to replicate data between the Master and Slave Data-Grid asynchronously. With multi-master architecture the WAN Replication Gateway may run a conflict resolver to handle concurrent updates of the same object in both sites.

## Controlling Primary/Backup Location
The Primary-Backup Zone Controller can be deployed as a PU or run as a stand-alone utility. It allows you to specify a specific zone for primary instances and a different zone for backup instances. Once the Primary-Backup Zone Controller deployed/started it relocates all the primary instances into GSCs associated with the primary zone and later relocates all the backup instances into GSCs associated with the backup zone. The Primary-Backup Zone Controller periodically checks the status of the deployed data grid and relocates relevant instances as needed.

# Running the Primary-Backup Zone Controller

The example below will show how to use the Primary-Backup Zone Controller to place all primary instances on Zone A and all backup instances on Zone B. We will start GSCs with 3 zones, deploy a data-grid and use the Primary-Backup Zone Controller to execute the relocation activity.

Step 1. Download the [Primary-Backup Zone Controller](/attachment_files/sbp/PrimaryBackupZoneController.zip) and extract it into an empty folder. You will find the source under the `src` folder and the compiled code under the `bin` folder.

Step 2. Start LUS and GSM:

{% inittab 1|top %}

{% tabcontent Windows %}

{% highlight java %}
gs-agent gsa.gsc 0 gsa.lus 1 gsa.gsm 1
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
./gs-agent.sh gsa.gsc 0 gsa.lus 1 gsa.gsm 1
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

Step 3. Start Zone A:

{% inittab 3|top %}

{% tabcontent Windows %}

{% highlight java %}
set GSC_JAVA_OPTIONS=-Dcom.gs.zones="A"
gs-agent gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
export GSC_JAVA_OPTIONS=-Dcom.gs.zones="A"
./gs-agent.sh gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

Step 4. Start Zone B:

{% inittab 4|top %}

{% tabcontent Windows %}

{% highlight java %}
set GSC_JAVA_OPTIONS=-Dcom.gs.zones="B"
gs-agent gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
export GSC_JAVA_OPTIONS=-Dcom.gs.zones="B"
./gs-agent.sh gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

Step 5. Start Zone C:

{% inittab 5|top %}

{% tabcontent Windows %}

{% highlight java %}
set GSC_JAVA_OPTIONS=-Dcom.gs.zones="C"
gs-agent 0 gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
export GSC_JAVA_OPTIONS=-Dcom.gs.zones="C"
./gs-agent.sh gsa.gsc 2 gsa.lus 0 gsa.gsm 0
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

Step 6. Deploy a Data-Grid:

{% inittab 6|top %}

{% tabcontent Windows %}

{% highlight java %}
gs deploy-space -cluster schema=partitioned-sync2backup total_members=2,1 mySpace
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
./gs.sh deploy-space -cluster schema=partitioned-sync2backup total_members=2,1 mySpace
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

Step 7. Run the Primary-Backup Zone Controller utility:

{% inittab 7|top %}

{% tabcontent Windows %}

{% highlight java %}
call c:\gigaspaces-xap-root\bin\setenv.bat
set JARS=%GS_JARS%;.\bin
java %JARS% -DpuName=mySpace -DprimaryZone=A -DbackupZone=B -Dlocators=127.0.0.1 com.gigaspaces.admin.PrimaryBackupController
{% endhighlight %}

{% endtabcontent %}

{% tabcontent Linux %}

{% highlight java %}
JSHOMEDIR=`dirname $0`/../gigaspaces-xap-root; export JSHOMEDIR
. ${JSHOMEDIR}/bin/setenv.sh
export JARS=$GS_JARS:./bin
java $JARS -DpuName=mySpace -DprimaryZone=A -DbackupZone=B -Dlocators=127.0.0.1 com.gigaspaces.admin.PrimaryBackupController
{% endhighlight %}

{% endtabcontent %}

{% endinittab %}

## Deploying the Primary-Backup Zone Controller as a PU
When deploying the Primary-Backup Zone Controller utility as a PU the PU configuration includes the following:

{% highlight java %}
<bean id="PrimaryBackupController" class="com.gigaspaces.admin.PrimaryBackupController" >
	<property name="primaryZone" value="A" />
	<property name="backupZone" value="B" />
	<property name="locators" value="127.0.0.1" />
	<property name="groups" value="" />
	<property name="puName" value="mySpace" />
	<property name="delayBetweenChecks" value="60" />
</bean>
{% endhighlight %}

