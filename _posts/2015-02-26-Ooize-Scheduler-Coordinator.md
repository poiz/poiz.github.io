---
layout: post
title: "Oozie Coordinator"
description: "Cron scheduled coordinator job"
category: "bigdata"
tags: ["bigdata", "hadoop", "oozie", "coordinator", "scheduled", "cron"]
comments: true
---
{% include JB/setup %}

Run Cron scheduled coordinator job with Oozie.

>local/oozie_job_conf_path/job.properties

    # cluster
    namenode=hdfs://hdp-nn:8020
    yarn=hdp-nn:8050

    # oozie
    oozie.use.system.libpath=true
    oozie.coord.application.path=${namenode}/user/hansen/oozie/test

    poiz_workflow_path=${namenode}/user/hansen/oozie/test

    # job
    poiz_job_mainclass=com.poiz.ignition.hadoop.common.ScheduleAction

    poiz_schedular=0 0/1 * * *
    poiz_schedular_interval=120
    poiz_schedular_start=2015-02-11T08:00Z
    poiz_schedular_end=2015-02-12T08:00Z
    poiz_schedular_timezone=UTC

>hdfs/oozie_job_path/workflow.xml

    <workflow-app name="schedular-java-main-wf" xmlns="uri:oozie:workflow:0.2">

        <start to="java-node" />

        <action name="java-node">
            <java>
                <job-tracker>${yarn}</job-tracker>
                <name-node>${namenode}</name-node>
                <main-class>${poiz_job_mainclass}</main-class>
                <arg>${poiz_schedular_interval}</arg>
            </java>

            <ok to="end" />
            <error to="fail" />
        </action>

        <kill name="fail">
            <message>Java failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
        </kill>

        <end name="end" />

    </workflow-app>

>hdfs/oozie_job_path/coordinator.xml

    <coordinator-app name="schedular-application" xmlns="uri:oozie:coordinator:0.2"
        frequency="${poiz_schedular}"
        start="${poiz_schedular_start}"
        end="${poiz_schedular_end}"
        timezone="${poiz_schedular_timezone}">
        <action>
            <workflow>
                <app-path>${poiz_workflow_path}</app-path>
            </workflow>
        </action>
    </coordinator-app>


1. Put _workflow.xml_ & _coordinator.xml_ to HDFS/_oozie_job_path_ which defined in _job.properties_ with the key **oozie.coord.application.path**.

2. Put related jar packages to HDFS/_oozie_job_path_/lib.

3. Run Sqoop ooize job

    sudo -u hdfs oozie job -oozie http://oozie-server-hostname:11000/oozie -config job.properties -submit
