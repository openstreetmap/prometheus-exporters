[![Continuous Integration](https://github.com/nginxinc/nginx-prometheus-exporter/workflows/Continuous%20Integration/badge.svg)](https://github.com/nginxinc/nginx-prometheus-exporter/actions?query=workflow%3A%22Continuous+Integration%22)  [![FOSSA Status](https://app.fossa.io/api/projects/custom%2B1062%2Fgithub.com%2Fnginxinc%2Fnginx-prometheus-exporter.svg?type=shield)](https://app.fossa.io/projects/custom%2B1062%2Fgithub.com%2Fnginxinc%2Fnginx-prometheus-exporter?ref=badge_shield)  [![Go Report Card](https://goreportcard.com/badge/github.com/nginxinc/nginx-prometheus-exporter)](https://goreportcard.com/report/github.com/nginxinc/nginx-prometheus-exporter)

# NGINX Prometheus Exporter

NGINX Prometheus exporter makes it possible to monitor NGINX or NGINX Plus using Prometheus.

## Overview

[NGINX](http://nginx.org) exposes a handful of metrics via the [stub_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status). [NGINX Plus](https://www.nginx.com/products/nginx/) provides a richer set of metrics via the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and the [monitoring dashboard](https://www.nginx.com/products/nginx/live-activity-monitoring/). NGINX Prometheus exporter fetches the metrics from a single NGINX or NGINX Plus, converts the metrics into appropriate Prometheus metrics types and finally exposes them via an HTTP server to be collected by [Prometheus](https://prometheus.io/).

## Getting Started

In this section, we show how to quickly run NGINX Prometheus Exporter for NGINX or NGINX Plus.

### A Note about NGINX Ingress Controller

If you’d like to use the NGINX Prometheus Exporter with [NGINX Ingress Controller](https://github.com/nginxinc/kubernetes-ingress/) for Kubernetes, see [this doc](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md#5-access-the-live-activity-monitoring-dashboard) for the installation instructions.

### Prerequisites

We assume that you have already installed Prometheus and NGINX or NGINX Plus. Additionally, you need to:
* Expose the built-in metrics in NGINX/NGINX Plus:
    * For NGINX, expose the [stub_status page](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status) at `/stub_status` on port `8080`.
    * For NGINX Plus, expose the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) at `/api` on port `8080`.
* Configure Prometheus to scrape metrics from the server with the exporter. Note that the default scrape port of the exporter is `9113` and the default metrics path -- `/metrics`.

### Running the Exporter in a Docker Container

To start the exporter we use the [docker run](https://docs.docker.com/engine/reference/run/) command.

* To export NGINX metrics, run:
    ```
    $ docker run -p 9113:9113 nginx/nginx-prometheus-exporter:0.9.0 -nginx.scrape-uri=http://<nginx>:8080/stub_status
    ```
    where `<nginx>` is the IP address/DNS name, through which NGINX is available.

* To export NGINX Plus metrics, run:
    ```
    $ docker run -p 9113:9113 nginx/nginx-prometheus-exporter:0.9.0 -nginx.plus -nginx.scrape-uri=http://<nginx-plus>:8080/api
    ```
    where `<nginx-plus>` is the IP address/DNS name, through which NGINX Plus is available.

### Running the Exporter Binary

* To export NGINX metrics, run:
    ```
    $ nginx-prometheus-exporter -nginx.scrape-uri=http://<nginx>:8080/stub_status
    ```
    where `<nginx>` is the IP address/DNS name, through which NGINX is available.

* To export NGINX Plus metrics:
    ```
    $ nginx-prometheus-exporter -nginx.plus -nginx.scrape-uri=http://<nginx-plus>:8080/api
    ```
    where `<nginx-plus>` is the IP address/DNS name, through which NGINX Plus is available.

* To export and scrape NGINX metrics with unix domain sockets, run:
    ```
    $ nginx-prometheus-exporter -nginx.scrape-uri=unix:<nginx>:/stub_status -web.listen-address=unix:/path/to/socket.sock
    ```
    where `<nginx>` is the path to unix domain socket, through which NGINX stub status is available.

**Note**. The `nginx-prometheus-exporter` is not a daemon. To run the exporter as a system service (daemon), configure the init system of your Linux server (such as systemd or Upstart) accordingly. Alternatively, you can run the exporter in a Docker container.

## Usage

### Command-line Arguments

```
Usage of ./nginx-prometheus-exporter:
  -nginx.plus
        Start the exporter for NGINX Plus. By default, the exporter is started for NGINX. The default value can be overwritten by NGINX_PLUS environment variable.
  -nginx.retries int
        A number of retries the exporter will make on start to connect to the NGINX stub_status page/NGINX Plus API before exiting with an error. The default value can be overwritten by NGINX_RETRIES environment variable.
  -nginx.retry-interval duration
        An interval between retries to connect to the NGINX stub_status page/NGINX Plus API on start. The default value can be overwritten by NGINX_RETRY_INTERVAL environment variable. (default 5s)
  -nginx.scrape-uri string
        A URI or unix domain socket path for scraping NGINX or NGINX Plus metrics.
        For NGINX, the stub_status page must be available through the URI. For NGINX Plus -- the API. The default value can be overwritten by SCRAPE_URI environment variable. (default "http://127.0.0.1:8080/stub_status")
  -nginx.ssl-ca-cert string
        Path to the PEM encoded CA certificate file used to validate the servers SSL certificate. The default value can be overwritten by SSL_CA_CERT environment variable.
  -nginx.ssl-client-cert string
        Path to the PEM encoded client certificate file to use when connecting to the server. The default value can be overwritten by SSL_CLIENT_CERT environment variable.
  -nginx.ssl-client-key string
        Path to the PEM encoded client certificate key file to use when connecting to the server. The default value can be overwritten by SSL_CLIENT_KEY environment variable.
  -nginx.ssl-verify
        Perform SSL certificate verification. The default value can be overwritten by SSL_VERIFY environment variable. (default true)
  -nginx.timeout duration
        A timeout for scraping metrics from NGINX or NGINX Plus. The default value can be overwritten by TIMEOUT environment variable. (default 5s)
  -prometheus.const-labels value
        A comma separated list of constant labels that will be used in every metric. Format is label1=value1,label2=value2... The default value can be overwritten by CONST_LABELS environment variable.
  -web.listen-address string
        An address or unix domain socket path to listen on for web interface and telemetry. The default value can be overwritten by LISTEN_ADDRESS environment variable. (default ":9113")
  -web.telemetry-path string
        A path under which to expose metrics. The default value can be overwritten by TELEMETRY_PATH environment variable. (default "/metrics")
  -web.secured-metrics
        Expose metrics using https. The default value can be overwritten by SECURED_METRICS variable.  (default false)
  -web.ssl-server-cert string
        Path to the PEM encoded certificate for the nginx-exporter metrics server(when web.secured-metrics=true). The default value can be overwritten by SSL_SERVER_CERT variable.
  -web.ssl-server-key string
        Path to the PEM encoded key for the nginx-exporter metrics server (when web.secured-metrics=true). The default value can be overwritten by SSL_SERVER_KEY variable.
  -version
        Display the NGINX exporter version. (default false)
```

### Exported Metrics

* Common metrics:
    * `nginxexporter_build_info` -- shows the exporter build information.
* For NGINX, the following metrics are exported:
    * All [stub_status](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html) metrics.
    * `nginx_up` -- shows the status of the last metric scrape: `1` for a successful scrape and `0` for a failed one.

    Connect to the `/metrics` page of the running exporter to see the complete list of metrics along with their descriptions.
* For NGINX Plus, the following metrics are exported:
    * [Connections](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_connections).
    * [HTTP](http://nginx.org/en/docs/http/ngx_http_api_module.html#http_).
    * [SSL](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_ssl_object).
    * [HTTP Server Zones](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_server_zone).
    * [Stream Server Zones](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_server_zone).
    * [HTTP Upstreams](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_upstream). Note: for the `state` metric, the string values are converted to float64 using the following rule: `"up"` -> `1.0`, `"draining"` -> `2.0`, `"down"` -> `3.0`, `"unavail"` –> `4.0`, `"checking"` –> `5.0`, `"unhealthy"` -> `6.0`.
    * [Stream Upstreams](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_upstream). Note: for the `state` metric, the string values are converted to float64 using the following rule: `"up"` -> `1.0`, `"down"` -> `3.0`, `"unavail"` –> `4.0`, `"checking"` –> `5.0`, `"unhealthy"` -> `6.0`.
    * [Stream Zone Sync](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_zone_sync).
    * `nginxplus_up` -- shows the status of the last metric scrape: `1` for a successful scrape and `0` for a failed one.
    * [Location Zones](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_location_zone).
    * [Resolver](http://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_resolver_zone).


    Connect to the `/metrics` page of the running exporter to see the complete list of metrics along with their descriptions. Note: to see server zones related metrics you must configure [status zones](https://nginx.org/en/docs/http/ngx_http_status_module.html#status_zone) and to see upstream related metrics you must configure upstreams with a [shared memory zone](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone).

### Troubleshooting

The exporter logs errors to the standard output. When using Docker, if the exporter doesn’t work as expected, check its logs using [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) command.

## Releases

For each release, we publish the corresponding Docker image at `nginx/nginx-prometheus-exporter` [DockerHub repo](https://hub.docker.com/r/nginx/nginx-prometheus-exporter/) and the binaries on the GitHub [releases page](https://github.com/nginxinc/nginx-prometheus-exporter/releases).

## Building the Exporter

You can build the exporter using the provided Makefile. Before building the exporter, make sure the following software is installed on your machine:
* make
* git
* Docker for building the container image
* Go for building the binary

### Building the Docker Image

To build the Docker image with the exporter, run:
```
$ make container
```

Note: go is not required, as the exporter binary is built in a Docker container. See the [Dockerfile](build/Dockerfile).

### Building the Binary

To build the binary, run:
```
$ make
```

Note: the binary is built for the OS/arch of your machine. To build binaries for other platforms, see the [Makefile](Makefile).

The binary is built with the name `nginx-prometheus-exporter`.

## Grafana Dashboard
The official Grafana dashboard is provided with the exporter for NGINX. Check the [Grafana Dashboard](./grafana/README.md) documentation for more information.

## Support

The commercial support is available for NGINX Plus customers when the NGINX Prometheus Exporter is used with NGINX Ingress Controller.

## License

[Apache License, Version 2.0](LICENSE).
