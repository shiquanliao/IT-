# VirtualAPK分析

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

