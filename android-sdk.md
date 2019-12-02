## Android 各版本新特性

### Android Q

#### Q 行为变更：所有应用

不管 targetSdk是多少，对所有跑在 Q 设备上的应用均有影响

- 用户隐私权限变更

  - 存储权限

    Android Q 在外部存储设备为每个应用提供了一个“隔离存储沙盒”，简而言之就是应用专属文件夹，并且访问这个文件夹无需权限。使用`Context.getExternalFilesDir()`访问自己的文件夹，替换并取消了`READ_EXTERNAL_STORAG` 和 `WRITE_EXTERNAL_STORAGE`权限

  - 定位权限

    Q中也加入了后台位置权限 `ACCESS_BACKGROUND_LOCATION`，如果应用需要在后台时也获得用户位置(比如滴滴)，就需要动态申请`ACCESS_BACKGROUND_LOCATION`权限。targetSDK <= P 应用如果请求了`ACCESS_FINE_LOCATION` 或 `ACCESS_COARSE_LOCATION`权限，**Q**设备会自动帮你申请`ACCESS_BACKGROUND_LOCATION`权限。

  - 设备唯一标识符（IMEI）

    原来的`READ_PHONE_STATE`权限已经**不能获得IMEI和序列号**，官方所说的`READ_PRIVILEGED_PHONE_STATE`权限只提供给系统app。

- minSdk 警告

  谷歌要求运行在**Q**设备上的应用`targetSDK>=23`,不然会向用户发出警告。

#### Q 行为变更：targetSdk=29

- 非 SDK 接口限制

  **非SDK接口**限制在Android P中就已提出，但是在**Q**中，**被限制的接口的分类有较大变化**。

  **非SDK接口限制**就是某些`SDK`中的私用方法，如`private`方法，你通过**Java反射等方法**获取并调用了。那么这些调用将在`target>=P`或`target>=Q`的设备上被限制使用，当你使用了这些方法后，会报错。