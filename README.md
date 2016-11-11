# metronome
A Docker image (with build process) for dcos/metronome

## Usage

You can pass all configuration properties via environment variables, but with `_` instead of `.` (all lowercase). The properties can be found at [src/main/scala/dcos/metronome/MetronomeConfig.scala](https://github.com/dcos/metronome/blob/master/src/main/scala/dcos/metronome/MetronomeConfig.scala#L16-L49).

The current list of configuration properties is:

There are many options understood by metronome.  The following is an incomplete list:

```
metronome.framework.name
metronome.mesos.master.url
metronome.mesos.leader.ui.url
metronome.mesos.role
metronome.mesos.user
metronome.mesos.executor.default
metronome.mesos.failover.timeout
metronome.mesos.authentication.principal
metronome.mesos.authentication.secret.file
metronome.features.enable
metronome.plugin.dir
metronome.plugin.conf
metronome.history.count
metronome.behavior.metrics
metronome.leader.election.hostname
metronome.akka.ask.timeout
metronome.zk.url
metronome.zk.session_timeout
metronome.zk.timeout
metronome.zk.compression.enabled
metronome.zk.compression.threshold
metronome.scheduler.reconciliation.interval
metronome.scheduler.reconciliation.timeout
metronome.scheduler.store.cache
metronome.scheduler.task.launch.timeout
metronome.scheduler.task.launch.confirm.timeout
metronome.scheduler.task.env.vars.prefix
metronome.scheduler.task.lost.expunge.gc
metronome.scheduler.task.lost.expunge.initial.delay
metronome.scheduler.task.lost.expunge.interval
metronome.leader.preparation.timeout
metronome.leader.proxy.timeout
metronome.akka.actor.startup.max
```

There are 2 ways to set these variables using this container:

- JAVA_OPTS
```
docker run -e JAVA_OPTS="-Dmetronome.zk.url=zk://localhost:2181/metronome -Dmetronome.mesos.master.url=localhost:5050 f4tq/metronome:0.9.1
```
- individual environment variables that are transformed into the above notation
```
docker run -e METRONOME_ZK_URL=zk://localhost:2181/metronome -e METRONOME_MESOS_MASTER_URL=localhost:5050 f4tq/metronome:0.9.1
```
- to override the entrypoint in this release and run metronome directly, run:
```
docker run -i --rm --entryppoint /bin/bash -t f4tq/metronome:0.9.1 /app/metronome-0.1.9/bin/metronome -Dmetronome.mesos.master.url=localhost:5050 -Dmetronome.zk.url=zk://localhost:2181/metronome -Dmetronome.play.server.http_port=9000 
```

> Note: metronome is based on the [play framework](https://www.playframework.com): There are **many** more options that can be passed via the command that relate to play; so many in fact that metronone takes -Duser.dir=X where X should contain a `X/conf` directory with `application.conf` contained within.  Mesos takes the `-Duser.dir` approach.

 

### Start with Marathon

At minimum, you need to specify `metronome_mesos_master_url` and `metronome_zk_url` in an equivalent way as outlined below.

```
{
  "id": "/metronome",
  "cpus": 1,
  "mem": 1024,
  "disk": 0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "f4tq/metronome:0.1.9",
      "network": "HOST",
      "privileged": false,
      "parameters": [],
      "forcePullImage": true
    }
  },
  "env": {
    "metronome_mesos_master_url": "172.17.10.101:5050",
    "metronome_zk_url": "zk://172.17.10.101:2181/metronome"
  },
  "ports": [0],
  "healthChecks": [
    {
      "path": "/ping",
      "portIndex": 0,
      "protocol": "HTTP",
      "gracePeriodSeconds": 30,
      "intervalSeconds": 10,
      "timeoutSeconds": 3,
      "maxConsecutiveFailures": 1
    }
  ]
}
```
