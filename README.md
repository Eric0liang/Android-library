# 使用AS把Android library分享到jCenter和Maven Central
原文地址 https://github.com/Eric0liang/Android-library/blob/master/README.md

## jCenter
jcenter是一个由 bintray.com维护的Maven仓库</br>
我们在build.gradle文件中如下定义仓库，就能使用jcenter了：

    allprojects {
        repositories {
            jcenter()
        }
    }
    
## Maven Central
Maven Central 则是由sonatype.org维护的Maven仓库</br>
同样在build.gradle文件中如下定义仓库
```groovy
    allprojects {
        repositories {
            mavenCentral()
        }
    }
 ```
不管是jcenter还是Maven Central ，两者都是Maven仓库</br>
新版本的Android Studio创建一个项目，jcenter()自动被定义，而不是mavenCentral()，由于Maven Central的最大问题是对开发者不够友好，
上传library异常困难，而使用jcenter的好处多：
* jcenter是全世界最大的Java仓库
* 上传library到仓库很简单，不需要像在 Maven Central上做很多复杂的事情
* 友好的用户界面
* jcenter一个按钮就能实现可以将library上传到Maven Central</br>

所以说使用jcenter是明智之举

## 第一部分：在bintray上创建package

第一步：在[bintray.com](https://bintray.com)上注册账号(有GitHub账号可以直接登录)，完成注册之后，登录网站，然后点击Add New Repository

<img src="images/1.png" width="600px"/>

第二步：点击Add New Package，为library创建一个新的package

<img src="images/2.png"/>

<img src="images/3.png" width="500px"/>

<img src="images/4.png" />

到这里Jcenter的Bintray账户的注册就完成了，并创建了Package。

## 第二部分：为MavenCentral在Sonatype上创建账号
需要在[Sonatype Dashboard](https://issues.sonatype.org/secure/Dashboard.jspa)上申请个Issue权限，它的作用就是允许你上次匹配Maven Central提供的GROUP_ID的library</br>
创建的帐号登录，然后点击顶部菜单的Create，填写信息如下：</br>
Project: Community Support - Open Source Project Repository Hosting</br>
Issue Type: New Project</br>
Summary: 你的 library名称的概要，比如The Base Library。</br>
Group Id: 输入根GROUP_ID，比如，com.github.eric0liang 。一旦批准之后，每个以com.github.eric0liang</br>
开始的library都允许被上传到仓库，比如com.github.eric0liang.lib。</br>
Project URL: 输入任意一个你想贡献的library的URL，比如， https://github.com/Eric0liang/cardocr。</br>
SCM URL: 版本控制的URL，比如 https://github.com/Eric0liang/cardocr.git。</br>
<img src="images/5.png" />
<img src="images/7.png" />
点击Create后就是等待的审核时间，大概1个小时左右申请通过后，就有权限把library分享到Maven Central

<img src="images/8.png" />

最后把Sonatype OSS用户名绑定到Bintray

<img src="images/9.png" />


## 第三部分：启用bintray里的自动签名

打开终端输入（如果你用的是windows，请安装cygwin环境）
注：gpg即GNU Privacy Guard，它是加密工具PGP（Pretty Good Privacy ）的非商业化版本，用于对Email、文件及其他数据的收发进行加密与验证，确保通信数据的可靠性和真实性
### gpg安装
```groovy
    brew install gpg
```
### 生成key
有几个必填项。部分可以采用默认值，但是某些项需要你自己输入恰当的内容，比如，你的真实名字，密码 等等。
```groovy
    gpg --gen-key
```
最终公钥和私钥已经生成并经签名。
```groovy
pub   rsa2048 2017-11-09 [SC] [有效至：2019-11-09]
      8BE15DECD54188B1A5E0BCAF1D32355F7D533BF6
uid                      yourname <yourmail@email.com>
sub   rsa2048 2017-11-09 [E] [有效至：2019-11-09]
```

### 上传至公钥服务器
现在你需要把key上传到keyserver。为此，请将PUBLIC_KEY_ID替换成自己的keyId,譬如8BE15DECD54188B1A5E0BCAF1D32355F7D533BF6
```groovy
    gpg --keyserver hkp://pool.sks-keyservers.net --send-keys PUBLIC_KEY_ID
```

在发布过程中我遇到了 gpg: keyserver send failed: No route to host的错误，解决方法是多试几个地址，下面是几个候选地址：

* p80.pool.sks-keyservers.net:80
* pool.sks-keyservers.net
* ha.pool.sks-keyservers.net

### 导出公钥和私钥

    gpg -a --export yourmail@email.com > public_key_sender.asc
    gpg -a --export-secret-key yourmail@email.com > private_key_sender.asc
    
把刚才导出的公钥和私钥粘贴进去，包括虚线的两行文本，然后点击 Update

<img src="images/10.png" />
<img src="images/11.png" />

Update完密钥后就可以开启自动签名功能
<img src="images/12.png" />

<img src="images/13.png" />

点击Update保存这些步骤。完成。现在只需点击一下，每个上传到我们Maven仓库的东西都会自动签名并做好转向Maven Central 。
请注意这是一次性的操作，以后创建的每一个library都要应用此操作。</br>
Bintray和Maven Central都已经准备好了！！！！


## 第四部分：创建Android Studio项目
Application Module用于展示库的用法</br>
Library Module是library的源代码，也就是要发布到jcenter的library

<img src="images/14.png" />

### 在项目的build.gradle中添加bintray插件如下：
```groovy
    classpath 'com.android.tools.build:gradle:2.2.1'
    classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
    classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:1.6"
```
注：gradle、android-maven-gradle-plugin、gradle-bintray-plugin一定要对应匹配版本号，否则会出现很多不可思议的bug

### local.properties添加如下：

    bintray.user=YOUR_BINTRAY_USERNAME
    bintray.apikey=YOUR_BINTRAY_API_KEY
    bintray.gpg.password=YOUR_GPG_PASSWORD
    
local.properties文件一般被添加到.gitignore了，因此这些敏感数据不会被误传到git服务器</br>
*YOUR_BINTRAY_USERNAME* bintray账户登录名</br>
*YOUR_BINTRAY_API_KEY* API Key可以在Edit Profile页面的API Key 选项卡中找到</br>
*YOUR_GPG_PASSWORD* GPG key的密码</br>

### 在Module Library的build.gradle中添加如下：
```groovy
apply plugin: 'com.android.library'
ext {
    bintrayRepo = 'maven'
    bintrayName = 'lib_cardocr'

    publishedGroupId = 'com.github.eric0liang'
    libraryName = 'LibCardocr'
    artifact = 'lib_cardocr'

    libraryDescription = 'idcard and bankcard ocr'

    siteUrl = 'https://github.com/Eric0liang/cardocr'
    gitUrl = 'https://github.com/Eric0liang/cardocr.git'

    libraryVersion = '0.9.3'

    developerId = 'ericliang'
    developerName = 'Eric Liang'
    developerEmail = 'youmail@email.com'

    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]
}
apply from: 'https://raw.githubusercontent.com/Eric0liang/Android-library/master/maven/installv1.gradle'
apply from: 'https://raw.githubusercontent.com/Eric0liang/Android-library/master/maven/bintrayv1.gradle'
```

bintrayName修改成你上面创建的 package name。其余的项也修改成和你library信息相匹配的值</br>
publishedGroupId artifact libraryVersion 就是定义jcenter的引用路径
```groovy
compile 'com.github.eric0liang:lib_cardocr:0.9.3'
```
最后在文件的后面追加两行代码来应用两个脚本，用于构建library文件和上传文件到bintray</br>
为了方便，我直接使用了github上连接到相关文件的链接，这两个脚本不用拷贝到项目，其他项目也可以直接引用
### 上传编译的文件到bintray
使用如下的命令：
```groovy
gradle bintrayUpload
```
上传成功后在bintray查看文件
<img src="images/18.png" />
点击Add to Jcenter的按钮发布到Jcenter(没有出现按钮就稍等一下)
<img src="images/15.png" />
发布后等待审核，审核通过后Linked To会看到一些变化
<img src="images/17.png" />

到这里，任何开发者都可以使用jcenter() repository 外加一行gradle脚本来使用我们的library了

    compile 'com.github.eric0liang:lib_cardocr:0.9.3'

在[Jcenter](http://jcenter.bintray.com)上检查library，在本例中就是http://jcenter.bintray.com/com/github/eric0liang/lib_cardocr/0.9.3/
<img src="images/19.png" />

请注意链接到jcenter是一个只需做一次的操作。如果你对你的package做了任何修改，比如上传了一个新版本的binary，删除了旧版本的binary等等，这些改变也会影响到jcenter。不过毕竟你自己的仓库和jcenter在不同的地方，所以需要等待2－3分钟让jcenter同步这些修改。

同时注意，如果你决定删除整个package，放在jcenter仓库上的library不会被删除。它们会像僵尸一样的存在，没有人再能删除它了。因此我建议，如果你想删除整个package，请在移除package之前先在网页上删除每一个版本。

## 第五部分：上传library到Maven Central
如果前面工作都准备好了，上传到Maven Central只是几秒钟的事情
<img src="images/20.png" />

上传成功后在[ Maven Central Repository](https://oss.sonatype.org/content/repositories/releases)上找到你上传的library，在本例中就是https://oss.sonatype.org/content/repositories/releases/com/github/eric0liang/lib_cardocr/0.9.3/
<img src="images/21.png" />

最后，虽然需要许多步骤，但是大部分操作都是一劳永逸的。

### 其它问题

发现任何问题可以提issue

------

## 关于作者

#### 联系方式

* Github: <https://github.com/Eric0liang>
* Email: [liangjiangli@bluemoon.com.cn](mailto:liangjiang@bluemoon.com.cn)

------

## License

    Copyright 2017 - 2019 Jiangli Liang

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.





