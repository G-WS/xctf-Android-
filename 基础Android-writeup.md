# 基础Android-writeup

首先将该软件放入雷电模拟器中，发现需要输入密码；

使用jadx打开该文件，查看AndroidManifest.xml,可以得到如下的信息：

![image-20230220170405581](C:\Users\33551\AppData\Roaming\Typora\typora-user-images\image-20230220170405581.png)

从中我们可以发现该APK拥有如下的Activity：MainActivity、MainActivity2、NextContent；还有一个广播为GetAndChange。

## MainActivity

首先进入主活动，阅读代码逻辑，代码逻辑如下：

- 对密码进行验证，如果正确则进入MainActivity，不正确则弹出弹窗false
- 判断密码是否正确的类为Check类

![image-20230220170959086](C:\Users\33551\AppData\Roaming\Typora\typora-user-images\image-20230220170959086.png)

在这里有两种处理方法：

- 一种是直接通过AndroidKiller对AndroidManifest.xml文件中的入口活动进行修改，修改后重新进行编译和打包签名，我们就可以直接进入MainActivity2，属于是一种逃课。

- 第二种方法为，进入Check类，Check类的代码逻辑如下：

  -  该代码逻辑需要我们的密码长度为12，且输入的密码经过循环语句的计算之后，每一位都变成`'0'`,通过计算后，我们发现密码为如下

    ```java
    kjihgfedcba`
    ```

    ```java
    public class Check {
        public boolean checkPassword(String str) {
            char[] pass = str.toCharArray();
            if (pass.length != 12) {
                return false;
            }
            for (int len = 0; len < pass.length; len++) {
                pass[len] = (char) (((255 - len) - 100) - pass[len]);
                if (pass[len] != '0' || len >= 12) {
                    return false;
                }
            }
            return true;
        }
    }
    
    ```

## MainActivity2

接下来我们对于第二关进行分析，阅读源码可以发现

![image-20230220171648011](C:\Users\33551\AppData\Roaming\Typora\typora-user-images\image-20230220171648011.png)

这一关发送了一个广播，回顾之前得到的广播信息，我们转到GetAndChange类中进行查看

```java
public void onReceive(Context context, Intent intent) {
        Intent intent1 = new Intent(context, NextContent.class);
        context.startActivity(intent1);
}
```

可以看到，该广播直接过渡到了NextContent活动中，进入该活动查看源码：

可以看到该活动中主要包括两个方法，重点的方法在Change方法：

```java
public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_next_content);
        init();
        Change();
    }
```

分析Change逻辑：

```java

    public void Change() {
        String strFile = getApplicationContext().getDatabasePath("img.jpg").getAbsolutePath();
        try {
            File f = new File(strFile);
            if (f.exists()) {
                f.delete();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        try {
            InputStream is = getApplicationContext().getResources().getAssets().open("timg_2.zip");
            FileOutputStream fos = new FileOutputStream(strFile);
            byte[] buffer = new byte[1024];
            while (true) {
                int count = is.read(buffer);
                if (count <= 0) {
                    break;
                }
                fos.write(buffer, 0, count);
            }
            fos.flush();
            fos.close();
            is.close();
        } catch (Exception e2) {
            e2.printStackTrace();
        }
        this.imageView.setImageBitmap(BitmapFactory.decodeFile(strFile));
    }

```

可以发现该函数中出现了两个关键文件，一个是img.jpg,另一个为timg_2.zip，函数的逻辑就是将压缩未见的扩展名进行修改，转为一个jpg文件格式，我们直接将apk后缀名修改为zip，解压后在assets中寻找到该压缩文件，然后将该压缩文件的后缀名修改正jpg，即可获得带有flag的图片。

