---
layout: post
title: "Head First Ooize Sqoop Action"
description: "Step by step to run an Oozie Sqoop action"
category: "bigdata"
tags: ["bigdata", "hadoop", "oozie", "sqoop"]
comments: true
---
{% include JB/setup %}

Run Sqoop Action with Oozie.

>local/oozie_job_conf_path/job.properties

    oozie.use.system.libpath=true
    namenode=hdfs://hdp-nn:8020
    yarn=hdp-nn:8050
    oozie.wf.application.path=${namenode}/oozie_job_path

    outputpath=${namenode}/temp

>hdfs/oozie_job_path/workflow.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <workflow-app xmlns="uri:oozie:workflow:0.2" name="sqoop-wf">
        <start to="table-test"/>
    
        <action name="table-test">
            <sqoop xmlns="uri:oozie:sqoop-action:0.2">
                <job-tracker>${yarn}</job-tracker>
                <name-node>${namenode}</name-node>
                <prepare>
                    <delete path="${outputpath}"/>
                    <mkdir path="${outputpath}"/>
                </prepare>
                <arg>import</arg>
                <arg>--connect</arg>
                <arg>jdbc:postgresql://192.168.1.101:5432/dbname</arg>
                <arg>--username</arg>
                <arg>postgres</arg>
                <arg>--password</arg>
                <arg>PWD!@#</arg>
                <arg>--table</arg>
                <arg>this_is_table_name</arg>
                <arg>--split-by</arg>
                <arg>vehicle_sk</arg>
                <arg>--fields-terminated-by</arg>
                <arg>","</arg>
                <arg>--target-dir</arg>
                <arg>${outputpath}</arg>
                <arg>--append</arg>
                <arg>-m</arg>
                <arg>1</arg>
                <arg>--</arg>
                <arg>--schema</arg>
                <arg>dw</arg>
            </sqoop>
            <ok to="end"/>
            <error to="fail"/>
            
        </action>
        <kill name="fail">
            <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
        </kill>
        <end name="end"/>
    </workflow-app>

1. Put _workflow.xml_ to HDFS/_oozie_job_path_ which defined in _job.properties_ with the key **oozie.wf.application.path**.

2. Put related jar packages to HDFS/_oozie_job_path_/lib. Such as JDBC connector.

3. Run Sqoop ooize job

    sudo -u hdfs oozie job -oozie http://oozie-server-hostname:11000/oozie -config job.properties -run
