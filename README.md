# 如何使用Android Studio把自己的Android library分享到jCenter和Maven Central

这个库的底层是使用腾讯优图云平台识别技术，识别速度大概4秒左右 [`http://open.youtu.qq.com/`](http://open.youtu.qq.com/) 

## 第一部分：在bintray上创建package

第一步：在[bintray.com](https://bintray.com)上注册账号(有GitHub账号可以直接登录)，完成注册之后，登录网站，然后点击Add New Repository

<img src="https://github.com/Eric0liang/Android-library/blob/master/1.png" width="600px"/>

第二步：点击Add New Package，为library创建一个新的package

<img src="https://github.com/Eric0liang/Android-library/blob/master/2.png"/>

<img src="https://github.com/Eric0liang/Android-library/blob/master/3.png" width="500px"/>

<img src="https://github.com/Eric0liang/Android-library/blob/master/4.png" />

到这里Jcenter的Bintray账户的注册就完成了，并创建了Package。

## 第一部分：为MavenCentral在Sonatype上创建账号
需要在Sonatype Dashboard上申请个Issue权限，它的作用就是允许你上次匹配Maven Central提供的GROUP_ID的library</br>
创建的帐号登录，然后点击顶部菜单的Create，填写信息如下：</br>
Project: Community Support - Open Source Project Repository Hosting</br>
Issue Type: New Project</br>
Summary: 你的 library名称的概要，比如The Base Library。</br>
Group Id: 输入根GROUP_ID，比如，cn.com.bluemoon 。一旦批准之后，每个以cn.com.bluemoon</br>
开始的library都允许被上传到仓库，比如cn.com.bluemoon.lib。</br>
Project URL: 输入任意一个你想贡献的library的URL，比如， https://github.com/Eric0liang/cardocr。</br>
SCM URL: 版本控制的URL，比如 https://github.com/Eric0liang/cardocr.git。</br>
<img src="https://github.com/Eric0liang/Android-library/blob/master/5.png" />
<img src="https://github.com/Eric0liang/Android-library/blob/master/6.png" />
<img src="https://github.com/Eric0liang/Android-library/blob/master/7.png" />

```groovy
    allprojects {
        jcenter()
        maven { url "https://raw.githubusercontent.com/Eric0liang/lib_cardocr/master" }
    }
    
```

```groovy
    compile 'cn.com.bluemoon:lib_cardocr:1.0.2'
```
### 依赖的jar添加到libs
[fastjson.jar](https://github.com/Eric0liang/cardocr/blob/master/app/libs/fastjson-1.2.6.jar)

[BASE64Decoder.jar](https://github.com/Eric0liang/cardocr/blob/master/app/libs/sun.misc.BASE64Decoder.jar)

## demo apk下载地址: 
[点击下载](https://github.com/Eric0liang/cardocr/blob/master/app-debug.apk)

## 效果
<img src="https://github.com/Eric0liang/cardocr/blob/master/images/2.png" width="400px"/>       <img src="https://github.com/Eric0liang/cardocr/blob/master/images/3.png" width="400px"/>

<img src="https://github.com/Eric0liang/cardocr/blob/master/images/4.png" width="400px"/>       <img src="https://github.com/Eric0liang/cardocr/blob/master/images/1.png" width="400px"/>

## 使用指南（2017.11.7更新）

**使用前请阅读对应模块的文档和示例，如果有不清楚的地方，可以看源码，或者向我提问。**

### CaptureActivity 识别身份证、银行卡照相机类

#### API
startAction(Activity context, CardType type, int requestCode) </br>
startAction(Activity context, CardType type, String url, int requestCode) </br>
startAction(Activity context, CardType type, @StringRes int titleId, int requestCode)</br>

* context 调起照相机的activity类
* type 枚举类，有三个类型
```groovy
   public enum CardType {
    //身份证头像面,身份证国徽面,银行卡
    TYPE_ID_CARD_FRONT, TYPE_ID_CARD_BACK,TYPE_BANK
   }
```
* titleId 自定义照相机顶部的title，比如<string name="txt_id_card_title">请确保身份证头像面边缘在框内</string>
* url 是否需要保存身份证的截图，传保存的文件夹路径，比如Environment.getExternalStorageDirectory() + "/images"
* requestCode onActivityResult使用

#### 使用demo MainActivity代码片段
```groovy
    @Override
    public void onClick(View v) {
        String dirPath = Environment.getExternalStorageDirectory() + "/images";
        switch (v.getId()) {
            case R.id.btn_id_card_back:
                CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_BACK, checkbox.isChecked() ? dirPath : null, 0);
                //CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_BACK, 0);
                //CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_BACK, R.string.txt_id_card_title, 0);
                break;
            case R.id.btn_id_card_front:
                CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_FRONT, checkbox.isChecked() ? dirPath : null, 0);
                //CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_FRONT, 0);
                //CaptureActivity.startAction(this, CardType.TYPE_ID_CARD_FRONT, R.string.txt_id_card_title, 0);
                break;
            case R.id.btn_bank:
                CaptureActivity.startAction(this, CardType.TYPE_BANK, 1);
                break;
        }
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == RESULT_OK) {
            imgIdCard.setVisibility(View.GONE);
            if (requestCode == 0) {
                IdCardInfo info = (IdCardInfo)data.getSerializableExtra(CaptureActivity.BUNDLE_DATA);
                txtInfo.setText(info.toString());
                if (!TextUtils.isEmpty(info.getImageUrl())) {
                    imgIdCard.setVisibility(View.VISIBLE);
                    imgIdCard.setImageBitmap(BitmapFactory.decodeFile(info.getImageUrl()));
                }

            } else {
                BankInfo info = (BankInfo)data.getSerializableExtra(CaptureActivity.BUNDLE_DATA);
                txtInfo.setText(info.toString());
            }
        }
    }
```
#### IdCardInfo

    private String authority; //签发机关，XXX公安局
    private String validDate; //有效期限，200702.14-2017.02.14
    private String imageUrl; //截图保存地址
    private String name; //姓名
    private String sex; //性别
    private String nation; //民族
    private String birth; //出生
    private String address; //住址
    private String id; //公民身份号码

#### BankInfo

    private String bankInfo; //银行信息，农业银行
    private String bankName; //卡名字，金穗通宝卡(银联卡)
    private String bankType; //卡类型，借记卡
    private String bankNumber; //卡号，6228475757548

### 自定义照相机界面

可以参考CoustomCaptureActivity类，继承BaseCaptureActivity，并重写getLayoutId，initCustomView两个方法即可。
```groovy
    @Override
    public int getLayoutId() {
        return R.layout.activity_coustom;
    }

    @Override
    public void initCustomView() {
        Button btnBack = (Button)findViewById(R.id.btn_back);
        Button btnTake = (Button)findViewById(R.id.btn_take);
        View.OnClickListener listener = new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int id = v.getId();
                if (id == R.id.btn_back) {
                    finish(); //关闭
                } else if (id == R.id.btn_take) {
                    identification(); //拍照识别
                }
            }
        };
        btnBack.setOnClickListener(listener);
        btnTake.setOnClickListener(listener);
    }
```

## 更新记录
- **1.0.2** 2017.11.8
	* 添加友好提示
- **1.0.1** 2017.11.8
	* 修复断网的时候一直在loading
	* 重命名meta-data
- **1.0.0** 2017.11.7
	* first commit

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





