1. 首先编写一个可以响应外部链接唤起的 Activity. 并且配置好对应的 Intent-filter

2. 在 Activity 中得到外部链接, 交由 Component 路由处理

```xml
<activity android:name=".view.ProxyAct">
  <intent-filter>
    <data android:scheme="<这里是你的 scheme>" />
    <action android:name="android.intent.action.VIEW" />

    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
  </intent-filter>
</activity>
```

```kotlin
class ProxyAct : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.proxy_act)
    val url = intent.data.toString()
    // 转给 Component
    Router.with(this)
    .url(url)
    .forward()
  }
}
```