# Helium Router ![CI](https://github.com/helium/routerv3/workflows/CI/badge.svg?branch=master) ![BUILD_AND_PUSH_IMG](https://github.com/helium/routerv3/workflows/BUILD_AND_PUSH_IMG/badge.svg)

Helium's LoRa Network Server (LNS) backend.

## Usage

### Testing / Local

```
# Build
make docker-build

# Run
make docker-run

# Running tests
make docker-test

```

### Production

Image hosted on https://quay.io/repository/team-helium/router.

> The `docker-compose` in this repo is an example and only runs `Router` and `metrics server` (via prometheus) please see https://github.com/helium/console to run full LNS stack. 

```
# Build
docker-compose build --force-rm

# Up
docker-compose up -d

# Down
docker-compose down

# Tail logs
docker-compose logs -f --tail=20

# Get in container
docker exec -it helium_router bash
```

### Data

Data is stored in `/var/data`.

> **WARNING**: The `swarm_key` file in the `blockchain` directory is router's identity and linked to your `OUI` (and routing table). **DO NOT DELETE THIS EVER**.

### Logs

Logs are contained in `/var/data/log/router.log`, by default logs will rotate every day and will remain for 7 days.
### Config

Config is in `.env`. See `.env-template` for details.

Full config is in `config/sys.config.src`.

Router's deafult port for blockchain connection is `2154`.

Prometheus template config file in `prometheus-template.yaml`.
## CLI
Commands are run in the `routerv3` directory using a docker container.
> **_NOTE:_**  `sudo` may be required

```
docker exec -it helium_router _build/default/rel/router/bin/router [CMD]
```
Following commands are appending to the docker command above.

### Device Worker `device`

#### All Devices
```
device all
```

#### Info for a single device
```
device --id=<id>
```
##### Id Option
`--id`
Device IDs are binaries, but should be provided plainly.
```
# good
device --id=1234-5678-890
# bad
device --id=<<"1234-5678-890">>
```

#### Single Device Queue
```
device queue --id=<id>
```
##### Options
[ID Options](#id-option)
#### Clear Device's Queue
```
device queue clear --id=<id>
```
##### Options
[ID Options](#id-option)

#### Add to Device's Queue
```
device queue add --id=<id> [--payload=<content> --channel-name=<name> --port=<port> --ack]
```
##### Options
`--id`
[ID Options](#id-option)
`--payload [default: "Test cli downlink message"]`
Set custom message for downlink to device.

`--channel-name [default: "CLI custom channel"]`
Channel name Console will show for Integration.

`--port [default: 1]`
Port to downlink on.

`--ack [default: false]`
Boolean flag for requiring acknowledgement from the device.

#### Trace a device's logs
```
device trace --id=<id>
```
##### Options
[ID Options](#id-option)

#### Stop trancing  device's logs
```
device trace stop --id=<id>
```
##### Options
[ID Options](#id-option)

#### Force XOR Filter push
```
device xor
```
##### Options
XOR will only happen if `--commit` is used, otherwise it will be a dry run.

### DC Tracker `dc_tracker`

#### All Orgs
```
dc_tracker info all [--less 42] [--refetch]
```
##### Options
`--less <amount>`
Filter to Organizations that have a balance less than `<amount>`.

`--refetch`
Update Router information about organizations in list.

#### Info for 1 Org
```
dc_tracker info <org_id> [--refetch]
```
##### Options
`--refetch`
Update Router information about this organization.

#### Refill Org Balance
```
dc_tracker refill <org_id> balance=<balance> nonce=<nonce>
```

#### Helpers
```
dc_tracker info no-nonce
dc_tracker info no-balance
```
are aliases for

```
dc_tracker info nonce <amount=0>
dc_tracker info balance <balance=0>
```

## Integrations Data

Data payload example sent to integrations

```
{
    "id": "device_uuid",
    "name": "device_name",
    "dev_eui": "dev_eui",
    "app_eui": "app_eui",
    "metadata": {},
    "fcnt": 2,
    "reported_at": 123,
    "payload": "base64 encoded payload",
    "payload_size": 22,
     // ONLY if payload is decoded
    "decoded": {
        "status": "success | error",
        "error": "...",
        "payload": "..."
    },
    "port": 1,
    "devaddr": "devaddr",
    "hotspots": [
        {
            "id": "hotspot_id",
            "name": "hotspot name",
            "reported_at": 123,
            "status": "success | error",
            "rssi": -30,
            "snr": 0.2,
            "spreading": "SF9BW125",
            "frequency": 923.3,
            "channel": 12,
            // WARNING: if the hotspot is not found (or asserted) in the chain the lat/long will come in as a string "unknown"
            "lat": 37.00001962582851,
            "long": -120.9000053210367
        }
    ],
    "dc" : {
        "balance": 3000,
        "nonce": 2
    }
}
```

## Metrics

Router includes metrics that can be retrieved via [prometheus](https://prometheus.io/docs/introduction/overview/).

### Enable metrics

To enable Router's metrics you will need to add a new container for Prometheus, you can find an example of the setup in any of the `docker-compose` files.
You can also get a basic configuration for prometheus in `prometheus-template.yml`, [more config here](https://prometheus.io/docs/prometheus/latest/configuration/configuration/).

### Available metrics

- `router_blockchain_blocks` Gauge, difference between Router's local blockchain and Helium's main API (a negative number means Router is ahead).
- `router_console_api_duration` Histogram,  caculate time for API calls made to console.
- `router_dc_balance` Gauge, how many DC are left in Router's account.
- `router_decoder_decoded_duration` Histogram, calculate decoders running time (with status).
- `router_device_downlink_packet` Counter, count number of downlinks (with their origin).
- `router_device_packet_trip_duration` Histogram, time taken by an uplink (or join) from offer to packet handling (potentially including downlink).
- `router_device_routing_offer_duration` Histogram, time to handle any type of offer (include reason for success or failure).
- `router_device_routing_packet_duration` Histogram, time to handle any type of packet (include reason for success or failure).
- `router_function_duration` Histogram, time for some specific function to run.
- `router_state_channel_active` Gauge, active state channel balance.
- `router_state_channel_active_count` Gauge, number of open state channels.
- `router_vm_cpu` Gauge, individual CPU usage
- `router_ws_state` Gauge, websocket connection to Console (1 connected, 0 disconnected).
- `erlang_vm_memory_*` Gauge, Erlang internal memory usage.
- `erlang_vm_process_count` Gauge, number of processes running in the Erlang VM.

Note that any [histogram](https://prometheus.io/docs/concepts/metric_types/#histogram) will include `_count`, `_sum` and `_bucket`.


### Displaying metrics

##### Grafana

Here are some of the most commun queries that can be added into grafana.

`sum(rate(router_device_routing_packet_duration_count{type="packet", status="accepted"}[5m]))` Rate of accepted packet.

`rate(router_device_routing_offer_duration_sum{status="accepted"}[5m])/rate(router_device_routing_offer_duration_count{status="accepted"}[5m])` Time (in ms) to handle offer (accepted)

`router_dc_balance{}` Simple count of your Router's balance

`sum(rate(router_console_api_duration_count{status="ok"}[5m])) / (sum(rate(router_console_api_duration_count{status="error"}[5m])) + sum(rate(router_console_api_duration_count{status="ok"}[5m]))) * 100` API success rate
