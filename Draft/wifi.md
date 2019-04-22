# Wi-Fi scanning overview

## Wi-Fi scanning process / Wi-Fi 扫描 3 步走
1. Register a broadcast listener
2. Request a scan
3. Get scan results

Kotlin 的写法
```
//context AS会建议使用 applicationContext
val wifiManager = context.getSystemService(Context.WIFI_SERVICE) as WifiManager

val wifiScanReceiver = object : BroadcastReceiver() {

  override fun onReceive(context: Context, intent: Intent) {
    val success = intent.getBooleanExtra(WifiManager.EXTRA_RESULTS_UPDATED, false)
    if (success) {
      scanSuccess()
    } else {
      scanFailure()
    }
  }
}

val intentFilter = IntentFilter()
intentFilter.addAction(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION)
context.registerReceiver(wifiScanReceiver, intentFilter)

val success = wifiManager.startScan()
if (!success) {
  // scan failure handling
  scanFailure()
}

....

private fun scanSuccess() {
  val results = wifiManager.scanResults
  ... use new scan results ...
}

private fun scanFailure() {
  // handle failure: new scan did NOT succeed
  // consider using old scan results: these are the OLD results!
  val results = wifiManager.scanResults
  ... potentially use older scan results ...
}
```

Java 的写法
```
//context AS会建议使用 applicationContext
WifiManager wifiManager = (WifiManager)
                   context.getSystemService(Context.WIFI_SERVICE);

BroadcastReceiver wifiScanReceiver = new BroadcastReceiver() {
  @Override
  public void onReceive(Context c, Intent intent) {
    boolean success = intent.getBooleanExtra(
                       WifiManager.EXTRA_RESULTS_UPDATED, false);
    if (success) {
      scanSuccess();
    } else {
      // scan failure handling
      scanFailure();
    }
  }
};

IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION);
context.registerReceiver(wifiScanReceiver, intentFilter);

boolean success = wifiManager.startScan();
if (!success) {
  // scan failure handling
  scanFailure();
}

....

private void scanSuccess() {
  List<ScanResult> results = wifiManager.getScanResults();
  ... use new scan results ...
}

private void scanFailure() {
  // handle failure: new scan did NOT succeed
  // consider using old scan results: these are the OLD results!
  List<ScanResult> results = wifiManager.getScanResults();
  ... potentially use older scan results ...
}
```

> 别急着运行，还需要一些权限的东西，先看下面的介绍

## Permissions / 权限

Android 8.0 and Android 8.1:

想获取扫描结果需要下面的权限：
A successful call to WifiManager.getScanResults() requires any one of the following permissions:
- ACCESS_FINE_LOCATION
- ACCESS_COARSE_LOCATION
- CHANGE_WIFI_STATE

Android 9 and later:

开始扫描 Wi-Fi 需要下面的权限
A successful call to WifiManager.startScan() requires all of the following conditions to be met:

Your app has the ACCESS_FINE_LOCATION or ACCESS_COARSE_LOCATION permission.
Your app has the CHANGE_WIFI_STATE permission.
Location services are enabled on the device (under Settings > Location).

> *** 注意 Android 6.0 及以上 ACCESS_FINE_LOCATION or ACCESS_COARSE_LOCATION 定位权限，需要 AndroidManifest.xml 中注册权限的同时，还需要动态申请

## Throttling / 次数限制
WifiManager.startScan()

Android 8.0 and Android 8.1:

//后台APP 1次/30分钟
Each background app can scan one time in a 30-minute period.

Android 9 and later:

//前台APP 4次/2分钟、后台APP 1次/30分钟
Each foreground app can scan four times in a 2-minute period. This allows for a burst of scans in a short time.