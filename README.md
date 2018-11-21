# ipcam2mqtt

[![npm](https://img.shields.io/npm/v/ipcam2mqtt.svg?style=flat-square)](https://www.npmjs.com/package/ipcam2mqtt)
[![travis](https://img.shields.io/travis/svrooij/ipcam2mqtt.svg?style=flat-square)](https://travis-ci.org/svrooij/ipcam2mqtt)
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg?style=flat-square)](https://github.com/mqtt-smarthome/mqtt-smarthome)
[![Support me on Patreon][badge_patreon]][patreon]
[![PayPal][badge_paypal_donate]][paypal-donations]
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg?style=flat-square)](https://github.com/semantic-release/semantic-release)

This node.js application is a bridge between the your IP Cameras (with sound or motion detection) and a mqtt server. That way your can have your home respond to sound detection events.

It's intended as a building block in heterogenous smart home environments where an MQTT message broker is used as the centralized message bus. See [MQTT Smarthome on Github](https://github.com/mqtt-smarthome/mqtt-smarthome) for a rationale and architectural overview.

## Installation

Using ipcam2mqtt is really easy, but it requires at least [Node.js](https://nodejs.org/) v6 or higher. (This app is tested against the latest version and the last lts version).

`sudo npm install -g ipcam2mqtt`

## Usage

```plain
ipcam2mqtt 0.0.0-development
Usage: ipcam2mqtt [options]

Options:
  -m, --mqtt         mqtt broker url. See
                     https://github.com/svrooij/ipcam2mqtt#mqtt-url
                                                   [default: "mqtt://127.0.0.1"]
  -n, --name         instance name. used as mqtt client id and as topic prefix
                                                            [default: "cameras"]
  -p, --port         The port to run on                          [default: 8000]
  --timeout          The timeout in seconds for resetting back to inactive, -1
                     for no reset                                  [default: 10]
  -k, --keep-images  Set this if you want to keep the images in mqtt   [boolean]
  -h, --help         Show help                                         [boolean]
  -l, --logging      possiblevalues: "error", "warn","info","debug"
                                                               [default: "info"]
  --version          Show version number                               [boolean]
```

### MQTT Url

Use the MQTT url to connect to your specific mqtt server. Check out [mqtt.connect](https://github.com/mqttjs/MQTT.js#connect) for the full description.

```plain
Connection without port (port 1883 gets used)
[protocol]://[address] (eg. mqtt://127.0.0.1)

Connection with port
[protocol]://[address]:[port] (eg. mqtt://127.0.0.1:1883)

Secure connection with username/password and port
[protocol]://[username]:[password]@[address]:[port] (eg. mqtts://myuser:secretpassword@127.0.0.1:8883)
```

### Configure your cameras

You now have and FTP server running on your computer. Now you can configure the cameras to send FTP snapshots to it when it detects movement or sound. The username you supply will be used as the device name.

## Topics

Every message starts with the instance name (specified with the `-n` argument), which defaults to `cameras` so we'll asume the default.

### Connect messages

This bridge uses the `cameras/connected` topic to send retained connection messages. Use this topic to check your if your ipcam2mqtt bridge is still running.

- `0` or missing is not connected (set by will functionality).
- `1` is connected to mqtt, but have not received an image.
- `2` is connected to mqtt and received our first image from a camera.

### Motion detected

If there is motion detected (eg. a file is received over FTP), you will see two messages on your mqtt server.
A motion message on `cameras/username/motion` with the following properties

- `name` The username used for the connection
- `val` current state of the device. `active` or `inactive`
- `filename` The filename of the uploaded image
- `kind` The guessed kind of detection (based on the filename)
- `ts` timestamp of last update.

And an image message on `cameras/username/image`, this will just contain the raw image data. And can be displayed by various sources.

## Use [PM2](http://pm2.keymetrics.io) to run in background

If everything works as expected, you should make the app run in the background automatically. Personally I use PM2 for this. And they have a great [guide for this](http://pm2.keymetrics.io/docs/usage/quick-start/).

To start ipcam2mqtt with PM2, you have to use this command.

```bash
pm2 start ipcam2mqtt -x -- [regular-options]
# the -x -- part is to tell pm2 you want to specify arguments to the script. example:
pm2 start ipcam2mqtt -x -- -n cameras -m mqtt://your.mqtt.host:1883
```

## Docker

You can also run this bridge on docker. Be sure to specify your own mqtt connection string! This command connects port `8821` (you can change this) to the container where the bridge runs at `8021`.

You can also set the other properties by using the `-e "IPCAM2MQTT_...=newvalue"` argument. All the properties can be set with the prefix `IPCAM2MQTT_` followed by the full name.

```bash
docker run -d -e "IPCAM2MQTT_MQTT=mqtt://your.mqtt.nl:1883" -p 8821:8021 --name ipcam2mqtt svrooij/ipcam2mqtt:latest
# Open (and follow) the logs
docker logs ipcam2mqtt -f
```

## Special thanks

This bridge is inspired on [hue2mqtt.js](https://github.com/hobbyquaker/hue2mqtt.js) by [Sabastian Raff](https://github.com/hobbyquaker). That was a great sample on how to create a globally installed, command-line, something2mqtt bridge.

The actual FTP server part is mostly copied, improved and simplified from [mqtt-camera-ftpd](https://github.com/stjohnjohnson/mqtt-camera-ftpd/). It wasn't really working anymore and it wasn't a true CLI tool hence this new and improved version.

## Beer

This bridge took me a lot of hours to build, so I invite everyone using it to [Buy me a beer](https://svrooij.nl/buy-me-a-beer/)

[badge_paypal_donate]: https://svrooij.nl/badges/paypal_donate.svg
[badge_patreon]: https://svrooij.nl/badges/patreon.svg
[paypal-donations]: https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=T9XFJYUSPE4SG
[patreon]: https://www.patreon.com/svrooij