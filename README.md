# Keyboard application

## Overview
The Keyboard application is a single chip SoC. It can be built to BLE and/or classic BT keyboard application. It provides a turnkey solution using on-chip keyscan HW component. It can operate in both BR/EDR Bluetooth mode and LE, HID over GATT Profile (HOGP).

When start pairing, it operates in BR/EDR mode first. During pairing, if user re-start pairing again, it switches to LE mode pairing. If user re-start again during LE mode pairing, it stops pairing. The keyboard will operate in either BR/EDR or LE mode based on last paired host.

During initialization the app registers with both LE and BR/EDR stack, WICED HID Device Library and keyscan HW to receive various notifications including bonding complete, connection status change, peer GATT request/commands, SDP protocol, and interrupts for key pressed/released.

If the device is not paired, press any key will start pairing. When device is successfully bonded, the app saves bonded host's information in the NVRAM.

When user presses/releases key, a key report will be sent to the host. On connection up or battery level changed, a battery report will be sent to the host. When battery level is below shutdown voltage, device will do critical shutdown. Host can send LED report to the device to control LED.

## Features demonstrated
- BR/EDR Bluetooth operation
- SDP protocol support
- GATT database and Device configuration initialization
- Registration with LE stack for various events
- Sending HID reports to the host
- Processing write requests from the host
- Low power management
- Over the air firmware update (OTAFWU)

## Instructions
To demonstrate the app, walk through the following steps.
1. Plug the keyboard HW into your computer
2. Build and download the application
3. Unplug the keyboard HW from your computer and power cycle the keyboard HW
4. Press 'Lock' key to start BR/EDR pairing, then pair with a PC or Tablet. The Lock key is located at right top most cornor.
5. To pair with LE host, during BR/EDR pairing, press Lock key again to stop BR/EDR pairing and start LE advertisment.
6. Once connected, it becomes the keyboard of the PC or Tablet.

In case you don't have the right hardware, reference keyboard, platform CYW920819REF-KB-01, which is required to support the 8x18 key matrix used, you can build evaluation board version in conjunction to work with ClientControl. You can choose CYW920819EVB-02, CYW920820EVB-02, or CYW920735Q60EVB-01 platfrom. In this case, the key-matrix will not be functioning correctly due to the lack of key-matrix pin assignment and eval board perpheral pins conflicts. However, you can test the basic Bluetooth functions and to mimic to send key report by using ClientControl.

NOTE: To use Client Control, make sure you use "TESTING\_USING\_HCI=1" in application settings. 208xx device and 20735 device firmware behaves differently. The following steps shows how to establish communication between ClientControl and the device.

For 20819/20820 devices:
1. Plug the hardware into your computer
2. Build and download the application
3. Run ClientControl.exe.
4. Choose 3M as Baudrate and select the serial port in ClientControl tool window.
5. Open the port and then reset the device. Close and re-open the port so the HID tab gets activated.

For 20735/20835 devices:
1. Plug the hardware into your computer
2. Build and download the application
3. Run ClientControl.exe.
4. Choose 3M as Baudrate and select the serial port in ClientControl tool window.
5. Reset the device. (Press reset button or unplug/plug the USB cable). Within 2 seconds, before the device enters deep sleep, open the port. If HIDD tab is not activated, close the port and repeat step 5. Once the HID tab is activated, the HID buttons will become available.
6. Click on "Enter Pairing Mode" to start BR/EDR pairing, then pair with a PC or Tablet. The "Enter/Exit Pairing Mode" click button is acting as 'Lock' key on the keyboard. Click on "Exit Pairing Mode" while it is in BR/EDR pairing, it switches to LE advertising for pairing. Click one more time while it is in LE advertising, it will exit pairing mode.
7. Once connected, it becomes the keyboard of the PC or Tablet.
8  Click on the key buttons, to send the key reports.  For example to send key down event when key '1' is pushed, report should be 01 00 00 1e 00 00 00 00 00. When key is released, it should send all keys up 01 00 00 00 00 00 00 00 00.

## Notes
The application GATT is located in bt/ble.h and SDP databases is located in bt/bredr.h. If you create a GATT or SDP database using Bluetooth Configurator, update the database in the location mentioned above.

## Application Settings
- LE
    - Use this option to enable LE Bluetooth compliant with HID over GATT Profile (HOGP). The following options are available only when LE is enabled.

- DISCONNECTED\_ENDLESS\_ADV
    - Use this option to enable disconnected endless advertisement. When this option is used, the device will do advertising forever until it is connected. To conserve power, it allows SDS/ePDS and do the advertising in a long interval. If AUTO\_RECONNECT option is not set, then pressing a key will try to reconnect and stays in adv forever until it is connected.

- SKIP\_PARAM\_UPDATE
    - Use this option to skip to send link parameter update request. When this option is disabled, if the peer device (master) assigned link parameter is not within the device's preferred range, the device will send a request for the desired link parameter change. This option can be enabled to stop the device from sending the reuqest and accept the given link parameter as is
    - In some OS (peer host), after link is up, it continuously sends different parameter of LINK\_PARAM\_CHANGE over and over for some time. When the parameter is not in our device preferred range, the firmware was rejecting and renegotiating for new preferred parameter. It can lead up to endless and unnecessary overhead in link parameter change. Instead of keep rejecting the link parameter, by using this option, we accept peer requested link parameter as it and starts a timer to send the final link parameter change request later when the peer host settles down in link parameter change.

- ASSYMETRIC\_SLAVE\_LATENCY
    - Use this option to enable assymetric slave latency.
    - In early days, some HID host devices will always reject HID slave's link parameter update request. Because of this, HID device will end up consuming high power when slave latency was short. To work around this issue, we use Asymmetric Slave Latency method to save power by waking up only at multiple time of the communication anchor point. When this option is enabled,

        1.  We do not send LL\_CONNECTION\_PARAM\_REQ.
        2.  We simply start Asymmetric Slave Latency by waking up at multiple times of given slave latency.

    - Since this is not a standard protocol, we do not recommend enabling this option unless if it is necessary to save power to work around some HID hosts.

- LE\_LOCAL\_PRIVACY
    - When enabled, the device uses RPA (Random Private Address). When disabled, the device uses Public static address.


- BREDR
    - Use this option to enable classic BR/EDR.
    - Since 20735 device does not support BREDR. If 20735 TARGET is used, this option will forced to turn off.

- TESTING\_USING\_HCI
    - Use this option for testing with Bluetooth Profile Client Control. The Client Control UI can be used to provide input. When this option is enabled, the device will not enter SDS/ePDS for power saving.

- OTA\_FW\_UPGRADE
    - Use this option for enabling firmware upgrade over the Air (OTA) capability. The host pier tool applications in wiced\_btsdk\tools\btsdk-peer-apps-ota can be used to upgrade OTA firmware image. The OTA firmware image with extention *.ota.bin is created under built folder. Due to limiited internal flash space, the 20819/20820 device must use external flash for OTA firmware upgrade to be functional. OTAFWU is not supported in CYW920819EVB-02 and CYW920820EVB-02 targets.

- OTA\_SEC\_FW\_UPGRADE
    - Use this option for secure OTA firmware upgrade. OTA\_FW\_UPGRADE option must be enabled for this option to take effect. The OTA firmware image with extention *.ota.bin.signed must be used for OTA firmware upgrade. Please follow the instruction in wiced\_btsdk\tools\btsdk-peer-apps-ota\readme.txt to create *.ota.bin.signed file image.

- AUTO\_RECONNECT
    - Use this option to enable auto reconnect. By enabling this option, the device will always stay connected. If it is disconnected, it try to reconnect until it is connected.
    - This option should be used together with DISCONNECTED\_ENDLESS\_ADV. When this option is enabled, the HID device will always try to maintain connection with the paired HID host; therefore, if the link is down, it will continuously try to reconnect. To conserve power, it should allow entering SDS/ePDS while advertising; thus, the DISCONNECTED\_ENDLESS\_ADV option should be enabled; otherwise, it may drain battery quickly if host was not available to reconnect.

- SLEEP\_ALLOWED
    - Use this to set sleep option

- LED
    - Use this option to turn on/off LED function (Useful when turned off for power measurement)

## Note:
When testing with Client Control on CYW920819EVB-02, please do not pair to a host that runs Client Control. You will not see the key presses. The reason is because with CYW920819EVB-02 platform, since the key matrix is disabled, the only way to send a character is by using simulated Client Control buttons. The moment you click on the key button, the Windows focus on Client Control application. The text editor loses focus. The character sent to host will be delivered to the focused application, Client Control. Client Control ignores key '1', '2', '3', or 'a', 'b', 'c'. To send the character to text editor, you will need another PC (host) to pair to CYW920819EVB-02 device so when you click on button in Client Control, the other PC (host) focused on text editor application does not lose focus and can receive the key.

## Common application settings

Application settings below are common for all BTSDK applications and can be configured via the makefile of the application or passed in via the command line.

- BT\_DEVICE\_ADDRESS<br/>
    - Set the BDA (Bluetooth Device Address) for your device. The BT address is 6 bytes, for example, 20819A10FFEE. By default, the SDK will set a BDA for your device by combining the 7 hex digit device ID with the last 5 hex digits of the host PC MAC address.

- UART<br/>
    - Set to the UART port you want to use to download the application. For example 'COM6' on Windows or '/dev/ttyWICED\_HCI\_UART0' on Linux or '/dev/tty.usbserial-000154' on macOS. By default, the SDK will auto-detect the port.

- ENABLE_DEBUG<br/>
    - For HW debugging, select the option '1'. See the document [WICED-Hardware-Debugging](https://github.com/cypresssemiconductorco/btsdk-docs/blob/master/docs/BT-SDK/WICED-Hardware-Debugging.pdf) for more information. This setting configures GPIO for SWD.
      - CYW920819EVB-02/CYW920820EVB-02: SWD signals are shared with D4 and D5, see SW9 in schematics.
      - CYBT-213043-MESH/CYBT-213043-EVAL: SWD signals are routed to P12=SWDCK and P13=SWDIO. Use expansion connectors to connect VDD, GND, SWDCK, and SWDIO to your SWD Debugger probe.
      - CYW989820EVB-01: SWDCK (P02) is routed to the J13 DEBUG connector, but not SWDIO. Add a wire from J10 pin 3 (PUART CTS) to J13 pin 2 to connect GPIO P10 to SWDIO.
      - CYW920719B2Q40EVB-01: PUART RX/TX signals are shared with SWDCK and SWDIO. Remove RX and TX jumpers on J10 when using SWD. PUART and SWD cannot be used simultaneously on this board unless these pins are changed from the default configuration.
      - CYW920721B2EVK-02: SWD hardware debugging supported. SWD signals are shared with D4 and D5, see SW9 in schematics.
      - CYW920721B2EVK-03: SWD hardware debugging is not supported.
      - CYW920706WCDEVAL: SWD debugging requires fly-wire connections. The default setup uses P15 (J22 pin 3) for SWDIO and P30 (J19 pin 2) for SWDCK. P30 is shared with BTN1.
      - CYW920735Q60EVB-01: SWD hardware debugging supported. The default setup uses the J13 debug header, P3 (J13 pin 2) for SWDIO and P2 (J13 pin 4) for SWDCK.  They can be optionally routed to D4 and D4 on the Arduino header J4, see SW9 in schematics.
      - CYW920835REF-RCU-01: SWD hardware debugging is not supported.
      - CYW9M2BASE-43012BT: SWD hardware debugging is not supported.

## Building code examples

**Using the ModusToolbox IDE**

1. Install ModusToolbox 2.2.
2. In the ModusToolbox IDE, click the **New Application** link in the Quick Panel (or, use **File > New > ModusToolbox IDE Application**).
3. Pick your board for BTSDK under WICED Bluetooth BSPs.
4. Select the application in the IDE.
5. In the Quick Panel, select **Build** to build the application.
6. To program the board (download the application), select **Program** in the Launches section of the Quick Panel.


**Using command line**

1. Install ModusToolbox 2.2
2. On Windows, use Cygwin from \ModusToolbox\tools_2.2\modus-shell\Cygwin.bat to build apps.
3. Use the tool 'project-creator-cli' under \ModusToolbox\tools_2.2\project-creator\ to create your application.<br/>
   > project-creator-cli --board-id (BSP) --app-id (appid) -d (dir) <br/>
   See 'project-creator-cli --help' for useful options to list all available BSPs, and all available apps per BSP.<br/>
   For example:<br/>
   > project-creator-cli --app-id mtb-example-btsdk-empty --board-id CYW920706WCDEVAL -d .
4. To build the app call make build. For example:<br/>
   > cd mtb-examples-btsdk-empty<br/>
   > make build<br/>
6. To program (download to) the board, call:<br/>
   > make qprogram<br/>
7. To build and program (download to) the board, call:<br/>
   > make program<br/>

   Note: make program = make build + make qprogram


## Downloading an application to a board

If you have issues downloading to the board, follow the steps below:

- Press and hold the 'Recover' button on the board.
- Press and hold the 'Reset' button on the board.
- Release the 'Reset' button.
- After one second, release the 'Recover' button.

Note: this is only applicable to boards that download application images to FLASH storage. Boards that only support RAM download (DIRECT_LOAD) such as CYW9M2BASE-43012BT can be power cycled to boot from ROM.

## SDK software features

- Dual-mode Bluetooth stack included in the ROM (BR/EDR and BLE)
- BT stack and profile level APIs for embedded BT application development
- WICED HCI protocol to simplify host/MCU application development
- APIs and drivers to access on-board peripherals
- Bluetooth protocols include GAP, GATT, SMP, RFCOMM, SDP, AVDT/AVCT, BLE Mesh
- BLE and BR/EDR profile APIs, libraries, and sample apps
- Support for Over-The-Air (OTA) upgrade
- Device Configurator for creating custom pin mapping
- Bluetooth Configurator for creating BLE GATT Database
- Peer apps based on Android, iOS, Windows, etc. for testing and reference
- Utilities for protocol tracing, manufacturing testing, etc.
- Documentation for APIs, datasheets, profiles, and features
- BR/EDR profiles: A2DP, AVRCP, HFP, HSP, HID, SPP, MAP, PBAP, OPP
- BLE profiles: Mesh profiles, HOGP, ANP, BAP, HRP, FMP, IAS, ESP, LE COC
- Apple support: Apple Media Service (AMS), Apple Notification Center Service (ANCS), iBeacon, Homekit, iAP2
- Google support: Google Fast Pair Service (GFPS), Eddystone
- Amazon support: Alexa Mobile Accessories (AMA)

Note: this is a list of all features and profiles supported in BTSDK, but some WICED devices may only support a subset of this list.

## List of boards available for use with BTSDK

- CYW20819A1 chip: CYW920819EVB-02, CYBT-213043-MESH, CYBT-213043-EVAL, CYW920819REF-KB-01
- CYW20820A1 chip: CYW920820EVB-02, CYW989820EVB-01
- CYW20721B2 chip: CYW920721B2EVK-02, CYW920721B2EVK-03, CYW920721M2EVK-01, CYW920721M2EVK-02, CYBT-423060-EVAL, CYBT-483062-EVAL, CYBT-413061-EVAL
- CYW20719B2 chip: CYW920719B2Q40EVB-01, CYBT-423054-EVAL, CYBT-413055-EVAL, CYBT-483056-EVAL
- CYW20706A2 chip: CYW920706WCDEVAL, CYBT-353027-EVAL, CYBT-343026-EVAL
- CYW20735B1 chip: CYW920735Q60EVB-01
- CYW20835B1 chip: CYW920835REF-RCU-01
- CYW43012C0 chip: CYW9M2BASE-43012BT, CYW9M2BASE-43012BT20

## Folder structure

All BTSDK code examples need the 'mtb\_shared\wiced\_btsdk' folder to build and test the apps. 'wiced\_btsdk' includes the 'dev-kit' and 'tools' folders. The contents of the 'wiced\_btsdk' folder will be automatically populated incrementally as needed by the application being used.

**dev-kit**

This folder contains the files that are needed to build the embedded BT apps.

* baselib: Files for chips supported by BTSDK. For example CYW20819, CYW20719, CYW20706, etc.

* bsp: Files for BSPs (platforms) supported by BTSDK. For example CYW920819EVB-02, CYW920721B2EVK-02, CYW920706WCDEVAL etc.

* btsdk-include: Common header files needed by all apps and libraries.

* btsdk-tools: Build tools needed by BTSDK.

* libraries: Profile libraries used by BTSDK apps such as audio, BLE, HID, etc.

**tools**

This folder contains tools and utilities need to test the embedded BT apps.

* btsdk-host-apps-bt-ble: Host apps (Client Control) for BLE and BR/EDR embedded apps, demonstrates the use of WICED HCI protocol to control embedded apps.

* btsdk-host-peer-apps-mesh: Host apps (Client Control) and Peer apps for embedded Mesh apps, demonstrates the use of WICED HCI protocol to control embedded apps, and configuration and provisioning from peer devices.

* btsdk-peer-apps-ble: Peer apps for embedded BLE apps.

* btsdk-peer-apps-ota: Peer apps for embedded apps that support Over The Air Firmware Upgrade.

* btsdk-utils: Utilities used in BTSDK such as BTSpy, wmbt, and ecdsa256.

See README.md in the sub-folders for more information.

## ModusToolbox Tools

Source code generation tools installed by ModusToolbox installer:

- Device Configurator:
      A GUI tool to create custom pin mappings for your device.
- Bluetooth Configurator:
      A GUI tool to create and configure the BLE GATT Database and BR/EDR SDP records for your application.

## Using BSPs (platforms)

BTSDK BSPs are located in the \mtb\_shared\wiced\_btsdk\dev-kit\bsp\ folder by default.

#### a. Selecting an alternative BSP

The application makefile has a default BSP. See "TARGET". The makefile also has a list of other BSPs supported by the application. See "SUPPORTED_TARGETS". To select an alternative BSP, use Library Manager from the Quick Panel to deselect the current BSP and select an alternate BSP. Then right-click the newly selected BSP and choose 'Set Active'.  This will automatically update TARGET in the application makefile.

#### b. Custom BSP

**Complete BSP**

To create and use a complete custom BSP that you want to use in applications, perform the following steps:

1. Select an existing BSP you wish to use as a template from the list of supported BSPs in the mtb\_shared\wiced\_btsdk\dev-kit\bsp\ folder.
2. Make a copy in the same folder and rename it. For example mtb\shared\wiced\_btsdk\dev-kit\bsp\TARGET\_mybsp.<br/>
   **Note:** This can be done in the system File Explorer and then refresh the workspace in Eclipse to see the new project.  Delete the .git sub-folder from the newly copied folder before refreshing in Eclipse.
   If done in the IDE, an error dialog may appear complaining about items in the .git folder being out of sync.  This can be resolved by deleting the .git sub-folder in the newly copied folder.

3. In the new mtb\_shared\wiced\_btsdk\dev-kit\bsp\TARGET\_mybsp folder, rename the existing/original (BSP).mk file to mybsp.mk.
4. In the application makefile, set TARGET=mybsp and add it to SUPPORTED\_TARGETS as well as TARGET\_DEVICE\_MAP.  For example: mybsp/20819A1
5. Update design.modus for your custom BSP if needed using the **Device Configurator** link under **Configurators** in the Quick Panel.
6. Update the application makefile as needed for other custom BSP specific attributes and build the application.

**Custom Pin Configuration Only - Multiple Apps**

To create a custom pin configuration to be used by multiple applications using an existing BSP that supports Device Configurator, perform the following steps:

1. Create a folder COMPONENT\_(CUSTOM)\_design\_modus in the existing BSP folder. For example mtb\_shared\wiced\_btsdk\dev-kit\bsp\TARGET\_CYW920819EVB-02\COMPONENT\_my\_design\_modus
2. Copy the file design.modus from the reference BSP COMPONENT\_bsp\_design\_modus folder under mtb\_shared\wiced\_btsdk\dev-kit\bsp\ and place the file in the newly created COMPONENT\_(CUSTOM)\_design\_modus folder.
3. In the application makefile, add the following two lines<br/>
   DISABLE\_COMPONENTS+=bsp\_design\_modus<br/>
   COMPONENTS+=(CUSTOM)\_design\_modus<br/>
   (for example COMPONENTS+=my\_design\_modus)
4. Update design.modus for your custom pin configuration if needed using the **Device Configurator** link under **Configurators** in the Quick Panel.
5. Building of the application will generate pin configuration source code under a GeneratedSource folder in the new COMPONENT\_(CUSTOM)\_design\_modus folder.

**Custom Pin Configuration Only - Per App**

To create a custom configuration to be used by a single application from an existing BSP that supports Device Configurator, perform the following steps:

1. Create a folder COMPONENT\_(BSP)\_design\_modus in your application. For example COMPONENT\_CYW920721B2EVK-03\_design\_modus
2. Copy the file design.modus from the reference BSP under mtb\_shared\wiced\_btsdk\dev-kit\bsp\ and place the file in this folder.
3. In the application makefile, add the following two lines<br/>
   DISABLE\_COMPONENTS+=bsp\_design\_modus<br/>
   COMPONENTS+=(BSP)\_design\_modus<br/>
   (for example COMPONENTS+=CYW920721B2EVK-03\_design\_modus)
4. Update design.modus for your custom pin configuration if needed using the **Device Configurator** link under **Configurators** in the Quick Panel.
5. Building of the application will generate pin configuration source code under the GeneratedSource folder in your application.


## Using libraries

The libraries needed by the app can be found in in the mtb\_shared\wiced\_btsdk\dev-kit\libraries folder. To add an additional library to your application, launch the Library Manager from the Quick Panel to add a library. Then update the makefile variable "COMPONENTS" of your application to include the library. For example:<br/>
   COMPONENTS += fw\_upgrade\_lib


## Documentation

BTSDK API documentation is available [online](https://cypresssemiconductorco.github.io/btsdk-docs/BT-SDK/index.html)

Note: For offline viewing, git clone the [documentation repo](https://github.com/cypresssemiconductorco/btsdk-docs)

BTSDK Technical Brief and Release Notes are available [online](https://community.cypress.com/community/software-forums/modustoolbox-bt-sdk)
