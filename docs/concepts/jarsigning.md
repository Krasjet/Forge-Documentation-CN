Jar签名
=======

Java允许开发者对他们的Jar文件签名。然而，这些签名并不是为了安全而设计的，而且也不应该这样使用。签名是用来检查完整性的，这样子开发者就能够确认是否在运行他们自己未修改的代码。

!!! note

	请再一次记住这个系统并不能作为一个安全手段。签名的检查能够被恶意手段所绕开。

创建一个密钥库
-------------

**密钥库**(Keystore)是一个私有的数据库文件，它包含了签名Jar文件所需的信息。要想签名一个Jar，你需要一个公钥和一个私钥。公钥将会在之后被用来检测Jar是否被正确签名。

要生成一个密钥的话，请执行下面的指令，并遵循 keytool 的指示。  
密钥需要是**SHA-1编码**(SHA-1 Encoded)的。

```shell
keytool -genkey -alias signFiles -keystore examplestore.jks
```

- `keytool` 是Java开发包(JDK)的一部分，它可以在 `bin` 目录下找到
- `alias signFiles` 指定了将来引用这个密钥库的别名(Alias)
- `-keystore examplestore.jks` 指的是密钥库将会被存储在 `examplestore.jks` 文件中

### 获取公钥

要获取Forge所需的公钥的话，执行下面的指令：

```shell
keytool -list -alias signFiles -keystore examplestore.jks
```

现在复制这个公钥，并删除所有的冒号以及将所有的大写字母改为小写字母来保证FML能够识别这个密钥。

### 运行时检查

如果要让FML对比密钥的话，将公钥添加到 `@Mod` 注解中的 `certificateFingerprint` 参数中。

当FML检测到密钥不匹配时，它将会触发一个持续整个Mod生命周期的时间。如何处理密钥不匹配这一事件将由开发者来决定。

- `event.isDirectory()` - 当Mod在开发环境下允许是返回true。
- `event.getSource()` - 返回不匹配密钥的文件。
- `event.getExpectedFingerprint()` - 返回公钥。
- `event.getFingerprints()` - 返回发现的所有公钥。

构建脚本设置
-----------

最后，要让Gradle使用生成的密钥对来签名Jar文件的话，我们需要在 `build.gradle` 中新创建一个任务(Task)。


```groovy
    task signJar(type: SignJar, dependsOn: reobfJar) {
        inputFile = jar.archivePath
        outputFile = jar.archivePath
    }
    
    build.dependsOn signJar
```

- `dependsOn: reobfJar` - 这个片段非常重要，因为Gradle必须要在ForgeGradle重混淆Jar**之后**来对Jar签名。
- `jar.archivePath` - 档案(Jar)构造时所处的路径。
- `build.dependsOn signJar` - 这一行告诉Gradle这个任务是 `build` 任务的一部分，通过 `gradlew build`来启动。

下面 `signJar` 中的这些值必须要有定义，来保证Gradle能够找到密钥库从而给Jar签名。这必须要在 `gradle.properties` 文件中来定义。

- `keyStore` - 这个值告诉Gradle该去哪里找之前生成的密钥库。
- `alias` - 必须要提供之前定义的那个别名来让Gradle对Jar签名。
- `storePass` - 创建密钥库时使用的密码，Gradle需要它来访问这个文件。
- `keyPass` - 创建密钥库时生成的密钥的密码，Gradle需要它来访问这个文件。这个密码可能与 `storePass` 相同，但也可能不同。