# Wi-Fi peer-to-peer overview
Wi-Fi peer-to-peer (P2P) allows Android 4.0 (API level 14) or later devices with the appropriate hardware to connect directly to each other via Wi-Fi without an intermediate access point (Android's Wi-Fi P2P framework complies with the Wi-Fi Alliance's Wi-Fi Direct™ certification program). Using these APIs, you can discover and connect to other devices when each device supports Wi-Fi P2P, then communicate over a speedy connection across distances much longer than a Bluetooth connection. This is useful for applications that share data among users, such as a multiplayer game or a photo sharing application.

Wi-Fi 点对点支持 Android 4.0 api 14 及以上设备，设备之间通过 Wi-Fi 连接不需要中间，调用API 可以发现设备和连接设备，这种技术比蓝牙的传输距离更长，传输速度更快。


The Wi-Fi P2P APIs consist of the following main parts:
Methods that allow you to discover, request, and connect to peers are defined in the WifiP2pManager class.
Listeners that allow you to be notified of the success or failure of WifiP2pManager method calls. When calling WifiP2pManager methods, each method can receive a specific listener passed in as a parameter.
Intents that notify you of specific events detected by the Wi-Fi P2P framework, such as a dropped connection or a newly discovered peer.

Wi-Fi P2P APIs 主要包含：
- 发现设备、请求连接、连接
- 发现、请求、连接设备的成功和失败的监听
- 还能检测连接是否断开，是否发现新的设备

## Create a Wi-Fi P2P application 
1. 添加权限
```
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```
