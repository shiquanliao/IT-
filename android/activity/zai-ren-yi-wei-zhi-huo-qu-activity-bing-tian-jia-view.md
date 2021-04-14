# 在任意位置获取activity并添加view

获取当前的Activity

```java
public class ActivityManager{
    private static WeakReference<Activity> currentActivityRef;
    private static ActivityLifecycleCallbacks callbacks = new ActivityLifecycleCallbacks(){
        @Override
        public void onActivityCreated(Activity, Bundle saveInstanceState){
            currentActivityRef = new WeakReference<>(activity);
        }
    };
}
```

添加view

```java
public class ActivityExt{
    private final Activity activity;
    private final ViewGroup contentParent;
    public ActivityExt(Activity activity){
        this.activity = activity;
        contentParent = (ViewGroup)activity.findViewById(android.R.id.content);
    }
    
    public void addContentView(View view, ViewGroup.LayoutParams params){
        contentParent.addView(view, params);
    }
    
    public void removeContentView(View view){
        contentParent.removeView(view);
    }
}
```

