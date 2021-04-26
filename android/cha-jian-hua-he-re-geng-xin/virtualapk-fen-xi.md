# VirtualAPK分析



## 加载类

```java
protected ClassLoader createClassLoader(Context context, File apk,
                                   File libsDir, Classloader parent) throw Exception{
    File dexOutputDir = getDir(context, Constants.OPTMIZE_DIR);
    String dexOutputPath = dexOutputDir.getAbsolutePath();
    DexClassloader loader = new DexClassloader(apk.getAbsolutedPath(),
                     dexOutputPath, libsDir.getAbsolutePath(), parent);
    // parent是宿主的classloader
    if(Constants.COMBINE_CLASSLOADER){
        DexUtil.insertDex(loader, parent, libsDir);
    }
    reutnr loader;
}

// 调用
createClassLoader(context, apk, this.mNativeLibDir, context.getClassLoader());
```

```java
public static void insertDex(DexClassLoader dexClassLoader, 
         ClassLoader baseClassLoader, File nativeDir) throw Exception{
    Object baseDexElements = getDexElements(getPathList(baseClassLoader));
    Ojbect newDexElements = getDexElements(getPathList(dexClassLoader));
    Ojbect allDexElements = combineArray(baseDexElements, newDexElements);
    Object pathList = getPathList(baseClassLoader);
    Reflector.with(pathList).field("dexElements").set(allDexElements);
    
    insertNativeLibrary(dexClassLoader, baseClassLoader, nativeDir);
}
```

## 加载资源

```java
protected Resources createResources(Context context, String packageName,
                                   File apk) throw Exception{
    if(Constants.COMBINE_RESOURCES){
        returnt ResourcesManager.createResources(context, packageName, apk);
    }else{
        Resources hostResources = context.getResources();
        AssetManager assetManager = createAssetManager(context, apk);
        return new Resources(assetManager, hostResources.getDisplayMetrics(), 
                            hostResources.getConfiguration());
    }
}
```

## 加载Activity

```java
// 发送之前的替换
public void markIntentIfNeed(Intent intent){
    ...
    String targetPackageName = intent.getComponent().getPackageName();
    String targetClassName = intent.getComponent().getClassName();
    if(!targetPackageName.equals(mContext.getPackageName()) &&
      mPluginManager.getLoaderPlugin(targetPackageName) != null){
        intent.putExtra(Constants.KEY_IS_PLUGIN, true);
        intent.puxExtra(Constants.KEY_TARGET_PACKAGE, targetPackageName);
        intent.puxExtra(Constants.KEY_TARGET_ACTIVITY, targetClassName);
        dispatchStubActivity(intent);
    }
}
```

```java
// 启动之前的还原
public static ComponentName getComponent(Intent intent){
    if(intent == null) return null;
    if(isIntentFromPlugin(intent)){
        return new ComponentName(
        	intent.getStringExtra(Constants.KEY_TARGET_PACKAGE);
        	intent.getStringExtra(Constants.KEY_TARGET_ACTIVITY);
        );
    }
    return intent.getComponent();
}
```



## 加载Service

```java
protected void hookSystemService(){
    Singleton<IActivityManager> defalutSingleton;
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
        defalutSingleton = Reflector.on(ActivityManager.class)
            .field("IActivityManagerSingleton").get();
    }else{
        defalutSingleton = Reflector.on(ActivityManagerNative.class)
            .field("gDefault").get();
    }
    IActivityManager origin = defaultSingleton.get();
    IActivityManager activityManagerProxy = (IActivityManager)Proxy.newProxyInstance(
    	mContext.getClassLOader(), new Class[]{IActivity.class},
    	createActivityManagerProxy(origin));
    Reflector.with(defaultSingleton).field("mInstance").set(activityManagerProxy);
}
```

```java
protected Intent wrapperTargetIntent(Intent target, ServiceInfo serviceInfo, 
                                     Bundle extra, int command){
    target.setComponent(new ComponnentName(serviceInfo.packageName, 
                                           serviceInfo.name));
    String pluginLocation = mPluginManager.getLoadedPlugin(target.getComponent())
        .getLocation();
    boolean local = PluginUtil.isLocalService(serviceInfo);
    Class<? extends Service> delegate = local ? LocalService.class 
        : RemoteService.class;
    Intent intent = new Intent();
    intent.setClass(mPluginManager.getHostContext(), delegate);
    intent.putExtra(RemoteService.EXTRA_TARGET, target);
    intent.putExtra(RemoteService.EXTRA_COMMAND, command);
    intent.putExtra(RemoteService.EXTRA_PLUGIN_LOCATION, pluginLocation);
    if(extra != null) intent.putExtras(extras);
    return intent;
}
```

```java
public int onStartCommand(Intent intent, int flags, int startId){
    Intent target = intent.getParcelableExtra(EXTRA_TARGET);
    target.setExtrasClassLoader(plugin.getClassLoader());
    ...
    case EXTRA_COMMAND_START_SERVICE{
    	ActivityThread mainThread = ActivityThread.currentActivityThread();
        IApplicationThread appThread = mainThread.getApplicationThread();
        Service service;
        service = (Service)plugin.getClassLoader()
            .loadClass(component.getClassLoaderName()).newInstance();
        Method attach = service.getClass().getMethod("attach", ...);
        IActivityManager am = mPluginManager.getActivityManager();
        attach.invoke(service, plugin.getPluginContext(),
                      mainThread, component.getClassName(), 
                      token, app, am);
        
        service.onStartCommand(target, 0, ...);
    }
}
```



## 加载广播

* 解析插件Manifest, 静态广播转动态注册
* 插件广播在宿主未启动时无法被外部唤醒

