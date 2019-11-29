## 1、快捷图标

```java
package com.android.launcher3.snail.shortcut.ui;

public class ShortcutContainer extends LinearLayout {
    

    private void registerBroadcast() {
        IntentFilter filter = new IntentFilter();
        // WLAN
        filter.addAction(BaseController.WIFI_CHANGED_ACTION);
//        filter.addAction("android.intent.action.ANY_DATA_STATE");
        // 蓝牙
        filter.addAction(BaseController.BLUE_CHANGED_ACTION);
        //飞行模式
        filter.addAction(BaseController.AIRPLANE_CHANGED_ACTION);

        filter.addAction(SIM_ACTION);

        mStateReceiver = new ShortcutStateReceiver();
        getContext().registerReceiver(mStateReceiver, filter);
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        getContext().unregisterReceiver(mStateReceiver);
        unRegisterObserver();
    }

    class ShortcutStateReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            BaseController controller = null;
            switch (action) {
                case BaseController.WIFI_CHANGED_ACTION:
                    controller = mWifiController;
                    break;
                case BaseController.BLUE_CHANGED_ACTION:
                    controller = mBluetoothController;
                    break;
                case BaseController.AIRPLANE_CHANGED_ACTION:
                    controller = mAirplaneController;
                    break;
                case SIM_ACTION:
                    checkSim(intent);
                    break;
                default:
                    break;
            }
            if (controller != null) {
                controller.updateState(intent);
            }
        }
    }

    private void checkSim(Intent intent){
        String simState = intent.getStringExtra("ss");
        if(!TextUtils.isEmpty(simState)){
            int subId = intent.getIntExtra("subscription", SubscriptionManager.INVALID_SUBSCRIPTION_ID);
            if(subId == -1){
                return;
            }
            if (simState.equals("READY") ||  //卡正常状态  即可以读出卡信息
                    simState.equals("IMSI") ||
                    simState.equals("LOADED")) {
                registerSimOberver(subId);
            } else if (simState.equals("ABSENT")) {
                unRegisterSimObserver(subId);
            }
        }
    }

    private void registerSimOberver(int subId) {
        if (SimOberverMap.contains(subId)) {
            return;
        }
        ContentResolver resolver = getContext().getContentResolver();
        final String key = MobileDataController.SETTINGS_KEY_MOBILE_DATA + subId;
        SettingsValueChangeContentObserver mobileDataObserver = new SettingsValueChangeContentObserver(getContext(), key, mobileHandlerListener);
        resolver.registerContentObserver(getSettingsUri(key), true, mobileDataObserver);
        SimOberverMap.put(subId, mobileDataObserver);
        Log.i("lmq", "registerSimOberver subId = " + subId+" SimOberverMap="+SimOberverMap.size());
    }

    private void unRegisterSimObserver(int subId) {
        SettingsValueChangeContentObserver mobileDataObserver = SimOberverMap.get(subId);
        if (mobileDataObserver != null) {
            getContext().getContentResolver().unregisterContentObserver(mobileDataObserver);
        }
    }

    private void registerObserver() {
        ContentResolver resolver = getContext().getContentResolver();

        // 勿扰监听
        mZenModeObserver = new SettingsValueChangeContentObserver(getContext(), ZenModeController.SETTINGS_KEY_ZEN_CODE, zenModeHandlerListener);
        resolver.registerContentObserver(getSettingsUri(ZenModeController.SETTINGS_KEY_ZEN_CODE), true, mZenModeObserver);

        // 屏幕旋转监听
        mRotationObserver = new SettingsValueChangeContentObserver(getContext(), USER_ROTATION, rotationHandlerListener);
        resolver.registerContentObserver(Settings.System.getUriFor(USER_ROTATION), true, mRotationObserver);
        mAutoRotationObserver = new SettingsValueChangeContentObserver(getContext(), ACCELEROMETER_ROTATION,rotationHandlerListener);
        resolver.registerContentObserver(Settings.System.getUriFor(ACCELEROMETER_ROTATION), true, mAutoRotationObserver);

        // 自动亮度监听
        mBrightObserver = new SettingsValueChangeContentObserver(getContext(), SCREEN_BRIGHTNESS_MODE, brightHandlerListener);
        resolver.registerContentObserver(Settings.System.getUriFor(SCREEN_BRIGHTNESS_MODE), true, mBrightObserver);
    }

    private void unRegisterObserver() {
        getContext().getContentResolver().unregisterContentObserver(mZenModeObserver);
        getContext().getContentResolver().unregisterContentObserver(mRotationObserver);
        getContext().getContentResolver().unregisterContentObserver(mBrightObserver);
        getContext().getContentResolver().unregisterContentObserver(mAutoRotationObserver);
    }

    private Uri getSettingsUri(String key) {
        return Settings.Global.getUriFor(key);
    }

    private void updateUI(TextView view, int drawableId, boolean hasFilter) {
        Drawable icon = getResources().getDrawable(drawableId);
        int width = getResources().getDimensionPixelSize(R.dimen.short_icon_drawable_width);
        icon.setBounds(0, 0, width, width);
        if (hasFilter) {
            icon.setColorFilter(mMainColor, PorterDuff.Mode.MULTIPLY);
        }
        view.setCompoundDrawablesRelative(null, icon, null, null);
    }

    private HandlerListener getWifiHandlerListener() {
        return new HandlerListener() {
            @Override
            public boolean onLongClick(View v) {
                Intent intent = new Intent(Settings.ACTION_WIFI_SETTINGS);
                startSettings(intent);
                return false;
            }

            @Override
            public void onClick(View v) {
                mWifiController.switchState();
            }

            @Override
            public void shortcutViewTurnOn(EnabledState enabled) {
                updateUI(mWlanTv, R.drawable.ic_wifi_on, false);
            }

            @Override
            public void shortcutViewTurnOff(EnabledState enabled) {
                updateUI(mWlanTv, R.drawable.ic_wifi_off, false);
            }
        };
    }

    private HandlerListener getBlueToothHandlerListener() {
        return new HandlerListener() {
            @Override
            public boolean onLongClick(View v) {
                Intent intent = new Intent(Settings.ACTION_BLUETOOTH_SETTINGS);
                startSettings(intent);
                return false;
            }

            @Override
            public void onClick(View v) {
                mBluetoothController.switchState();
            }

            @Override
            public void shortcutViewTurnOn(EnabledState enabled) {
                updateUI(mBluetoothTv, R.drawable.ic_bluetooth_on, false);
            }

            @Override
            public void shortcutViewTurnOff(EnabledState enabled) {
                updateUI(mBluetoothTv, R.drawable.ic_bluetooth_off, false);
            }
        };
    }
    private static final ComponentName CELLULAR_SETTING_COMPONENT = new ComponentName(
            "com.android.settings", "com.android.settings.Settings$DataUsageSummaryActivity");
    private static final ComponentName DATA_PLAN_CELLULAR_COMPONENT = new ComponentName(
            "com.android.settings", "com.android.settings.Settings$DataPlanUsageSummaryActivity");
    private HandlerListener getMobileHandlerListener() {
        return new HandlerListener() {

            @Override
            public boolean onLongClick(View v) {
                Intent intent = new Intent().setComponent(CELLULAR_SETTING_COMPONENT);
                startSettings(intent);
                return false;
            }

            @Override
            public void onClick(View v) {
                mDataController.switchState();
            }

            @Override
            public void shortcutViewTurnOn(EnabledState enabled) {
                updateUI(mMobileDataTv, R.drawable.ic_mobile_on, false);
            }

            @Override
            public void shortcutViewTurnOff(EnabledState enabled) {
                updateUI(mMobileDataTv, R.drawable.ic_mobile_off, false);
            }
        };
    }

   ......
}

```



### 1.1飞行模式：

```java
public class AirplaneController extends BaseController {

    ConnectivityManager mConnectivityManager;

    public AirplaneController(Context context, IView iView) {
        super(context, iView);
    }

    @Override
    public void switchState() {
        super.switchState();
        try {
            boolean enable = isAirPlaneOn();
            Log.i("lmq", "switchState airplane enable = " + enable);

            // 需要重启才会获取权限
            Class ownerClass = mConnectivityManager.getClass();
            Class[] argsClass = new Class[1];
            argsClass[0] = boolean.class;
            Method method = ownerClass.getMethod("setAirplaneMode", argsClass);
            method.invoke(mConnectivityManager, !enable);

//            Settings.System.putInt(mContext.getContentResolver(),
//                    Settings.Global.AIRPLANE_MODE_ON, !enable ? 1 : 0);
//            Intent intent = new Intent(Intent.ACTION_AIRPLANE_MODE_CHANGED);
//            intent.putExtra("state", !enable);
//            mContext.sendBroadcast(intent);
        } catch (Exception e) {
            Log.e("lmq", "switchState fail e " + e);
        }
    }

    private boolean isAirPlaneOn() {
        return Settings.Global.getInt(mContext.getContentResolver(), Settings.Global.AIRPLANE_MODE_ON, 0) != 0;
    }
```

### 1.2蓝牙：

```java
public class BluetoothController extends BaseController {
    public BluetoothAdapter mBluetoothAdapter;

    public BluetoothController(Context context, IView iView) {
        super(context, iView);
    }

    @Override
    public void switchState() {
        boolean curState = mBluetoothAdapter.isEnabled();
        boolean ret = curState ? mBluetoothAdapter.disable() : mBluetoothAdapter.enable();
        Log.i("lmq", "switchState  curState=" + curState + " ret=" + ret);
    }
}
```

### 1.3 屏幕亮度

```java
public class BrightnessController extends BaseController {

    @Override
    public void switchState() {
        super.switchState();
        int brightnessMode = getCurMode();
        if (brightnessMode == SCREEN_BRIGHTNESS_MODE_MANUAL) {
            brightnessMode = SCREEN_BRIGHTNESS_MODE_AUTOMATIC;
        } else {
            brightnessMode = SCREEN_BRIGHTNESS_MODE_MANUAL;
        }
        try {
            Settings.System.putInt(mContext.getContentResolver(), Settings.System.SCREEN_BRIGHTNESS_MODE, brightnessMode);
        } catch (Exception e) {

        }
    }
}
```

### 1.4 手电筒

```java
public class FlashLightController extends BaseController {

    private CameraManager mCameraManager;
    private String mCameraId;

    /**
     * Lock on {@code this} when accessing
     */
    private boolean mFlashlightEnabled;
    private Handler mHandler;
    private static final long CAMERA_INIT_DELAY_TIME = 2000;


    public FlashLightController(Context context, IView iView) {
        super(context, iView);
    }

    @Override
    public void init() {
        mCameraManager = (CameraManager) mContext.getSystemService(Context.CAMERA_SERVICE);
        mHandler = new Handler();
        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                tryInitCamera();
            }
        }, CAMERA_INIT_DELAY_TIME);  // 必须延迟注册回调否则，空指针异常

        updateView(mFlashlightEnabled);
    }

    private void updateView(boolean flashlightEnabled) {
        IView.EnabledState state = IView.EnabledState.generateState(flashlightEnabled);
        if (flashlightEnabled) {
            mIView.shortcutViewTurnOn(state);
        } else {
            mIView.shortcutViewTurnOff(state);
        }
    }

    @TargetApi(Build.VERSION_CODES.M)
    private void tryInitCamera() {
        try {
            mCameraId = getCameraId();
            if (mCameraId != null) {
                mCameraManager.registerTorchCallback(mTorchCallback, null);
            }
        } catch (Throwable e) {
            Log.e(TAG, "Couldn't initialize.", e);
            return;
        }
    }

    @TargetApi(Build.VERSION_CODES.M)
    public void setFlashlight(boolean enabled) {
        synchronized (this) {
            if (mCameraId == null) return;
            if (mFlashlightEnabled != enabled) {
                mFlashlightEnabled = enabled;
                try {
                    mCameraManager.setTorchMode(mCameraId, enabled);
                } catch (CameraAccessException e) {
                    Log.e(TAG, "Couldn't set torch mode", e);
                    mFlashlightEnabled = false;
                }
            }
        }
    }

    private String getCameraId() throws CameraAccessException {
        String[] ids = mCameraManager.getCameraIdList();
        for (String id : ids) {
            CameraCharacteristics c = mCameraManager.getCameraCharacteristics(id);
            Boolean flashAvailable = c.get(CameraCharacteristics.FLASH_INFO_AVAILABLE);
            Integer lensFacing = c.get(CameraCharacteristics.LENS_FACING);
            Log.i("lmq", "id=" + id + " flashAvailable=" + flashAvailable + " lensFacing=" + lensFacing);
            if (flashAvailable != null && flashAvailable) {
                return id;
            }
        }
        return null;
    }

    private final CameraManager.TorchCallback mTorchCallback =
            new CameraManager.TorchCallback() {

                @Override
                public void onTorchModeUnavailable(String cameraId) {
                    updateView(false);
                }

                @Override
                public void onTorchModeChanged(String cameraId, boolean enabled) {
                    if (TextUtils.equals(cameraId, mCameraId)) {
                        setTorchMode(enabled);
                        updateView(enabled);
                    }
                }

                private void setTorchMode(boolean enabled) {
                    synchronized (FlashLightController.this) {
                        mFlashlightEnabled = enabled;
                    }
                }
            };

    @TargetApi(Build.VERSION_CODES.M)
    @Override
    public void switchState() {
        boolean enabled = !mFlashlightEnabled;
        setFlashlight(enabled);
    }

}

```

### 1.5 移动数据

```java
public class MobileDataController extends BaseController {

    public final static String SETTINGS_KEY_MOBILE_DATA = "mobile_data";
    public final static String SIM_ACTION= "android.intent.action.SIM_STATE_CHANGED";

    public void setSimChangeListener(ShortcutContainer.SimChangeListener simChangeListener) {
        this.simChangeListener = simChangeListener;
    }

    private ShortcutContainer.SimChangeListener simChangeListener;

    public MobileDataController(Context context, IView iView) {
        super(context, iView);

    }

    @Override
    public void switchState() {
        if (simChangeListener != null) {
            simChangeListener.change(getDefaultDataSubId(mContext));
        }
        boolean curState = getDataEnabled(mContext);
        Log.i(TAG, "mobile data curState=" + curState+" subid = "+getDefaultDataSubId(mContext));
        setDataEnabled(!curState, mContext);
    }

    public static void setDataEnabled(boolean enable, Context context) {
        try {
            TelephonyManager telephonyService = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
            Method setDataEnabled = telephonyService.getClass().getDeclaredMethod("setDataEnabled", boolean.class);
            if (null != setDataEnabled) {
                setDataEnabled.invoke(telephonyService, enable);
                Log.i(TAG, "setDataEnabled success");
            }
        } catch (Exception e) {
            Log.i(TAG, "setDataEnabled exception e=" + e);
        }
    }

    public static boolean getDataEnabled(Context context) {
        boolean enabled = false;
        try {
            TelephonyManager telephonyService = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
            Method getDataEnabled = telephonyService.getClass().getDeclaredMethod("isDataEnabled");
            if (null != getDataEnabled) {
                enabled = (Boolean) getDataEnabled.invoke(telephonyService);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return enabled;
    }

    @SuppressLint("NewApi")
    public static int getDefaultDataSubId(Context context) {
        int id = -1;
        SubscriptionManager sm = SubscriptionManager.from(context);
        try {
            Method getSubId = sm.getClass().getDeclaredMethod("getDefaultDataSubscriptionId");
            if (getSubId != null) {
                id = (int) getSubId.invoke(sm);
            }
            return id;
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return id;
    }

    public static boolean hasSimCard(Context context) {
        TelephonyManager telMgr = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
        int simState = telMgr.getSimState();
        boolean result = true;
        switch (simState) {
            case TelephonyManager.SIM_STATE_ABSENT:
                result = false; // 没有SIM卡
                break;
            case TelephonyManager.SIM_STATE_UNKNOWN:
                result = false;
                break;
        }
        return result;
    }

}

```

### 1.6 自动旋转

```java
public class RotationController extends BaseController {
    public RotationController(Context context, IView iView) {
        super(context, iView);
    }

    @Override
    public void switchState() {
        super.switchState();
        final int rotation = getRotation(mContext);
        final int auto = getAuto(mContext);
        Log.i(TAG, "switchState rotation = " + rotation + " auto=" + auto);
        try {
            if (rotation != 0) {// land to auto
                setRotation(mContext, 0);
                setAuto(mContext, 1);
            } else if (auto != 0) {  // auto to port
                setRotation(mContext, 0);
                setAuto(mContext, 0);
            } else { // port to land
                setRotation(mContext, 1);
                setAuto(mContext, 0);
            }
        } catch (Exception e) {

        }
    }

    private static void setAuto(Context context,int auto) {
        Settings.System.putInt(context.getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, auto);
    }

    private static void setRotation(Context context,int rotation) {
        Settings.System.putInt(context.getContentResolver(), Settings.System.USER_ROTATION, rotation);
    }

    private static int getRotation(Context context) {
        return Settings.System.getInt(context.getContentResolver(), Settings.System.USER_ROTATION, 0);
    }

    private static int getAuto(Context context) {
        return Settings.System.getInt(context.getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, 0);
    }

}
```

### 1.7 WIFI

```java
public class WifiController extends BaseController {
    private WifiManager mWifiManager;

    @Override
    public void switchState() {
        int curState = mWifiManager.getWifiState();
        Log.i("lmq", "wifi switchState curState = " + curState);
        if (curState == WifiManager.WIFI_STATE_DISABLED || curState == WifiManager.WIFI_STATE_DISABLING) {
            mWifiManager.setWifiEnabled(true);
        } else {
            mWifiManager.setWifiEnabled(false);
        }
    }


}
```

### 1.8 免打扰

```java
public class ZenModeController extends BaseController {

    public static final String SETTINGS_KEY_ZEN_CODE = "zen_mode";
    public static final int ZEN_MODE_OFF = 0;
    public static final int ZEN_MODE_NO_INTERRUPTIONS = 2;
    private NotificationManager mNoMan;
    private static int sLastZenMode;

    public ZenModeController(Context context, IView iView) {
        super(context, iView);
    }

    @Override
    public void switchState() {
        final int mode = Settings.Global.getInt(mContext.getContentResolver(), SETTINGS_KEY_ZEN_CODE, 0);
        Log.i(TAG, "switchState zenMode=" + mode);
        if (mode != ZEN_MODE_OFF) {
            sLastZenMode = mode; // 保存上次的勿扰模式
            setZenMode(ZEN_MODE_OFF); // 关闭
        } else {
            sLastZenMode = sLastZenMode != ZEN_MODE_OFF ? sLastZenMode : ZEN_MODE_NO_INTERRUPTIONS; // 避免误数据导致无法恢复，设置一个默认值
            setZenMode(sLastZenMode); // 打开
        }
    }

    private void setZenMode(int mode) {
//        public void setZenMode(int mode, Uri conditionId, String reason)
        try {
            Method setZen = mNoMan.getClass().getDeclaredMethod("setZenMode", int.class, Uri.class, String.class);
            if (null != setZen) {
                setZen.invoke(mNoMan, mode, null, TAG); // 需要添加android.permission.STATUS_BAR_SERVICE权限，重启
                Log.i(TAG, "setZenMode success mode = " + mode);
            }
        } catch (Exception e) {
            Log.i(TAG, "setZenMode e = " + e);
        }
    }
}

```

### 1.9 监听系统设置变化

```java
public class SettingsValueChangeContentObserver extends ContentObserver {
    /**
     * Creates a content observer.
     *
     * @param handler The handler to run {@link #onChange} on, or null if none.
     */
    Context mContext;
    String mSettingsKey;
    ShortcutContainer.HandlerListener mHandlerListener;

    public SettingsValueChangeContentObserver(Context context, String settingsKey, ShortcutContainer.HandlerListener handlerListener) {
        super(new Handler());
        mContext = context;
        mSettingsKey = settingsKey;
        mHandlerListener = handlerListener;
    }

    @Override
    public void onChange(boolean selfChange) {
        super.onChange(selfChange);
        if (TextUtils.isEmpty(mSettingsKey)) {
            Log.i("lmq", "mSettingsKey is empty");
            return;
        }
        int data = 0;
        if (!USER_ROTATION.equals(mSettingsKey) && !SCREEN_BRIGHTNESS_MODE.equals(mSettingsKey) && !ACCELEROMETER_ROTATION.equals(mSettingsKey)) {
            data = Settings.Global.getInt(mContext.getContentResolver(), mSettingsKey, 0);
        } else {
            data = Settings.System.getInt(mContext.getContentResolver(), mSettingsKey, 0); // 屏幕旋转在system中
        }
        Log.i("lmq", "onchange data= " + data + " mSettingsKey=" + mSettingsKey + " mHandlerListener=" + mHandlerListener);
        if (mHandlerListener != null) {
            IView.EnabledState enabledState = IView.EnabledState.generateValue(data);
            if (ACCELEROMETER_ROTATION.equals(mSettingsKey)) {
                enabledState.setAutoRotation(true);
            }
            if (data != 0) {
                mHandlerListener.shortcutViewTurnOn(enabledState);
            } else {
                mHandlerListener.shortcutViewTurnOff(enabledState);
            }
        }
    }
}

```





## STAR

- S（情景）：这可以是让你参与这个项目，或者是解决问题的背景；
- T（任务）：只需要在简历上写清楚自己的职责；
- A（策略）：如何实现，是否遇到困难，如何解决的。
- R（结果）：取得的成果，**作为程序员，一定用数字说话。**

我们来看看面试中，我们如何利用好 START 法则。

- S（Situation）：在简历上我们呈现的是项目的背景，但在面试中，我们还应该就项目的细节进行更加详细的讲解。
- T（Task）：我们在简历上主要是编写自己在该项目中承担的职责，但在面试中，除了说明自己的职责，建议带上团队的整体任务，展现出无论何时，你都很在乎你的团队。
- A（Action）：在简历上我们展现的是应对问题采取的策略和具体方法，在面试中，除了进行详细说明以外，还应该说清楚自己和团队成员的分工，一定需要记住的是：**不能否认团队的价值。**
- R（Result）：产生的结果，这一点在面试中和简历上可以基本保持一致。
- T（Thinking）：这一个词是我自己添加的，我觉得一个好的介绍还应该举一反三，总结这个事情哪里做的好，哪里做的不好，接下来如何去避免这个问题，以及可以复用在哪些场景。



基于定制化需求开发，

自定义页面：显示摩奇圈的功能，

冻结解冻（摩奇管家会冻结白名单以外的应用（如何冻结？）），

1）应用启动的时候会解冻，每次这样效率低；8.0之后添加了常用应用，如果属于常用应用，设置一个解冻的过期时间

2）应用更新的时候会被冻结，在处理更新的地方重新load一遍

3）冻结的应用图标隐藏问题



桌面应用宝图标重复问题：

1）多个启动Activity导致，一个是快捷图标，提示用户

