# ESP-AWS-IoT

This project is a fork of esp-aws-iot from Espressif, and it contain several modifications including support of ESP-IDF versions release/v4.2 and release/v4.3. Also, there is native support for ATECC608A TrustCustom, while other chip types (Trust&Go, TrustFlex) are not currently supported.


## Getting Started

- Installation instructions for ESP-IDF version release/v4.2 are here: https://docs.espressif.com/projects/esp-idf/en/release-v4.2/esp32/get-started/index.html

- Please clone this repository using,
    ```
    git clone --recursive https://github.com/PBearson/esp-aws-iot
    ```
## Using an ATECC608A with the ESP-AWS-IoT
   The `esp-aws-iot` contains the support for using a `secure element` chip from microchip i.e. `ATECC608A`. This will store the private key used for the TLS communication in the `secure element` which has the hardware security. The module [ESP32-WROOM-32SE](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32se_datasheet_en.pdf) which integrates the `ATECC608A`, can be readily used for this purpose. To use the `secure element` for the TLS, configure the project with help of the following steps.
> To use the `secure element`(ATECC608A) for the TLS connection, it must be already configured. Please refer to the [eps_cryptoauth_utility](https://github.com/espressif/esp-cryptoauthlib/tree/master/esp_cryptoauth_utility) for more details.
### Project Configuration

1. Add [esp-cryptoauthlib](https://github.com/espressif/esp-cryptoauthlib) as a component to your project with the help of following commands,
    (First change directory (cd) to your project directory)
```
    mkdir components
    cd components
    git clone https://github.com/espressif/esp-cryptoauthlib.git
```
2. Enable the mbedtls ECDSA support in esp-cryptoauthlib with the following config (`idf.py menuconfig`) option,
    `menuconfig->Component config->esp-cryptoauthlib->Enable Hardware ECDSA keys for mbedTLS`

3. Enable support for hardware secure element with the following config (`idf.py menuconfig`) option,
    `menuconfig->Component config->Amazon Web Services IoT Platform->Use the hardware secure element for authenticating TLS connections`

4. Ensure that you call the function `atcab_init` in the application to initialise the `secure_element` before you use the AWS IoT component.

5. Project TLS configuration -

    i) If you have the `Trust&GO` type of ATECC608A chip then you can set `mqttInitParams.pDeviceCertLocation = "#"` in your project. It will make the `esp-aws-iot` read the certificate from `ATECC608A` and use it for the TLS. For the other types of ATECC608A chips, you will need to provide the certificate externally (embed the certificate).

    ii) Tell the `esp-aws-iot` to find the client private key in a slot by setting `mqttInitParams.pDevicePrivateKeyLocation = “#0”` in your project, where the digit 0 indicates the slot in the ATECC608A in which the private key is stored.  In most circumstances, the key will be stored in slot 0.
> Along with the secure element configurations you will have to perform additional example specific configurations (if required).
