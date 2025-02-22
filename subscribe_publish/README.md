# Amazon Web Services IoT Examples

These examples are adaptations of some of the [AWS IoT C SDK](https://github.com/aws/aws-iot-device-sdk-embedded-C) examples.

The provisioning/configuration steps for these examples are the same, and are given in this README.

This README also contains some troubleshooting information for common problems found when connecting to AWS IoT.

# Provisioning/Configuration

There are some additional steps that need to be run before you can build this example.

The [Getting Started section of the AWS IoT Developer Guide](http://docs.aws.amazon.com/iot/latest/developerguide/iot-gs.html) lays out the steps to get started with AWS IoT.

To build and use this example, follow all the AWS IoT Getting Started steps from the beginning ("Sign in to the AWS Iot Console") up until "Configuring Your Device". For configuring the device, these are the steps:

# Authentication (Based on X.509 certificates)

### Device Authentication

AWS IoT can use AWS IoT-generated certificates or certificates signed by a CA certificate for device authentication. To use a certificate that is not created by AWS IoT, you must register a CA certificate. All device certificates must be signed by the CA certificate you register. Please refer to guide at https://docs.aws.amazon.com/iot/latest/developerguide/device-certs-your-own.html for step-by-step instructions to register custom X.509 certificates.

### Server Authentication

Server certificates allow devices to verify that they're communicating with AWS IoT and not another server impersonating AWS IoT. By default [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem) (signed by Amazon Trust Services Endpoints CA) is embedded in applications, for more information please refer to https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html#server-authentication

## Configuring Your Device

### Installing Private Key & Certificate

Please follow the instructions in the root directory for installing the device certificate. The device private key resides in the ECC608, and you do not need to install it.

### Find & Set AWS Endpoint Hostname

Your AWS IoT account has a unique endpoint hostname to connect to. To find it, open the AWS IoT Console and click the "Settings" button on the bottom left side. The endpoint hostname is shown under the "Custom Endpoint" heading on this page.

Run `make menuconfig` and navigate to `Component Config` -> `Amazon Web Service IoT Config` -> `AWS IoT MQTT Hostname`. Enter the host name here.

*Note: It may seem odd that you have to configure parts of the AWS settings under Component Config and some under Example Configuration.* The IoT MQTT Hostname and Port are set as part of the component because when using the AWS IoT SDK's Thing Shadow API (in examples or in other projects) the `ShadowInitParametersDefault` structure means the Thing Shadow connection will default to that host & port. You're not forced to use these config values in your own projects, you can set the values in code via the AWS IoT SDK's init parameter structures - `ShadowInitParameters_t` for Thing Shadow API or `IoT_Client_Init_Params` for MQTT API.

### (Optional) Set Client ID

Run `make menuconfig`. Under `Example Configuration`, set the `AWS IoT Client ID` to a unique value.

The Client ID is used in the MQTT protocol used to send messages to/from AWS IoT. AWS IoT requires that each connected device within a single AWS account uses a unique Client ID. Other than this restriction, the Client ID can be any value that you like. The example default should be fine if you're only connecting one ESP32 at a time.

In a production IoT app this ID would be set dynamically, but for these examples it is compiled in via menuconfig.

### (Optional) Locally Check The Root Certificate

The Root CA certificate provides a root-of-trust when the ESP32 connects to AWS IoT. We have supplied the root CA certificate already (in PEM format) in the file `main/certs/aws-root-ca.pem`.

If you want to locally verify that this Root CA certificate hasn't changed, you can run the following command against your AWS MQTT Host:

```
openssl s_client -showcerts -connect hostname:8883 < /dev/null
```

(Replace hostname with your AWS MQTT endpoint host.) The Root CA certificate is the last certificate in the list of certificates printed. You can copy-paste this in place of the existing `aws-root-ca.pem` file.

## Monitoring MQTT Data from the device

After flashing the example to your ESP32, it should connect to Amazon and start subscribing/publishing MQTT data.

The example code publishes MQTT data to the topic `test_topic/esp32`. Amazon provides a web interface to subscribe to MQTT topics for testing in the *AWS-IoT Core*:

* On the AWS IoT Core console, click " MQTT Test Client" in the menu on the left.
* Click "Subscribe to Topic"
* Enter "Subscription Topic" `test_topic/esp32`
* Click "Subscribe"

... you should see MQTT data published from the running example.
**Note**: This is not in the standard JSON format the AWS-IoT Core expects, so it may provide a warning message. This is expected and fine in this case.

To publish data back to the device:

* Click "Publish to Topic"
* Enter "Publish Topic" `test_topic/esp32`
* Enter a message in the payload field
* Click Publish


## Important Notes

We use the local component directory for the project at ```subscribe_publish\components ``` to store a slightly modified *esp-aws-iot* component. The only modification made is the additional requirement that *esp-cryptoauth* be compiled before *esp-aws-iot*. 

Additionally the *esp-cryptoauth* library has been modified to 1. work with esp-idf 5.x and 2. work with the crypto-coprocessor on the device.

## Troubleshooting

### Tips

* Raise the ESP debug log level to Debug in order to see messages about the connection to AWS, certificate contents, etc.

* Enable mbedTLS debugging (under Components -> mbedTLS -> mbedTLS Debug) in order to see even more low-level debug output from the mbedTLS layer.

* To create a successful AWS IoT connection, the following factors must all be present:
  - Endpoint hostname is correct for your AWS account.
  - Certificate & private key are both attached to correct Thing in AWS IoT Console.
  - Certificate is activated.
  - Policy is attached to the Certificate in AWS IoT Console.
  - Policy contains sufficient permissions to authorize AWS IoT connection.

### TLS connection fails

If connecting fails entirely (handshake doesn't complete), this usually indicates a problem with certification configuration. The error usually looks like this:

```
aws_iot: failed! mbedtls_ssl_handshake returned -0x7780
```

(0x7780 is the mbedTLS error code when the server sends an alert message and closes the connection.)

* Check your client private key and certificate file match a Certificate registered and **activated** in AWS IoT console. You can find the Certificate in IoT Console in one of two ways, via the Thing or via Certificates:
  - To find the Certificate directly, click on "Registry" -> "Security Certificates". Then click on the Certificate itself to view it.
  - To find the Certificate via the Thing, click on "Registry" -> "Things", then click on the particular Thing you are using. Click "Certificates" in the sidebar to view all Certificates attached to that Thing. Then click on the Certificate itself to view it.

Verify the Certificate is activated (when viewing the Certificate, it will say "ACTIVE" or "INACTIVE" near the top under the certificate name).

If the Certificate appears correct and activated, verify that you are connecting to the correct AWS IoT endpoint (see above.)

### TLS connection closes immediately

Sometimes connecting is successful (the handshake completes) but as soon as the client sends its `MQTT CONNECT` message the server sends back a TLS alert and closes the connection, without anything else happening.

The error returned from AWS IoT is usually -28 (`MQTT_REQUEST_TIMEOUT_ERROR`). You may also see mbedtls error `-0x7780` (server alert), although if this error comes during `mbedtls_ssl_handshake` then it's usually a different problem (see above).

In the subscribe_publish example, the error may look like this in the log:

```
subpub: Error(-28) connecting to (endpoint)...
```

In the thing_shadow example, the error may look like this in the log:

```
shadow: aws_iot_shadow_connect returned error -28, aborting...
```

This error implies the Certificate is recognised, but the Certificate is either missing the correct Thing or the correct Policy attached to it.

* Check in the AWS IoT console that your certificate is activated and has both a **security policy** and a **Thing** attached to it. You can find this in IoT Console by clicking "Registry" -> "Security Certificates", then click the Certificate. Once viewing the Certificate, you can click the "Policies" and "Things" links in the sidebar.

