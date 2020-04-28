# Body fat scale SDK Instructions - Android 

[![](https://jitpack.io/v/elinkthings/BodyFatScaleSDKRepositoryAndroid.svg)](https://jitpack.io/#elinkthings/BodyFatScaleSDKRepositoryAndroid)

[aar package download link](https://github.com/elinkthings/BodyFatScaleSDKRepositoryAndroid/releases)

[key registered address](http://sdk.aicare.net.cn)

[中文文档](README_CN.md)

## Contents
- Overview
- Conditions of Use
- Import SDK
- permission settings
- Start access
- scan the device, stop scanning the device, check the scan status
- connect the device, disconnect the device
- Successful connection, accept the data returned by the scale
- Call the algorithm to calculate the data in the SDK
- Give instructions to the device
- Class description
- Version History
- FAQ
- Contact Us

## Overview
- What is elink body fat scale SDK?

> The elink body fat scale SDK is a Bluetooth development tool provided to elink partners. The SDK implements and encapsulates the elink Bluetooth protocol and is responsible for the communication between the mobile phone App and the Bluetooth body fat scale device. Bluetooth body fat scale application.

- Scope of application

> Partners who need to personalize their Android Bluetooth body fat scale APP.



## Conditions of Use
1. Minimum version android4.4 (API 19)
2. The Bluetooth version used by the device requires 4.0 and above

##  Import SDK


```
repositories {
    flatDir {
        dirs 'libs'
    }
}


Step 1. Add the JitPack repository to your build file
Add it in your root build.gradle at the end of repositories:
    allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}

Step 2. Add the dependency
	dependencies {
	        implementation 'com.github.elinkthings:BodyFatScaleSDKRepositoryAndroid:1.2.4'
	}


You can also use aar package dependency,Please download it into the project's libs yourself




```

##   permission settings

```
<!-In most cases, you need to ensure that the device supports BLE .-->
<uses-feature
    android: name = "android.hardware.bluetooth_le"
    android: required = "true" />

<uses-permission android: name = "android.permission.BLUETOOTH" />
<uses-permission android: name = "android.permission.BLUETOOTH_ADMIN" />

<!-Android 6.0 and above. Bluetooth scanning requires one of the following two permissions. You need to apply at run time .-->
<uses-permission android: name = "android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android: name = "android.permission.ACCESS_FINE_LOCATION" />

<!-Optional. If your app need dfu function .-->
<uses-permission android: name = "android.permission.INTERNET" />
```

> 6.0 and above systems need to locate permissions and need to obtain permissions manually

##   Start access
> First configure the key and secret for the SDK, [application address](http://sdk.aicare.net.cn)
```
  // Called in the application of the main project
  AiFitSDK.getInstance (). Init (this, key, secret);
```

> Add below AndroidManifest.xml application tag
```
<application>
    ...

    <service android:name="aicare.net.cn.iweightlibrary.wby.WBYService"/>

</application>

```


> You can directly make your own `Activity` class extend` BleProfileServiceReadyActivity`

```
public class MyActivity extends BleProfileServiceReadyActivity

      @Override
    protected void onCreate (Bundle savedInstanceState) {
        super.onCreate (savedInstanceState);
        // Judge whether the mobile device supports Ble
        if (! ensureBLESupported ()) {
            T.showShort (this, R.string.not_support_ble);
            finish ();
        }
        // Judge whether there is positioning permission, this method is not encapsulated. The specific code can be obtained in the demo, or you can call the permission method in your own way.
        initPermissions ();
        // Judge whether Bluetooth is on, if you need to change the style, you can do it yourself
        if (! isBLEEnabled ()) {
            showBLEDialog ();
        }
    }
 
    
```

##   scan the device, stop scanning the device, check the scan status
The APIs related to scanning are as follows. For details, please refer to the BleProfileServiceReadyActivity class. For details, refer to the sample project.

```
  // Call the startScan method to start scanning
  startScan ();
  // The getAicareDevice (final BroadData broadData) interface will call back to get a body fat scale device that complies with the Aicare protocol
    @Override
    protected void getAicareDevice (BroadData broadData) {
               // Body fat scale equipment in compliance with Aicare protocol
               
      
    }
// Call the stopScan method to stop scanning
 stopScan ();
// Call the isScanning method to see if it is scanning true: scanning; false: scanning stopped
 isScanning ();
```
> Note: If it is a broadcast scale, you do not need to perform the connection operation. You can directly get the body fat data in the scan callback getAicareDevice (BroadData broadData) method. The broadcast scale calls stopScan () to get back no data.

```
 @Override
    protected void getAicareDevice (BroadData broadData) {
        // Broadcast scale can get data directly here
        if (broadData.getDeviceType () == AicareBleConfig.BM_09) {
            if (broadData.getSpecificData ()! = null) {
                 BM09Data data = AicareBleConfig.
                       getBm09Data (broadData.getAddress (), broadData.getSpecificData ());
            } else if (broadData.getDeviceType () == AicareBleConfig.BM_15) {
                    if (broadData.getSpecificData ()! = null) {
                        BM15Data data = AicareBleConfig.getBm15Data (broadData.getAddress (),
                                broadData.getSpecificData ());
                    }
           } else {
                  if (broadData.getSpecificData ()! = null) {
                        WeightData weightData =
                                AicareBleConfig.getWeightData (broadData.getSpecificData ());
                        
                    }
           }
           
    }
```
      

## connect the device, disconnect the device

The APIs related to the connection are as follows, please refer to the BleProfileServiceReadyActivity class for details.

```
// Call the startConnect method to connect the body fat scale device
// can be obtained in the getAicareDevice (BroadData broadData) callback method.
// If there is no filtering in getAicareDevice (BroadData broadData), the broadcast should be filtered when connecting, see the demo connection for details.
startConnect (String address)
// You can get the connection status in onStateChanged
@Override
    public void onStateChanged (String deviceAddress, int state) {
        super.onStateChanged (deviceAddress, state);
        // state See the class description for specific status
    }
// Call the disconnect method in WBYService.WBYBinder class to disconnect the body fat scale
 binder.disconnect ()
```

> Use the `startConnect` method to connect to the body fat scale, use the` onStateChanged` method to monitor the status of the connection, and use the `onError` method to monitor for exceptions during the connection process, in order to facilitate additional processing and troubleshooting. Use the `isDeviceConnected` method to determine whether a connection has been established.

##   Successful connection, accept the data returned by the scale
The following methods or interfaces are automatically obtained directly after inheriting the BleProfileServiceReadyActivity class

```
// OnServiceBinded method to get an instance of WBYService.WBYBinder
    @Override
    protected void onServiceBinded (WBYService.WBYBinder binder) {
        this.binder = binder;
      
   }
 // The change and stable weight data and temperature returned by the device (supported by AC03)
    @Override
    protected void onGetWeightData (WeightData weightData) {
         // If you want to get broadcast scale data here. Need to be in getAicareDevice (final BroadData broadData) method
         // Call onGetWeightData (WeightData weightData) to pass the data through
         
         
    }
// Get the setting status in the onGetSettingStatus method. For details, see AicareBleConfig.SettingStatus
    @Override
    protected void onGetSettingStatus (int status) {

    }
// Get the version number, measurement time, user number, and impedance value in the onGetResul method
    @Override
    protected void onGetResult (int index, String result) {
          
          
    }
// Get the device to return historical data or body fat data. True is historical data.
//Body fat data will only be generated after the current user is synchronized after STATE_INDICATION_SUCCESS
    @Override
    protected void onGetFatData (boolean isHistory, BodyFatData bodyFatData) {

    }
// Get information about the number of decimal places returned by the device
    @Override
    protected void onGetDecimalInfo (DecimalInfo decimalInfo) {

    }
// Algorithm sequence information returned by the device
    @Override
    protected void onGetAlgorithmInfo (AlgorithmInfo algorithmInfo) {

    }
    

```
> Note: Some of these interfaces or methods require APP to issue commands to body fat to return data.

##   Call the algorithm to calculate the data in the SDK
- If the device returns impedance, and there is no body fat data, you can obtain the BodyFatData object by inheriting the onGetFatData method in BleProfileServiceReadyActivity, and call the AicareBleConfig.getBodyFatData method to calculate the cn.net.aicare.algorithmutil.BodyFatData object
Instructions:
```
AicareBleConfig.getBodyFatData(AlgorithmUtil.AlgorithmType.TYPE_AIC
ARE, bodyFatData.getSex(), bodyFatData.getAge(),
Double.valueOf(ParseData.getKgWeight(bodyFatData.getWeight(),
bodyFatData.getDecimalInfo())), bodyFatData .getHeight(),
bodyFatData.getAdc());
```
- If you need to get 6 additional body indicators such as body fat removal and weight control, please call AicareBleConfig.getMoreFatData to get MoreFatData object
Instructions:
```
AicareBleConfig.getMoreFatData (int sex, int height, double weight,
double bfr, double rom, double pp)

```

##   Give instructions to the device
Get an instance of WBYService.WBYBinder in BleProfileServiceReadyActivity.onServiceBinded (WBYService.WBYBinder binder), and call the method in binder

```
    @Override
    protected void onServiceBinded (WBYService.WBYBinder binder) {
        this.binder = binder;
      
   }
   // If the history is obtained
      binder.syncHistory ();
      
      
      
   // Part of the method of WBYBinder
   public class WBYBinder extends LocalBinder {

        / **
         * Get history
         * /
        @Deprecated
        public void syncHistory () {
            mManager.sendCmd (AicareBleConfig.SYNC_HISTORY, AicareBleConfig.UNIT_KG);
        }

        / **
         * Synchronize the current user
         *
         * @param user
         * /
        public void syncUser (User user) {
            if (user == null) {
                return;
            }
            mManager.syncUser (user);
        }

        / **
         * Synchronized user list
         *
         * @param userList
         * /
        @Deprecated
        public void syncUserList (List <User> userList) {
            mManager.syncUserList (userList);
        }

        / **
         * Sync current unit
         *
         * @param unit {@link AicareBleConfig # UNIT_KG}
         * {@link AicareBleConfig # UNIT_LB}
         * {@link AicareBleConfig # UNIT_ST}
         * {@link AicareBleConfig # UNIT_JIN}
         * /
        public void syncUnit (byte unit) {
            mManager.sendCmd (AicareBleConfig.SYNC_UNIT, unit);
        }

        / **
         * synchronised time
         * /
        @Deprecated
        public void syncDate () {
            mManager.syncDate ();
        }

        / **
         * Query Bluetooth version information
         * /
        @Deprecated
        public void queryBleVersion () {
            mManager.sendCmd (AicareBleConfig.GET_BLE_VERSION, AicareBleConfig.UNIT_KG);
        }

        / **
         * Update user information
         *
         * @param user
         * /
        @Deprecated
        public void updateUser (User user) {
            if (user == null) {
                return;
            }
            mManager.updateUser (user);
        }

        / **
         * Setting mode
         * /
        public void setMode (@ AicareBleConfig.MODE int cmd) {
            mManager.setMode (cmd);
        }

        / **
         * Check if authorized
         * /
        @Deprecated
        public void auth () {
            mManager.auth ();
        }

        / **
         * Set DID
         *
         * @param did
         * /
        @Deprecated
        public void setDID (int did) {
            mManager.sendDIDCmd (AicareBleConfig.SET_DID, did);
        }

        / **
         * Query DID
         * /
        @Deprecated
        public void queryDID () {
            mManager.sendDIDCmd (AicareBleConfig.QUERY_DID, 0);
        }

        / **
         * Get the number of decimal places
         * /
        public void getDecimalInfo () {
            mManager.getDecimalInfo ();
        }
        
        ...
}
    
```

##   Class description

- aicare.net.cn.iweightlibrary.entity.AlgorithmInfo (Algorithm Sequence Information)

```
Type  Parameter  //Description
double weight   // weight
int   algorithmId  //algorithm ID
int   adc         // impedance value
DecimalInfo decimalInfo //number of decimal places
```
- BM09Data (BM09 data)

```
Type Parameter  //Description
int agreementType //agreement type
int unitType   //unit type
DecimalInfo decimalInfo //Decimal places
double weight //Weight
int adc //impedance value
double temp //temperature
int algorithmId //algorithm ID
int did //(currently useless)
String bleVersion //Bluetooth version
int bleType //Bluetooth type (0x09)
String address //device address
long timeMillis //measurement timestamp
whether boolean //isStable is stable

```
- BM15Data (BM15 data)
```
Type Parameter name //Description
String version //Bluetooth version
int agreementType //agreementType
int unitType    //unitType
double weight   // weight
int adc //impedance value
double temp //temperature (if temp = 6553.5, the scale does not support temperature)
int algorithmId //algorithm ID
int did //(currently useless)
int bleType// Bluetooth type (0x15)
String address// device address
```
- BodyFatData
```
Type Parameter // Description
String date //measurement date
String time //time
double weight //weight
double bmi
double bfr
double sfr
int uvi //visceral fat index
double rom //muscle rate
double bmr //basal metabolic rate
double bm //bone mass
double vwc //moisture content
double bodyAge //
double pp //protein rate
int number
int sex
int age //(1; male; 2, female)
int height
int adc  //impedance value
```
- BroadData (broadcast data)
```
Type Parameter  Description
String name //device name
String address //device address
boolean isBright //Whether the screen is bright
int rssi //signal value
byte [] specificData //broadcast data
int deviceType //device type
```
- DecimalInfo (decimal point information)
```
Type Parameter  Description
int sourceDecimal // source data decimal places
int kgDecimal //kg number of decimal places
int lbDecimal //lb decimal places
int stDecimal //st number of decimal places
int kg //Graduation kg
int lb //Graduation lb
```
- User (User Information)
```
Type Parameter name Description
int id
int sex
int age //(1; male; 2, female)
int height
int weight
int adc //impedance (deprecated)
```
- WeightData (weight data)
```
Type Parameter name Description
int cmdType //command type (1, change; 2, stable; 3, in impedance measurement)
double weight
double temp //temperature (if the temperature is Double.MAX_VALUE, the scale does not support temperature)

DecimalInfo decimalInfo
int adc //impedance value
int algorithmType // algorithm ID
int unitType
int deviceType //device type
```
- cn.net.aicare.algorithmutil.BodyFatData(Calculated body fat data)
```
Type Parameter name Description
double bmi;	Body mass index
double bfr;	 body fat rate
double sfr;	 Subcutaneous fat rate
int uvi;	Visceral fat index
double rom;  Rate of muscle
int bmr;  basal metabolic rate
double bm;  Bone Mass
double vwc; Water content
int bodyAge;  physical bodyAge
double pp;  protein percentage
```

- MoreFatData
```
Type Parameter name Description
double standardWeight;	Standard weight
double controlWeight;	Weight control
double fat;	Fat mass
double removeFatWeight;	Fat-free weight
double muscleMass; Muscle mass
double protein; Protein amount
MoreFatData.FatLevel fatLevel; Obesity grade
public static enum FatLevel {
        UNDER,  Underweight
        THIN,   Thin
        NORMAL,  standard
        OVER,  Favor
        FAT;  overweight
}
```
- BleProfileService Connection Status
```
public static final int STATE_CONNECTING = 4; // connecting
public static final int STATE_DISCONNECTED = 0; // disconnect
public static final int STATE_CONNECTED = 1; // The connection was successful
public static final int STATE_SERVICES_DISCOVERED = 2; // Discover services
public static final int STATE_INDICATION_SUCCESS = 3; // Enable success
public static final int STATE_TIME_OUT = 5; // connection timed out
```
- AicareBleConfig.SettingStatus Status information returned by the device
```
        int NORMAL = 0; // Normal
        int LOW_POWER = 1; // Low power
        int LOW_VOLTAGE = 2; // Low voltage
        int ERROR = 3; // overload
        int TIME_OUT = 4; // timeout
        int UNSTABLE = 5; // Unstable
        int SET_UNIT_SUCCESS = 6; // Set unit success
        int SET_UNIT_FAILED = 7; // Set unit failed
        int SET_TIME_SUCCESS = 8; // Successfully set time
        int SET_TIME_FAILED = 9; // Failed to set time
        int SET_USER_SUCCESS = 10; // Set user successfully
        int SET_USER_FAILED = 11; // Failed to set user
        int UPDATE_USER_LIST_SUCCESS = 12; // Update user list successfully
        int UPDATE_USER_LIST_FAILED = 13; // Update user list failed
        int UPDATE_USER_SUCCESS = 14; // Update user successfully
        int UPDATE_USER_FAILED = 15; // Update user failed
        int NO_HISTORY = 16; // There is no historical data
        int HISTORY_START_SEND = 17; // historical data starts to be sent
        int HISTORY_SEND_OVER = 18; // historical data transmission is complete
        int NO_MATCH_USER = 19; // No matching users
        int ADC_MEASURED_ING = 20; // Impedance measurement
        int ADC_ERROR = 21; // Impedance measurement failed
        int REQUEST_DISCONNECT = 22; // The device requested to disconnect
        int SET_DID_SUCCESS = 23; // DID set successfully
        int SET_DID_FAILED = 24; // Set DID failed
        int DATA_SEND_END = 25; // Measured data transmission is complete
        int UNKNOWN = -1; // unknown
```
- WBYService Bluetooth information returned by the device
```
    public final static int BLE_VERSION = 0; // Bluetooth version
    public final static int MCU_DATE = 1; // mcu date
    public final static int MCU_TIME = 2; // mcu time
    public final static int USER_ID = 3; // user number
    public final static int ADC = 4; // impedance value
```

## Version History
| Version number | Update time | Author | Update information
| ---- | --- | ----- | -----
| 1.0   | 2017/02/15 | Suzy | Preliminary version
| 1.0.1 | 2017/07/31 | Suzy | Fix some data conversion bugs
| 1.0.2 | 2017/09/11 | Suzy | The type in the instruction is set according to the type of device actually connected
| 1.0.3 | 2017/11/16 | Suzy | Code optimization
| 1.0.4 | 2018/07/12 | Suzy | compatible broadcast scale
| 1.0.5 | 2018/01/15 | Suzy | New protocol compatible
| 1.0.6 | 2018/02/25 | Suzy | Function optimization
| 1.0.7 | 2018/02/28 | Suzy | Encapsulated decimal point class
| 1.0.8 | 2018/03/14 | Suzy | Fix the bug of wrong body age
| 1.0.9 | 2018/4/9 | Suzy | Compatible with BM09 protocol
| 1.1.1 | 2018/5/10 | Suzy | Compatible with kg / lb indexing protocol
| 1.1.2 | 2018/5/11 | Suzy | Solve the bug that the st unit data is inconsistent with the scale display
| 1.1.3 | 2018/6/22 | Suzy | Solve the bug that the decimal point is sometimes not available
| 1.1.4 | 2018/7/7 | Suzy | Compatible with BM15 protocol
| 1.1.5 | 2018/7/20 | Suzy | Compatible algorithm ID protocol
| 1.1.7 | 2019/04/30 | Stan | Add BM15 body fat data calculation algorithm
| 1.1.8 | 2019/06/6 | Stan | Modify the original data to lb, st calculation error problem
| 1.1.9 | 2019/12/19 | Xing | Add history
| 1.2.0 | 2019/1/17 | Xing | Update and optimize Bluetooth library
| 1.2.1 | 2020/3/19 | Xing | Increase the calculation method of body fat data and the algorithm of lean body mass etc.
| 1.2.2 | 2020/3/23 | Xing | Modify SDK to gradle form dependency
| 1.2.3 | 2020/4/2 | Xing | Add key verification, modify the dependent environment to androidx
| 1.2.4 | 2020/4/10 | Xing | Fix known bugs

## FAQ

- How to judge whether the currently scanned BroadData (device) is a broadcast scale or a connected scale?
1. According to the value of deviceType attribute of BroadData:
deviceType == AicareBleConfig.TYPE_WEI_BROAD is a broadcast scale without temperature
deviceType == AicareBleConfig.TYPE_WEI_TEMP_BROAD is a broadcast scale with temperature
deviceType == AicareBleConfig.TYPE_WEI is a connected scale without temperature
deviceType == AicareBleConfig.TYPE_WEI_TEMP is a connected scale with temperature

- Which units does the Bluetooth protocol support?
1. Units only support up to 4 types (kg, lb, st, kg), please refer to the factory settings of the scale for specific units supported.

-Can't scan the Bluetooth device?
1. Check whether the permissions of the App are normal. The 6.0 and above systems must locate the permissions and need to manually obtain the permissions
2. Check whether the location service of the mobile phone is turned on, some mobile phones may need to turn on the GPS
3. Unplug the battery and restart the scale
4. Whether it is connected by other mobile phones (when the scale is not connected, the Bluetooth icon on the weighing pan will continue to flash)

- Which devices are supported?
1. Support BM series connection scale, BM15 broadcast scale

- How does the connected scale determine the end of measurement?
1. OnGetFatData () method callback means the measurement is completed

- How do broadcast scales determine the end of measurement?
1. All the data of the broadcast scale is returned from getAicareDevice and parsed to get the WeightData object. GetCmdType () == 3 in WeightData means the measurement is completed, please refer to the demo for details

-The data displayed by the scale is inconsistent with the data received by the app
1. The SDK will request decimals by default, and you can use getDecimalInfo () in WBYBinder to actively obtain decimals
2. When the app calculates the weight, it needs to pass in DecimalInfo (decimal object) for calculation
```
DecimalInfo {
    private int sourceDecimal; // The source data decimal places
    private int kgDecimal; // kg decimal places
    private int lbDecimal; // lb decimal places
    private int stDecimal; // st decimal places
    private int kgGraduation; // kg division
    private int lbGraduation; // lb division
}
```
- Why can I only measure my weight and no other body fat data?
1. You must take off your shoes and socks and stand barefoot on the electrode pads of the body fat scale to measure body fat data.

- When weighing, the scale always displays Error, and the app shows that the impedance measurement has failed. What is the reason?
1. Take off your shoes and socks and barefoot stand on the electrode pad of the body fat scale to measure, and then Error will no longer be displayed.

- How to provide feedback to technical support staff efficiently?
1. The SDK will print the log on the console, please save the log as txt and send it to the technical staff when feedback the problem
2. When feeding back the problem, describe the operation before and after the problem as clearly as possible. It is best to record a video to tell how to reproduce the problem.

- Are there any criteria and copy for body fat data?
1. Body fat determination standards may vary from manufacturer to manufacturer, and there is currently no industry-recognized reference standard. The following are the standards used by our company for reference only: [《Bluetooth Body Fat Scale Judgment Standard and Mini Program Copy 20200416》](https://shimo.im/sheets/8dGqCgyhX9P6Xpcw/GX3qk/)



## Contact Us
Shenzhen Yilian Internet of Things Co., Ltd.

Phone: 0755-81773367

Official website: www.elinkthings.com

Email: app@elinkthings.com