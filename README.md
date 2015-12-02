mesos-influxdb-collector
=====

Lightweight mesos stats collector for influxdb

Since this collector is intended to be deployed as a [marathon](https://mesosphere.github.io/marathon) app, it comes with a *lifetime* param. This defines how long the collector will run until it dies, so marathon will re-launch it, allowing easy allocation optimizations. Check the [fixtures/marathon/](https://github.com/kpacha/mesos-influxdb-collector/tree/master/fixtures/marathon) folder for more details on how to launch it.

# Goals

+ Discover the mesos cluster through `mesos-dns`
+ Collect the mesos leader stats
+ Collect the mesos master stats
+ Collect the mesos slave stats
+ Collect the marathon master stats
+ Collect the chronos task stats (TODO)

# Installing

Docker images are available at [docker hub](https://hub.docker.com/r/kpacha/mesos-influxdb-collector). Just pull the latest image available with:

```
$ docker pull kpacha/mesos-influxdb-collector:latest
```

Alternatively, get the last released binary from the [releases](https://github.com/kpacha/mesos-influxdb-collector/releases) section.


Finally, if you love the hard way and have Go installed:

```
$ go get github.com/kpacha/mesos-influxdb-collector
```

# Integration with `mesos-dns`

The `mesos-influxdb-collector` is able to discover all your mesos nodes (masters and slaves) and the marathon master using the REST API exposed by the [mesos-dns](http://mesosphere.github.io/mesos-dns/) service. Check the next section for details.

# Configuration

The collector use these environmental vars:

+ `INFLUXDB_USER`
+ `INFLUXDB_PWD`

It also requires a config file with the list of nodes to monitor or the details about the `mesos-dns` service among these other params:

+ *Lapse*: time between consecutive collections. Default: 30 seconds
+ *DieAfter*: duration of the running instance. Default: 1 hour

### MesosDNS

Optional. Add it if you have a `mesos-dns` service running in your mesos cluster.

```
mesosDNS {
  domain = "mesos" // the domain used by the mesos-dns service
  marathon = true // resolve marathon master
  host = "master.mesos" // host of the mesos-dns service
  port = 8123 // port of the REST API
}
```

### InfluxDB

Required.

```
influxdb {
  host = "influxdb.marathon.mesos" // host of the influxdb instance
  port = 8086 // port of the REST API
  db = "mesos" // name of the database to use
  checkLapse = 30 // ping frequency
}
```

### Mesos masters

For manual definition of some (or all) mesos masters, use the `Master` struct:

```
master "leader" {
  host = "master0.example.com"
  port = 5051
  leader = true // optional
}
Master "follower1" {
  host = "master1.example.com"
  port = 5052
}
```

### Mesos slaves

For manual definition of some (or all) mesos slave, use the `Slave` struct:

```
Slave "0" {
  host = "slave0.example.com"
  port = 5051
}
Slave "1" {
  host = "slave1.example.com"
  port = 5051
}
Slave "2" {
  host = "slave2.example.com"
  port = 5051
}
```

### Marathon instances

For manual definition of some (or all) marathon instances, use the `Marathon` struct:

```
Marathon {
  host = "$HOST"
  port = 8088
  bufferSize = 10000
  Server "0" {
    host = "marathon1"
    port = 8080
  }
  Server "1" {
    host = "marathon2"
    port = 8080
  }
}
```

Check [`config/configuration_test.go`](https://github.com/kpacha/mesos-influxdb-collector/blob/master/config/configuration_test.go) and [`conf.hcl`](https://github.com/kpacha/mesos-influxdb-collector/blob/master/conf.hcl) for examples.

# Running

## Dockerized version

Run the container with the default params:

```
$ docker pull --name mesos-influxdb-collector \
    -e INFLUXDB_USER=admin \
    -e INFLUXDB_PWD=secret \
    -it --rm kpacha/mesos-influxdb-collector
```

If you need to customize something, copy the `conf.hcl`, make your changes and link it as a volume:

```
$ docker pull --name mesos-influxdb-collector \
    -v /path/to/my/custom/conf.hcl:/tmp/conf.hcl \
    -it --rm kpacha/mesos-influxdb-collector -c /tmp/conf.hcl
```

Tip: if you link your config file to `/go/src/github.com/kpacha/mesos-influxdb-collector/conf.hcl` you don't need to worry about that flag!

## Binary version

```
$ ./mesos-influxdb-collector -h
Usage of ./mesos-influxdb-collector:
  -c string
      path to the config file (default "conf.hcl")
```

The credentials for the influxdb database are accepted just as env_var (`INFLUXDB_USER` & `INFLUXDB_PWD`)

# Grafana dashboards

The [fixtures/grafana]((https://github.com/kpacha/mesos-influxdb-collector/tree/master/fixtures/grafana) folder contains several grafana dashboard definitions. Go to the grafana website and, after configuring the influxdb datasource, import them and start monitoring your mesos cluster.
