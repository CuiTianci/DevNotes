
# AppBundle打包发布流程
## 打包阶段
### 签名
生成及配置签名的方式与此前打包apk的签名方式完全一致，遵循以下步骤：
- 根据项目签名模板生成签名文件
![项目签名模板](https://github.com/CuiTianci/DevNotes/blob/main/Android/AppBundle/src/%E9%A1%B9%E7%9B%AE%E7%AD%BE%E5%90%8D%E6%A8%A1%E6%9D%BF.png)
- 生成的签名文件为.jks格式文件，按公司要求将其转换为.keystore文件，推荐方式如下：
```terminal
bjmm100001@chengcaideiMac bar % keytool -importkeystore -srckeystore [SOURCE_JKS_PATH] -srcstoretype JKS -deststoretype PKCS12 -destkeystore [TARGET_KEYSTORE_PATH]
```
- 在项目中配置签名信息
```groovy
signingConfigs {
        release {
            storeFile file('../barcode.keystore')// 一般将签名文件置于项目第一级目录中。
            storePassword 'bar2021'
            keyAlias 'barcode'
            keyPassword 'bar2021'
        }
    }
```
### 打包
```terminal
./gradlew bundleRelease
```
通过上述方式，打包生成aab格式的文件，也就是用于发布到PlayStore的文件。
而为了方便进行测试，有时还需要从aab中导出apk文件，以供直接安装，这个阶段需要使用[BundleTool](https://github.com/google/bundletool/releases)
### 通过apks进行安装
- 导出apks文件
```terminal
java -jar [BUNDLE_TOOLS_JAR_PATH] build-apks --bundle=[SOURCE_AAB_PATH] --output=[TARGET_APKS_PATH] --ks=[KEYSTORE_PATH] --ks-pass=pass:[KEYSTORE_PASSWORD] --ks-key-alias=[KEYSTORE_ALIAS] --key-pass=pass:[KEYSTORE_ALIAS_PATH]
```
- 通过BundleTools将apks文件安装到设备中。
```terminal
bundletool install-apks --apks=[TARGET_APKS_PATH]
```
### 导出apk（更适合于与测试人员进行分享）
```terminal
java -jar [BUNDLE_TOOLS_JAR_PATH] build-apks --bundle=[SOURCE_AAB_PATH] --output=[TARGET_APKS_PATH] --mode=universal --ks=[KEYSTORE_PATH] --ks-pass=pass:[KEYSTORE_PASSWORD] --ks-key-alias=[KEYSTORE_ALIAS] --key-pass=pass:[KEYSTORE_ALIAS_PATH]
```
注：与之前的命令区别在于：增加了“--mode=universal”，其作用是：使bundletool只构建一个包含应用的所有代码和资源的APK，以使该APK与应用支持的所有设备配置兼容。
然后，解压生成的apks文件并取出universal.apk文件（修改为.zip文件，解压，更方便），即可进行直接安装和分享。

更多BundleTool的应用，请查看[BundleTool官方文档](https://developer.android.com/studio/command-line/bundletool)。

## 发布阶段
大体上，将aab文件交给运营即可。但在首次发布时，需要注意一个比较重要的问题：避免Google对应用进行重新签名，需要研发与运营配合完成，步骤如下：
- 默认情况下Google会对通过aab格式发布的应用进行重新签名，点击“更改应用签名密钥”：
![默认签名方式](https://github.com/CuiTianci/DevNotes/blob/main/Android/AppBundle/src/default.png)
- 建议使用下图中标注的方式生成并上传签名证书，命令行中的keystore即第一步生成的签名文件，pepk工具及命令行由运营提供。
![使用自己的签名](https://github.com/CuiTianci/DevNotes/blob/main/Android/AppBundle/src/%E6%9B%B4%E6%94%B9%E7%AD%BE%E5%90%8D.png)
- 查看修改结果
![修改结果](https://github.com/CuiTianci/DevNotes/blob/main/Android/AppBundle/src/result.png)

## 拓展链接
[About App Bundle](https://developer.android.com/guide/app-bundle)


