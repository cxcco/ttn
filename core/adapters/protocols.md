semtech ~ udp
=============
Have a look at [this document](http://iot.semtech.com/resources/Server_Release_2.1.1/files.xml?action=download&file=LoRa%20gateway%20to%20network%20server%20interface%20definition.pdf)

basic ~ http
============
The basic http protocol relies seemingly on `http`. 

An adapter which implements this protocol should provide at least one end-point:

- `[POST] /packets` 


#### Request 

Packets are sent as a json payload of the following shape:

```js
    {
        "payload": <base64Encoded Lorawan Physical payload>,
        "metadata": {
            "chan": ..., // Concentrator "IF" channel used for RX (unsigned integer)
            "codr": ..., // LoRa ECC coding rate identifier
            "datr": ..., // LoRa datarate identifier
            "fdev": ..., // FSK frequency deviation (unsigned integer, in Hz)
            "freq": ..., // RX Central frequency in MHx (unsigned float, Hz precision)
            "imme": ..., // Send packet immediately (will ignore tmst & time)
            "ipol": ..., // Lora modulation polarization inversion
            "lsnr": ..., // LoRa SNR ratio in dB (signed float, 0.1 dB precision)
            "modu": ..., // Modulation identifier "LORA"
            "ncrc": ..., // If true, disable the CRC of the physical layer (optional)
            "powe": ..., // TX output power in dBm (unsigned integer, dBm precision)
            "prea": ..., // RF preamble size (unsigned integer)
            "rfch": ..., // Concentrator "RF chain" used for RX (unsigned integer)
            "rssi": ..., // RSSI in dBm (signed integer, 1 dB precision)
            "size": ..., // RF packet payload size in bytes (unsigned integer)
            "stat": ..., // CRC status: 1 - OK, -1 = fail, 0 = no CRC
            "time": ..., // UTC time of pkt RX, us precision, ISO 8601 'compact' format
            "tmst": ...  // Internal timestamp of "RX finished" event (32b unsigned)
        }
    }
```

All fields in metadata are optional, so is the metadata field itself. The payload should be a
base64 encoded binary representation of a Physical Payload as defined by the
[lorawan](https://github.com/brocaar/lorawan) go package

#### Response

The adapter may provide two answers to the demander. 

- An `HTTP 200 Ok.` means that the packet has been accepted and is handled by the server. 

- An `HTTP 404 Not Found.` means that the server doesn't take care of packet coming from the
  end-device related to the packet.

Another type of response could be misinterpreted by the sender. An `404` response doesn't
contain any body payload. However, a `200` might. In such a case, the response has the same
shape as the one described above: a plain `json` with an encoded physical payload and some
possible metadata.

basic+pubsub ~ http
===================

The `pubsub` http adapter is an extension of the `basic` http adapter. In addition of the
behavior defined in the corresponding section, the `pubsub` adapter also provide the following
end-point:

- `[PUT] /end-devices/:devAddr`

where `:devAddr` identify a device address encoded as an hexadecimal string of 8 characters (2
characters for a single byte), for instance: "09a3bc52".

This end-point is used to register a handler for a given end-device such that every packet of
the network coming from that device will be forwarded via http to the handler.

#### Request

Requests are expected to come along with a `json` payload of the following shape:

```js
    {
        "app_id": ..., // Application identifier represented by a string
        "app_url": ..., // Webhook to which forward incoming data
        "nws_key": ..., // The network session key associated to the device.
    }
```


#### Response




basic+broadcast ~ http
======================