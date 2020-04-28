
# 蓝牙体脂秤SDK使用说明 - Android

[![](https://jitpack.io/v/elinkthings/BodyFatScaleSDKRepositoryAndroid.svg)](https://jitpack.io/#elinkthings/BodyFatScaleSDKRepositoryAndroid)

[aar包下载地址](https://github.com/elinkthings/BodyFatScaleSDKRepositoryAndroid/releases)

[key申请地址](http://sdk.aicare.net.cn)

[English documentation](README.md)


## 目录

[toc]

## 概述
- 什么是elink体脂秤SDK ?

> elink体脂秤SDK 是提供给elink合作伙伴的蓝牙开发工具，该SDK对elink蓝牙协议进行了实现和封装，负责手机App与蓝牙体脂秤设备之间的通信，旨在方便合作伙伴定制自己的蓝牙体脂秤应用。

- 适用范围

> 需要个性化定制自己的 Android 蓝牙体脂秤 APP 的合作伙伴。



## 使用条件
1,最低版本 android4.4（API 19）
2,设备所使用的蓝牙版本需要4.0及以上
3,依赖环境androidx

## SDK集成


```
repositories {
    flatDir {
        dirs 'libs'
    }
}


步骤1.将JitPack存储库添加到您的构建文件中
将其添加到存储库末尾的root build.gradle中：
	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}

步骤2.添加依赖项
	dependencies {
	        implementation 'com.github.elinkthings:BodyFatScaleSDKRepositoryAndroid:1.2.4'
	}

也可以使用aar包依赖,请自行下载放到项目的libs中


```

## 权限设置

```
<!--In most cases, you need to ensure that the device supports BLE.-->
<uses-feature
    android:name="android.hardware.bluetooth_le"
    android:required="true"/>

<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

<!--Android 6.0 and above. Bluetooth scanning requires one of the following two permissions. You need to apply at run time.-->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

<!--Optional. If your app need dfu function.-->
<uses-permission android:name="android.permission.INTERNET"/>
```

>  6.0及以上系统必须要定位权限，且需要手动获取权限

## 开始接入

> 首先给SDK配置key和secret，[申请地址](http://sdk.aicare.net.cn)
```
 //在主项目的application中调用
 AiFitSDK.getInstance().init(this, key, secret);
```

> 在AndroidManifest.xml application标签下面增加
```
<application>
    ...

    <service android:name="aicare.net.cn.iweightlibrary.wby.WBYService"/>

</application>

```


> 你可以直接让你自己的`Activity`类继承`BleProfileServiceReadyActivity`

```
public class MyActivity extends BleProfileServiceReadyActivity

      @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //判断手机设备是否支持Ble
        if (!ensureBLESupported()) {
            T.showShort(this, R.string.not_support_ble);
            finish();
        }
        //判断是否有定位权限,此方法没有进行封装具体代码可在demo中获得，也可按照自己方式去调用请求权限方法
        initPermissions();
        //判断蓝牙是否打开，若需要换样式，可自己去实现
        if (!isBLEEnabled()) {
            showBLEDialog();
        }
    }


```

## 扫描设备，停止扫描设备,查看扫描状态
与扫描相关的API如下，详情参考BleProfileServiceReadyActivity类，具体使用参考sample工程

```
  //调用startScan方法开启扫描
  startScan();
  //getAicareDevice(final BroadData broadData)接口会回调获取到符合Aicare协议的体脂秤设备
    @Override
    protected void getAicareDevice(BroadData broadData) {
               //符合Aicare协议的体脂秤设备


    }
//调用stopScan方法停止扫描
 stopScan();
//调用isScanning方法查看是否在扫描 true:正在扫描; false:已停止扫描
 isScanning();

```
> 注意： 如果为广播秤，不需要去执行连接操作，在扫描的回调getAicareDevice(BroadData broadData)方法中可以直接去获取到体脂数据,广播秤调用 stopScan()回获取不到数据

```
 @Override
    protected void getAicareDevice(BroadData broadData) {
        //广播秤可以直接在此获取数据
        if (broadData.getDeviceType() == AicareBleConfig.BM_09) {
            if (broadData.getSpecificData() != null) {
                 BM09Data data = AicareBleConfig.
                       getBm09Data(broadData.getAddress(),broadData.getSpecificData());
            }else if (broadData.getDeviceType() == AicareBleConfig.BM_15) {
                    if (broadData.getSpecificData() != null) {
                        BM15Data data = AicareBleConfig.getBm15Data(broadData.getAddress(),
                                broadData.getSpecificData());
                    }
           }else{
                  if (broadData.getSpecificData() != null) {
                        WeightData weightData =
                                AicareBleConfig.getWeightData(broadData.getSpecificData());

                    }
           }

    }
```


## 连接设备，断开设备

与连接相关的API如下，详情参考BleProfileServiceReadyActivity类，具体使用参考sample工程。

```
//调用startConnect方法去连接体脂秤设备 需要传入体脂秤设备的mac地址 mac地址
//可以在getAicareDevice（BroadData broadData）回调方法中获得 。
//如果在getAicareDevice(BroadData broadData) 没有做过滤，连接时要去过滤掉广播称，详情见demo连接处
startConnect(String address)
//在onStateChanged可以获取到连接状态
@Override
    public void onStateChanged(String deviceAddress, int state) {
        super.onStateChanged(deviceAddress, state);
        //state 具体状态看类说明
    }
//调用WBYService.WBYBinder类中disconnect方法去断开体脂秤
 binder.disconnect()
```

> 使用`startConnect`方法连接体脂秤，使用`onStateChanged`方法监听连接的状态，使用`onError`方法监听连接过程中的异常，以便于进行额外的处理和问题排查。使用`isDeviceConnected`方法判断连接是否已经建立。

## 连接成功，接受秤返回的数据
以下方法或接口可直接在继承BleProfileServiceReadyActivity类后自动获得

```
//onServiceBinded方法中获得WBYService.WBYBinder的实例
    @Override
    protected void onServiceBinded(WBYService.WBYBinder binder) {
        this.binder = binder;

   }
 //设备返回的变化和稳定的体重数据和温度(AC03才支持)
    @Override
    protected void onGetWeightData(WeightData weightData) {
         //如果想要在这里获取到广播秤数据。需要在getAicareDevice(final BroadData broadData)方法中
         //调用onGetWeightData(WeightData weightData)去把数据透传过来


    }
//onGetSettingStatus方法中获得设置状态 详情查看AicareBleConfig.SettingStatus
    @Override
    protected void onGetSettingStatus(int status) {

    }
//onGetResul方法中获得获得版本号，测量时间，用户编号，阻抗值
    @Override
    protected void onGetResult(int index, String result) {


    }
// 获得设备返回据历史数据或体脂数据  true为历史数据
    @Override
    protected void onGetFatData(boolean isHistory, BodyFatData bodyFatData) {

    }
//获取设备返回的小数点位数信息
    @Override
    protected void onGetDecimalInfo(DecimalInfo decimalInfo) {

    }
//设备返回的算法序列信息
    @Override
    protected void onGetAlgorithmInfo(AlgorithmInfo algorithmInfo) {

    }


```
> 注意：这些接口或方法部分需要APP给体脂下发命令才会有返回数据.

## 调用SDK中的算法库
- 如果设备返回阻抗,没有体脂数据可以通过继承BleProfileServiceReadyActivity中的onGetFatData方法获得 BodyFatData 对象,调用AicareBleConfig.getBodyFatData方法计算得到cn.net.aicare.algorithmutil.BodyFatData对象
使用方法:
```

AicareBleConfig.getBodyFatData(AlgorithmUtil.AlgorithmType.TYPE_AIC
ARE, bodyFatData.getSex(), bodyFatData.getAge(),
Double.valueOf(ParseData.getKgWeight(bodyFatData.getWeight(),
bodyFatData.getDecimalInfo())), bodyFatData .getHeight(),
bodyFatData.getAdc());
```
- 如需要获取去脂体重，体重控制量等额外的 6 项身体指标，请调用调用AicareBleConfig.getMoreFatData计算得到 MoreFatData 对象
使用方法:
```
AicareBleConfig.getMoreFatData(int sex, int height, double weight,
double bfr, double rom, double pp)

```

## APP给设备下发指令
在BleProfileServiceReadyActivity.onServiceBinded(WBYService.WBYBinder binder)获得WBYService.WBYBinder的实例，调用binder里面方法

```
    @Override
    protected void onServiceBinded(WBYService.WBYBinder binder) {
        this.binder = binder;

   }
   //如获取到历史记录
      binder.syncHistory();



   //WBYBinder的部分方法
   public class WBYBinder extends LocalBinder {

        /**
         * 获取历史记录
         */
        @Deprecated
        public void syncHistory() {
            mManager.sendCmd(AicareBleConfig.SYNC_HISTORY, AicareBleConfig.UNIT_KG);
        }

        /**
         * 同步当前用户
         *
         * @param user
         */
        public void syncUser(User user) {
            if (user == null) {
                return;
            }
            mManager.syncUser(user);
        }

        /**
         * 同步用户列表
         *
         * @param userList
         */
        @Deprecated
        public void syncUserList(List<User> userList) {
            mManager.syncUserList(userList);
        }

        /**
         * 同步当前单位
         *
         * @param unit {@link AicareBleConfig#UNIT_KG}
         *             {@link AicareBleConfig#UNIT_LB}
         *             {@link AicareBleConfig#UNIT_ST}
         *             {@link AicareBleConfig#UNIT_JIN}
         */
        public void syncUnit(byte unit) {
            mManager.sendCmd(AicareBleConfig.SYNC_UNIT, unit);
        }

        /**
         * 同步时间
         */
        @Deprecated
        public void syncDate() {
            mManager.syncDate();
        }

        /**
         * 查询蓝牙版本信息
         */
        @Deprecated
        public void queryBleVersion() {
            mManager.sendCmd(AicareBleConfig.GET_BLE_VERSION, AicareBleConfig.UNIT_KG);
        }

        /**
         * 更新用户信息
         *
         * @param user
         */
        @Deprecated
        public void updateUser(User user) {
            if (user == null) {
                return;
            }
            mManager.updateUser(user);
        }

        /**
         * 设置模式
         */
        public void setMode(@AicareBleConfig.MODE int cmd) {
            mManager.setMode(cmd);
        }

        /**
         * 校验是否授权
         */
        @Deprecated
        public void auth() {
            mManager.auth();
        }

        /**
         * 设置DID
         *
         * @param did
         */
        @Deprecated
        public void setDID(int did) {
            mManager.sendDIDCmd(AicareBleConfig.SET_DID, did);
        }

        /**
         * 查询DID
         */
        @Deprecated
        public void queryDID() {
            mManager.sendDIDCmd(AicareBleConfig.QUERY_DID, 0);
        }

        /**
         * 获取小数位数
         */
        public void getDecimalInfo() {
            mManager.getDecimalInfo();
        }

}

```


## 类说明

 - aicare.net.cn.iweightlibrary.entity.AlgorithmInfo(算法序列信息)
```
类型	参数名	说明
double	weight	体重
int	algorithmId	算法ID
int	adc	阻抗值
DecimalInfo	decimalInfo	小数点位数
```

- BM09Data(BM09数据)
```
类型	参数名	说明
int	agreementType	协议类型
int	unitType	单位类型
DecimalInfo	decimalInfp	小数点位数
double	weight	体重
int	adc	阻抗值
double	temp	温度
int	algorithmId	算法ID
int	did	（目前无用）
String	bleVersion	蓝牙版本
int	bleType	蓝牙类型（0x09）
String	address	设备地址
long	timeMillis	测量时间戳
boolean	isStable	是否稳定
```
- BM15Data(BM15数据)
```
类型	参数名	说明
String	version	蓝牙版本
int	agreementType	协议类型
int	unitType	单位类型
double	weight	体重
int	adc	阻抗值
double	temp	温度（若temp=6553.5，则表示秤不支持温度）
int	algorithmId	算法ID
int	did	（目前无用）
int	bleType	蓝牙类型（0x15）
String	address	设备地址
```

- BodyFatData(体脂数据)
```
类型	参数名	说明
String	date	测量日期
String	time	测量时间
double	weight	体重
double	bmi	身体质量指数
double	bfr	体脂率
double	sfr	皮下脂肪率
int	uvi	内脏脂肪指数
double	rom	肌肉率
double	bmr	基础代谢率
double	bm	骨量
double	vwc	水分率
double	bodyAge	身体年龄
double	pp	蛋白率
int	number	编号
int	sex	性别
int	age	年龄（1、男；2、女）
int	height	身高
int	adc	阻抗值
```

- BroadData(广播数据)
```
类型	参数名	说明
String	name	设备名
String	address	设备地址
boolean	isBright	是否亮屏
int	rssi	信号值
byte[]	specificData	广播数据
int	deviceType	设备类型
```
- DecimalInfo(小数点位数信息)
```
类型	参数名	说明
int	sourceDecimal	源数据小数点位数
int	kgDecimal	kg小数点位数
int	lbDecimal	lb小数点位数
int	stDecimal	st小数点位数
int	kgGraduation	kg分度
int	lbGraduation	lb分度
```
- User(用户信息)
```
类型	参数名	说明
int	id	编号
int	sex	性别
int	age	年龄（1、男；2、女）
int	height	身高
int	weight	体重
int	adc	阻抗值（弃用）
```
- WeightData(体重数据)
```
类型	参数名	说明
int	cmdType	命令类型（1、变化；2、稳定；3、阻抗测量中）
double	weight	体重
double	temp	温度（若温度为Double.MAX_VALUE则表示秤不支持温度）
DecimalInfo	decimalInfo	小数点位数信息
int	adc	阻抗值
int	algorithmType	算法ID
int	unitType	单位类型
int	deviceType	设备类型
```
- cn.net.aicare.algorithmutil.BodyFatData(计算得到的体脂数据)
```
类型	参数名	说明
double bmi;	身体质量指数
double bfr;	体脂率 body fat rate
double sfr;	皮下脂肪率 Subcutaneous fat rate
int uvi;	内脏脂肪指数
double rom; 肌肉率 Rate of muscle
int bmr; 基础代谢率 basal metabolic rate
double bm; 骨骼质量 Bone Mass
double vwc; 水含量
int bodyAge; 身体年龄 physical bodyAge
double pp; 蛋白率 protein percentage
```

- MoreFatData
```
类型	参数名	说明
double standardWeight;	标准体重
double controlWeight;	体重控制量
double fat;	脂肪量
double removeFatWeight;	去脂体重
double muscleMass; 肌肉量
double protein; 蛋白量
MoreFatData.FatLevel fatLevel; 肥胖等级
public static enum FatLevel {
        UNDER,  体重不足
        THIN,   偏瘦
        NORMAL,  标准
        OVER,  偏重
        FAT;  超重
}
```
- BleProfileService 连接状态
```
public static final int STATE_CONNECTING = 4; //连接中
public static final int STATE_DISCONNECTED = 0; //断开连接
public static final int STATE_CONNECTED = 1;//连接成功
public static final int STATE_SERVICES_DISCOVERED = 2;//发现服务
public static final int STATE_INDICATION_SUCCESS = 3;//使能成功
public static final int STATE_TIME_OUT = 5;//连接超时
```
- AicareBleConfig.SettingStatus 设备返回的状态信息
```
        int NORMAL = 0;//正常
        int LOW_POWER = 1;//低功耗
        int LOW_VOLTAGE = 2;//低电压
        int ERROR = 3;//超载
        int TIME_OUT = 4;//超时
        int UNSTABLE = 5;//称不稳定
        int SET_UNIT_SUCCESS = 6;//设置单位成功
        int SET_UNIT_FAILED = 7;//设置单位失败
        int SET_TIME_SUCCESS = 8;//设置时间成功
        int SET_TIME_FAILED = 9;//设置时间失败
        int SET_USER_SUCCESS = 10;//设置用户成功
        int SET_USER_FAILED = 11;//设置用户失败
        int UPDATE_USER_LIST_SUCCESS = 12;//更新用户列表成功
        int UPDATE_USER_LIST_FAILED = 13;//更新用户列表失败
        int UPDATE_USER_SUCCESS = 14;//更新用户成功
        int UPDATE_USER_FAILED = 15;//更新用户失败
        int NO_HISTORY = 16;//没有历史数据
        int HISTORY_START_SEND = 17;//历史数据开始发送
        int HISTORY_SEND_OVER = 18;//历史数据发送完成
        int NO_MATCH_USER = 19;//没有匹配的用户
        int ADC_MEASURED_ING = 20;//阻抗测量中
        int ADC_ERROR = 21;//阻抗测量失败
        int REQUEST_DISCONNECT = 22;//设备请求断开
        int SET_DID_SUCCESS = 23;//设置DID成功
        int SET_DID_FAILED = 24;//设置DID失败
        int DATA_SEND_END = 25;//测量数据发送完成
        int UNKNOWN = -1;//未知
```
- WBYService 设备返回的蓝牙信息
```
    public final static int BLE_VERSION = 0; //蓝牙版本
    public final static int MCU_DATE = 1;  //mcu日期
    public final static int MCU_TIME = 2;  //mcu 时间
    public final static int USER_ID = 3; //用户编号
    public final static int ADC = 4; //阻抗值
```
## 版本历史
|版本号|更新时间|作者|更新信息|
|:----|:---|:-----|-----|
|1.0|	2017/2/15|	Suzy|	初步版本|
|1.0.1|	2017/7/31|	Suzy|	修复部分数据转换bug
|1.0.2|	2017/9/11|	Suzy|	指令中的类型根据实际连上的设备类型设置
|1.0.3|	2017/11/16|	Suzy|	代码优化
|1.0.4|	2018/7/12|	Suzy|	兼容广播秤
|1.0.5|	2018/1/15|	Suzy|	兼容新协议
|1.0.6|	2018/2/25|	Suzy|	功能优化
|1.0.7|	2018/2/28|	Suzy|	封装小数点位数类
|1.0.8|	2018/3/14|	Suzy|	修复身体年龄错误的bug
|1.0.9|	2018/4/9|	Suzy|	兼容BM09协议
|1.1.1|	2018/5/10|	Suzy|	兼容kg/lb分度协议
|1.1.2|	2018/5/11|	Suzy|	解决st单位数据跟秤端显示不一致的bug
|1.1.3|	2018/6/22|	Suzy|	解决小数点位数有时获取不到的bug
|1.1.4|	2018/7/7|	Suzy|	兼容BM15协议
|1.1.5|	2018/7/20|	Suzy|	兼容算法ID协议
|1.1.7|	2019/04/30|	Stan|	添加BM15体脂数据计算算法
|1.1.8|	2019/06/6|	Stan|	修改原始数据转lb ,st计算错误问题
|1.1.9|	2019/12/19|	Xing|	增加历史记录
|1.2.0|	2019/1/17|	Xing|	更新优化蓝牙库
|1.2.1|	2020/3/19|	Xing|	增加体脂数据计算方法和去脂体重算法等
|1.2.2|	2020/3/23|	Xing|	修改SDK为gradle形式依赖
|1.2.3|	2020/4/2|	Xing|	增加key校验,修改依赖环境为androidx
|1.2.4|	2020/4/10|	Xing|	修复已知bug

## FAQ

- 如何判断区分当前扫描到的BroadData(设备)是广播秤还是连接秤？
1.根据BroadData的deviceType属性值区分：
deviceType==AicareBleConfig.TYPE_WEI_BROAD是不带温度的广播秤
deviceType==AicareBleConfig.TYPE_WEI_TEMP_BROAD是有温度的广播秤
deviceType==AicareBleConfig.TYPE_WEI是不带温度的连接秤
deviceType==AicareBleConfig.TYPE_WEI_TEMP是有温度的连接秤

- 蓝牙协议支持哪些单位？
1.单位最多只支持4种（kg，lb，st，斤），具体支持什么单位请参照秤的出厂设置。

- 扫描不到蓝牙设备？
1.查看App权限是否正常,6.0及以上系统必须要定位权限，且需要手动获取权限
2.查看手机的定位服务是否开启,部分手机可能需要打开GPS
3.拔掉电池重启秤
4.是否被其他手机连接(秤未被连接时，秤盘上蓝牙图标会不断闪烁)

- 支持哪些设备？
1.支持BM系列的连接秤、BM15广播秤

- 连接秤如何判定测量结束？
1.onGetFatData()方法回调就代表测量完成

- 广播秤如何判定测量结束？
1.广播秤所有的数据都是从getAicareDevice返回并解析得到WeightData对象,WeightData中的getCmdType()==3表示测量完成,详细请参考demo

- 秤显示的数据和app收到的数据不一致
1.SDK会默认请求获取小数的,可使用WBYBinder中的getDecimalInfo()主动获取小数位
2.app计算重量的时候需要传入DecimalInfo(小数对象)进行计算
```
DecimalInfo{
    private int sourceDecimal;//源数据小数位数
    private int kgDecimal;//kg小数位数
    private int lbDecimal;//lb小数位数
    private int stDecimal;//st小数位数
    private int kgGraduation;//kg分度
    private int lbGraduation;//lb分度
}
```
- 为什么只能测到体重，没有其他体脂数据？
1.必须脱掉鞋和袜子，光脚站在体脂秤的电极片上，才能测出体脂数据。

- 称量时秤总是显示Error，app显示阻抗测量失败，是什么原因？
1.脱掉鞋和袜子，光脚站在体脂秤的电极片上测量，就不会再显示Error。

- 如何高效的向技术支持人员提供反馈？
1.SDK会在控制台打印log，反馈问题时请先将log保存为txt，发给技术人员
2.反馈问题时，尽可能将问题出现时的前后操作描述清楚，最好能录制视频告知问题如何复现。

- 是否有各项体脂数据的判定标准和文案呢？
1.体脂判定标准各厂商标准都可能不一样，目前并没有行业公认的参考标准。如下是我司使用的标准，仅供参考： [《蓝牙体脂秤判定标准及小程序文案20200416》](https://shimo.im/sheets/8dGqCgyhX9P6Xpcw/GX3qk/)

## 联系我们
深圳市易连物联网有限公司

电话：0755-81773367

官网：www.elinkthings.com

邮箱：app@elinkthings.com
