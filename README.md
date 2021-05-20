 # Intro
 项目原始地址: https://github.com/jeasonlzy/ImagePicker    最新版到0.6.1
 二次维护地址：https://github.com/CysionLiu/ImagePicker     最新版到1.2.1.Q    android api 29
 
 由于项目历史问题，使用了 jeasonlzy 的lib，而后切到 CysionLiu 的lib； 现在发现 适配 30 还有新的问题...
 好吧，我也fork一下，接个力。
 
 
 
origination --------
 
# ImagePicker
Android自定义相册，仿微信UI，实现了拍照、图片选择（单选/多选）、裁剪等功能。

## 支持AndroidX(master分支),支持AndroidQ(feature分支),老版本见dev分支。

### 注意：若没有用到AndroidX，则不支持targetSdk>=29

### 注意：Q版本的资源获取是通过ImageItem的uri,其它版本通过path

### 注意：最新依赖库更新于2020-05-26

 对于Android Studio(建议用3.0版本+)的用户，可以选择添加:

```

//老版本，非androidx，targetsdk<29:

api 'com.cysion:ImagePicker:1.2.0'

-----------

若使用androidx:
    //target sdk <29，建议这样添加依赖：

    api 'com.cysion:ImagePicker:1.2.0.x'

    //若targetsdk>=29 ,则需要这样添加依赖：
    api 'com.cysion:ImagePicker:1.2.1.Q'
注意，Android Q 对存储框架有较大改动，最主要的是无法通过文件路径获得非*应用专有文件*。
在本版本库中，也完全放弃了文件路径的方式，全部是以Uri的方式提供文件访问。


## 演示
 ![image](https://github.com/jeasonlzy/Screenshots/blob/master/ImagePicker/demo1.png)![image](https://github.com/jeasonlzy/Screenshots/blob/master/ImagePicker/demo2.gif)


## 1.用法
对于Android Studio(建议用3.0版本+)的用户，可以选择添加:

```
//老版本至此不再维护
api 'com.cysion:ImagePicker:1.2.0'

//若使用androidx，则需要这样添加依赖：
api 'com.cysion:ImagePicker:1.2.0.x'

---

//若出现Failed to resolve: com.github.chrisbanes:PhotoView的问题，
//则应在项目的build.gradle添加如下：
 maven{url"https://jitpack.io"}

```


## 2.功能和参数含义


|配置参数|参数含义|
|:--:|--|
|multiMode|图片选着模式，单选/多选|
|selectLimit|多选限制数量，默认为9|
|showCamera|选择照片时是否显示拍照按钮|
|crop|是否允许裁剪（单选有效）|
|isFreeCrop|是否允许自由裁剪(单选有效,默认FREE,自由比例)新版本添加，推荐使用，**会覆盖crop设置**|
|style|有裁剪时，裁剪框是矩形还是圆形|
|focusWidth|矩形裁剪框宽度（圆形自动取宽高最小值）|
|focusHeight|矩形裁剪框高度（圆形自动取宽高最小值）|
|outPutX|裁剪后需要保存的图片宽度,仅对crop有效，对isFreeCrop无效|
|outPutY|裁剪后需要保存的图片高度,仅对crop有效，对isFreeCrop无效|
|isSaveRectangle|裁剪后的图片是按矩形区域保存还是裁剪框的形状，例如圆形裁剪的时候，该参数给true，那么保存的图片是矩形区域，如果该参数给fale，保存的图片是圆形区域|
|imageLoader|需要使用的图片加载器，自需要实现ImageLoader接口即可,推荐glide|
|isOrigin|选择的图片是否采用原图，在图片选择好之后根据此参数判断，android Q版本|
|setCropCacheFolder|设置裁剪图片的存储位置，建议设置为应用专有目录|

## 3.代码参考

更多使用，请下载demo参看源代码

1. 首先你需要继承 `com.lzy.imagepicker.loader.ImageLoader` 这个接口,实现其中的方法,比如以下代码是使用 `Picasso` 三方加载库实现的

**picasso 2.52版本有bug**


```java
public class PicassoImageLoader implements ImageLoader {

    @Override
    public void displayImage(Activity activity, String path, ImageView imageView, int width, int height) {
        Picasso.with(activity)//
                   .load(Uri.fromFile(new File(path)))//
                .placeholder(R.mipmap.default_image)//
                .error(R.mipmap.default_image)//
                .resize(width, height)//
                .centerInside()//
                .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)//
                .into(imageView);
    }

    @Override
    public void clearMemoryCache() {
        //这里是清除缓存的方法,根据需要自己实现
    }
}


Android Q 版本，废弃了文件路径访问方式：


public class PicassoImageLoader implements ImageLoader {

    @Override
    public void displayImage(Activity activity, Uri aUri, ImageView imageView, int width, int height) {
        Picasso.get()//
                .load(aUri)//
                .placeholder(R.drawable.ic_default_image)//
                .error(R.drawable.ic_default_image)//
                .resize(width, height)//
                .centerInside()//
                .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)//
                .into(imageView);
    }

    @Override
    public void clearMemoryCache() {
    }
}

```

2. 然后配置图片选择器，一般在Application初始化配置一次就可以,这里就需要将上面的图片加载器设置进来,其余的配置根据需要设置
```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_image_picker);
    
    ImagePicker imagePicker = ImagePicker.getInstance();
    imagePicker.setImageLoader(new PicassoImageLoader());   //设置图片加载器
    imagePicker.setShowCamera(true);  //显示拍照按钮
    imagePicker.setCrop(true);        //允许裁剪（单选才有效）
    imagePicker.setFreeCrop(true, FreeCropImageView.CropMode.FREE);//新版添加,自由裁剪，优先于setCrop
    imagePicker.setSaveRectangle(true); //是否按矩形区域保存
    imagePicker.setSelectLimit(9);    //选中数量限制
    imagePicker.setStyle(CropImageView.Style.RECTANGLE);  //裁剪框的形状
    imagePicker.setFocusWidth(800);   //裁剪框的宽度，仅对crop有效，对isFreeCrop无效
    imagePicker.setFocusHeight(800);  //裁剪框的高度，仅对crop有效，对isFreeCrop无效
    imagePicker.setOutPutX(1000);//保存文件的宽度。单位像素，仅对crop有效，对isFreeCrop无效
    imagePicker.setOutPutY(1000);//保存文件的高度。单位像素，仅对crop有效，对isFreeCrop无效
    imagePicker.setIToaster(this, new InnerToaster.IToaster());//设置吐司代理,保持lib与app中吐司风格一致
}
```

3. 以上配置完成后，在适当的方法中开启相册，例如点击按钮时
```java
public void onClick(View v) {
        Intent intent = new Intent(this, ImageGridActivity.class);
        startActivityForResult(intent, IMAGE_PICKER);  
    }
}
```

4. 如果你想直接调用相机
```

//1.0.5版本加入了更明确的权限申请，解决原程序直接打开拍照可能出现没数据的情况

Intent intent = new Intent(this, ImageGridActivity.class);
intent.putExtra(ImageGridActivity.EXTRAS_TAKE_PICKERS,true); // 是否是直接打开相机
startActivityForResult(intent, REQUEST_CODE_SELECT);

//若拍照之后需要裁剪，则在使用前，需要关闭多选模式，支持裁剪

ImagePicker.getInstance().setMultiMode(false);
ImagePicker.getInstance().setFreeCrop(true, FreeCropImageView.CropMode.FREE);
...

```

5. 重写`onActivityResult`方法,回调结果
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode == ImagePicker.RESULT_CODE_ITEMS) {
        if (data != null && requestCode == IMAGE_PICKER) {
            ArrayList<ImageItem> images = (ArrayList<ImageItem>) data.getSerializableExtra(ImagePicker.EXTRA_RESULT_ITEMS);
            MyAdapter adapter = new MyAdapter(images);
            gridView.setAdapter(adapter);
        } else {
            Toast.makeText(this, "没有数据", Toast.LENGTH_SHORT).show();
        }
    }
}
```

## 更新日志


V1.2.1

 * 修复Q版本加载图片时潜在的空指针
 * 对于【androidX  +  targetsdk>=29】,这样依赖api 'com.cysion:ImagePicker:1.2.1.Q'



V1.2.0

 * 统一library版本；老版本1.2.0;   andx:1.2.0.x;  targetsdk >= 29：1.2.0.Q
 * 优化配置裁剪图片存储逻辑;


V1.0.9.Q

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.9.Q'
 * 适配Android Q,完全以Uri的方式获得图片，废弃文件路径;
 * 增加原图选项；


V1.0.8

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.8'
 * 修复没有图片时不显示相机icon的bug;


V1.0.7

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.7'
 * 修复单选 裁剪返回--拍照--裁剪的图片bug;
 * 加入androidx支持，依赖版本为api 'com.cysion:ImagePicker:1.0.7.x'


V1.0.6

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.6'
 * 修复在预览页面，点击删除出现对话框时的中文乱码问题


V1.0.5

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.5'
 * 加入了更明确的权限申请，解决原程序直接打开拍照可能出现没数据的情况
 * 增加 点击直接拍照时，选择单选模式，并支持裁剪的说明


V1.0.1

 * 依赖方式改为  api 'com.cysion:ImagePicker:1.0.1'
 * 支持中英文下的多语言支持(若翻译不符合，可通过覆盖res-string的方式);
 * 增加无摄像头设备拍照时的校验;
 * 增加吐司注入，保持lib与app中吐司风格一致;
 * 解决图片裁剪时点击取消，返回后列表间隔变大的bug;
 * 加入一种新的裁剪方式--自由裁剪，可以按任意比例拖动裁剪;
 * 修复使用原裁剪时，取消后会多选一张图片的bug;

V1.0.0

 * 新作者重新发布，依赖方式改为  api 'com.cysion:ImagePicker:1.0.0'
 * 更新至Android sdk 27;
 * 修复loader生命周期造成的index问题，即图片列表页闪退和黑屏问题(预览返回和桌面重回场景)

V 0.6.1
 * [合并] [优化图片选择页UI， 适配预览页的横竖屏切换 #195](https://github.com/jeasonlzy/ImagePicker/pull/195)

V 0.6.0
 * [合并] [调整UI,真正的完全仿微信](https://github.com/jeasonlzy/ImagePicker/pull/193)
 * [合并] [fix(location): 解决不合法图片导致的Bug](https://github.com/jeasonlzy/ImagePicker/pull/188)

V 0.5.5
 * [修复]选择图页面进入预览取消选择或者选择后返回列表不更新的问题；
 * [修复]6.0动态权限可能导致崩溃的bug；

V 0.5.4
 * [修复]部分内存泄漏问题；
 * [修复]新增显示已选中图片的调用方法：详情请查看demo首页的ImagePickerActivity；如果不需要此功能，则WxDemoActivity可能是你想要的。

V 0.5.3
 * [修复]矫正图片旋转导致的oom；
 * [修复]部分手机TitleBar和状态栏重复的问题；

V 0.5.1
 * [更正] 由于原图功能其实还没有做，所以本版本隐去了原图的显示。以免用户误解原图问题。
 * [修复] 使用RecyclerView替换GridView解决改变选中状态全局刷新的问题；
 * [提示] 虽然本次解决了全局刷新，但是如果使用的是Picasso依然会出现重新加载一张图片的问题，这是Picasso自己的问题，建议使用Glide框架。
 
V 0.5.0
 * [修复] 解决provider冲突问题； 

V 0.4.8
 * [修复] 解决demo中直接呼起相机并裁剪不会返回数据的bug，不需要这个功能的可以不更新;
 
V 0.4.7
 * [新增] 新增可直接调起相机的功能;
 * [修复] 解决可能和主项目provider冲突的潜在问题；
 * [修复] 点击图片预览空指针崩溃问题；
 * [修复] 使用Intent传值限制导致的崩溃问题;
 * [修复] 部分机型拍照后图片旋转问题；
 * [修复] 更改选择框图片背景为灰色，以免白色图看不清。
 
V 0.3.5
 * [新增] 提供直接调起相机的方式，并可直接设置牌照是否裁剪；
 * [修复] Android7.0设备调系统相机直接崩溃的问题；
 * [注意] 如果出现 java.lang.RuntimeException: Unable to get provider android.support.v4.content.FileProvider: java.lang.SecurityException: Provider must not be exported，请直接clean再运行即可。

## Licenses
```
 Copyright 2016 jeasonlzy(廖子尧)，2019 CysionLiu

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
```

