## Step 1: Setup
In this step, you will set up your "VMs" by running the `web` and `api` services
on your local machine.

## Goals
* Start `web` and `api` services.
* Allow `web` to call `api`.

## Tasks

### Web
In one terminal tab, start `web`:

```bash
./vms/start-web-darwin.sh
2020-10-20T14:09:57.046-0700 [INFO]  Starting service: name=web upstreamURIs=http://api upstreamWorkers=1 listenAddress=0.0.0.0:8080 service type=http
2020-10-20T14:09:57.046-0700 [INFO]  Adding handler for UI static files
2020-10-20T14:09:57.046-0700 [INFO]  Settings CORS options: allow_creds=false allow_headers=Accept,Accept-Language,Content-Language,Origin,Content-Type allow_origins=*
```

**Windows command:** `./vms/start-web-windows.ps1`

The logs say that the service is listening on `0.0.0.0:8080`. Let's open our
browser to [http://localhost:8080](http://localhost:8080):

```json
{
  "name": "web",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "192.168.0.127"
  ],
  "start_time": "2020-10-20T14:12:29.470122",
  "end_time": "2020-10-20T14:12:29.472339",
  "duration": "2.216562ms",
  "body": "Hello World",
  "upstream_calls": [
    {
      "uri": "http://api",
      "code": -1,
      "error": "Error communicating with upstream service: Get \"http://api/\": EOF"
    }
  ],
  "code": 500
}
```

We see that we get a response, but the `web` service can't talk to its upstream
dependency `api`: `Error communicating with upstream service: Get \"http://api/\": EOF`.

This makes sense because we haven't started `api` yet.


### Api
Start the `api` service now in another terminal tab:

```bash
sudo ./vms/start-api-darwin.sh
2020-10-20T14:14:53.360-0700 [INFO]  Starting service: name=api-vm upstreamURIs= upstreamWorkers=1 listenAddress=127.0.0.1:80 service type=http
2020-10-20T14:14:53.360-0700 [INFO]  Adding handler for UI static files
2020-10-20T14:14:53.361-0700 [INFO]  Settings CORS options: allow_creds=false allow_headers=Accept,Accept-Language,Content-Language,Origin,Content-Type allow_origins=*
```

**Windows command:** `./vms/start-api-windows.ps1`

**NOTE:** You must use `sudo` because we're binding to port `80`. This is necessary
to demonstrate our no-downtime migration later in the tutorial.

If we open our browser to [http://127.0.0.1/](http://127.0.0.1/) ([http://127.0.0.1:9090/](http://127.0.0.1:9090/) on Windows) we should see

```json
{
  "name": "api-vm",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "192.168.0.127"
  ],
  "start_time": "2020-10-20T14:17:40.530634",
  "end_time": "2020-10-20T14:17:40.530723",
  "duration": "88.931??s",
  "body": "Hello World",
  "code": 200
}
```

This means our `api` service is working!

### Web calls Api

**Windows users:** Skip to "Everything Working"

All that's left is for `web` to call `api`.

If you look at our start script in `vms/start-web-darwin.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

export NAME="web"
export LISTEN_ADDR="0.0.0.0:8080"

export UPSTREAM_URIS="http://api"

./$(dirname "$0")/../bin/web/web-darwin
```

You'll see `export UPSTREAM_URIS="http://api"`. This tells `web` to contact
the `api` service at `http://api`.

This url won't work right now because `http://api` doesn't resolve to the `api`
service. To get this to work, we need to edit our `/etc/hosts` file. This file
lets us configure what hostnames, e.g. `api`, route to which IP addresses, e.g. `127.0.0.1`.

Edit `/etc/hosts` with `sudo`:

```bash
sudo vim /etc/hosts
```

And add this line to the bottom:
```
127.0.0.1 api
```

Then save the file.

In your browser, go to [http://api](http://api). You should see the same response
as [http://127.0.0.1/](http://127.0.0.1/):

```json
{
  "name": "api-vm",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "192.168.0.127"
  ],
  "start_time": "2020-10-20T14:17:40.530634",
  "end_time": "2020-10-20T14:17:40.530723",
  "duration": "88.931??s",
  "body": "Hello World",
  "code": 200
}
```

## Everything Working

Now that we have:
- [x] `web` running on [http://localhost:8080](http://localhost:8080)
- [x] `api` running on [http://127.0.0.1](http://127.0.0.1)
- [x] `/etc/hosts` configured to route [http://api](http://api) to [http://127.0.0.1](http://127.0.0.1)

When we navigate to [http://localhost:8080](http://localhost:8080), we should
see that `web` calls `api` successfully:

```json
{
  "name": "web",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "192.168.0.127"
  ],
  "start_time": "2020-10-20T14:24:59.192036",
  "end_time": "2020-10-20T14:24:59.194831",
  "duration": "2.794649ms",
  "body": "Hello World",
  "upstream_calls": [
    {
      "name": "api-vm",
      "uri": "http://api",
      "type": "HTTP",
      "ip_addresses": [
        "192.168.0.127"
      ],
      "start_time": "2020-10-20T14:24:59.192933",
      "end_time": "2020-10-20T14:24:59.193110",
      "duration": "176.557??s",
      "headers": {
        "Content-Length": "258",
        "Content-Type": "text/plain; charset=utf-8",
        "Date": "Tue, 20 Oct 2020 21:24:59 GMT"
      },
      "body": "Hello World",
      "code": 200
    }
  ],
  "code": 200
}
```

We know it's working because `upstream_calls` has the response from `api`.

## Conclusion
We now have `web` and `api` running on our development machine. This mimics
running them on VMs.

## Next Step

Go to [2-dockerize](../2-dockerize/README.md).
