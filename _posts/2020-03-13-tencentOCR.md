#  使用腾讯云提供的OCR接口实现从截屏到文本的实现

效果演示：

![](/images/2020_03_13/批注 2020-03-12 105106.jpg"一张来自百度文库的截图")

!["识别后生成的txt文本"](/images/2020_03_13/2020-03-13 125743.jpg)

应用场景：百度文库 PDF扫描件 的文字复制

环境要求：python3.6+

原材料：一张图片文件（使用PC自带的截屏功能保存到本地即可）

## Step 1：开通腾讯云文字识别服务

打开腾讯云官网[](https://cloud.tencent.com) 

使用QQ或者微信登录 然后按照提示完成个人实名认证（仅需两步 十分简单）

点击页面左侧菜单栏 找到"免费产品" 在云产品体验栏目下 找到"人工智能"

![](/images/2020_03_13/2020-03-13 095328.jpg)

看到那个"立即体验"了吗 冲之（貌似现在的体验优惠价是10元即可开通一年的资源包使用"每月1000次限额"）

按照提示信息继续操作即可完成第一步

## Step 2：在python中安装腾讯云sdk

打开Windows命令行(快捷键: win+r 键入cmd后回车) 

输入 

```
pip3 install tencentcloud-sdk-python
```

稍等片刻 提示安装成功即可

## Step 3：获取API访问密钥

打开 腾讯云控制台[](https://console.cloud.tencent.com/cam/capi)

![](/images/2020_03_13/2020-03-13 105851.jpg)

点击新建密钥 将密钥保存到剪切板中（后面会用到）

## Step 4：生成接口代码

使用官方工具生成接口代码 [](https://console.cloud.tencent.com/api/explorer?Product=ocr&Version=2018-11-19&Action=GeneralBasicOCR）

如果上述链接失效 请打开 [](https://cloud.tencent.com/document/product/866/33526)

找到

![](/images/2020_03_13/2020-03-13 111237.jpg)

点击 API 3.0 Explorer

![](/images/2020_03_13/2020-03-13 111540.jpg)

在中间区域填入准备好的个人密钥

在输入参数中 选择所在区域（注意：下面的选填项先不填）

然后点击右侧的代码生成 选择Python语言 将自动生成的代码复制下来

## Step 5：在模板代码的基础上作修改

现在的代码状况呢 应该是这样的

![](/images/2020_03_13/2020-03-13 113433.jpg)

官方代码生成工具中的图片方案有两种 一种是提供图片的base64编码信息 另一种是提供图片的URL链接

显然我不准备把我的本地截屏上传到网上（当然 上传到github上也不失为一个稳妥的选择）

那么我就需要获得我本地图片的base64编码 实现起来呢也是非常简单

```python
import base64

f = open(file_path,'rb')
base64_data = base64.b64encode(f.read())
ls_f = str(base64_data, 'utf-8')
# print(ls_f)
f.close
```

这样我们就得到了我们需要的编码（字符串类型）

不难发现 正常截屏得到的图片经过编码后内容是很庞大的 如果使用官方工具提供的载入方式 你的代码会极其丑陋 
并且会产生信息缺损

![](/images/2020_03_13/2020-03-13 115218.jpg)

为了解决这个问题 同时也是为后续的自动化载入多张图片提供便利 我们需要初步了解载入的过程 
上面三行代码的含义呢就是req通过调用from_json_string()方法将我们的base64编码传入req中
而这个步骤的本质呢还是对req中的某个变量进行赋值操作（将我们的编码赋值给req中的ImageBase64成员变量）

因此我们不妨直接执行赋值操作 将上图的三行代码改成：

```python
req = models.GeneralBasicOCRRequest()
req.ImageBase64 = ls_f //ls_f即为encode后的字符串编码
```

运行一下 看一下效果（有很多我们不需要的信息，而且内容的可读性很差）

![](/images/2020_03_13/2020-03-13 121251.jpg)

为了方便我们查看识别出来的效果 我们将结果保存到本地的一个txt文本中去
需要在代码最后加上

```python
with open('./for_test.txt','w',encoding='utf-8') as f:
    f.write(resp.to_json_string(indent=2))  //indent参数默认为None 此时输出的结果是紧凑保存 效果如之前截图所示
                          //将indent置为2后 可以得到可读性更高的自动缩进版本（你也可以设置其它数值 来修改缩进量）
```

运行之后打开txt文本

![](/images/2020_03_13/2020-03-13 122321.jpg)

ok 他看上去能给人看了 不过里面的多余信息属实鸡肋（其实也是很重要的信息 但是就txt文本来说 完全用不到） 那么就需要我们将这些内容去除

不难发现 我们保存的内容是一个Json字符串（不了解也没有关系 可以理解为一个字符串数组之类的）

总而言之 我们可以通过关键字索引 来获取我们需要的信息

```python
import json
print(json.loads(Json字符串)["关键字1"]["关键字2"]...)
```

得到这样的原理之后呢 我们再根据我们的数据 来选择适当的关键字（通过txt文本中的内容我们可以发现 它的结构为）

```
{
  "TextDetections":[
  {
    "DetectedText":
    ...
   }
  {...}
  {...}
  ...
  ]
  "Language": 
  "RequestId": 
}
```

可以看到 我们需要的的文本信息被分割成了很多段 因此 我们需要借助一个循环体来把我们需要的信息连接起来

```python
TextDetections = json.loads(resp)["TextDetections"]
DetectedText = ""
for i in range(len(TextDetections)):
    DetectedText+=TextDetections[i]["DetectedText"]+'\n'
```

最终呢 我们的完整代码就变成了这样：

```python
from tencentcloud.common import credential
from tencentcloud.common.profile.client_profile import ClientProfile
from tencentcloud.common.profile.http_profile import HttpProfile
from tencentcloud.common.exception.tencent_cloud_sdk_exception import TencentCloudSDKException 
from tencentcloud.ocr.v20181119 import ocr_client, models 
import base64
import json

file_path = r"C:\Users\..."


f = open(file_path,'rb')
base64_data = base64.b64encode(f.read())
ls_f = str(base64_data, 'utf-8')
# print(ls_f)
f.close

try: 
    cred = credential.Credential("***", "***") 
    httpProfile = HttpProfile()
    httpProfile.endpoint = "ocr.tencentcloudapi.com"

    clientProfile = ClientProfile()
    clientProfile.httpProfile = httpProfile
    client = ocr_client.OcrClient(cred, "ap-shanghai", clientProfile) 

    req = models.GeneralBasicOCRRequest()
    req.ImageBase64 = ls_f 

    # resp = client.GeneralBasicOCR(req)
    # # print(resp.to_json_string())
    
    # with open('./for_test.txt','w',encoding='utf-8') as f:
    #     f.write(resp.to_json_string(indent=2))

    resp = client.GeneralBasicOCR(req).to_json_string()

    TextDetections = json.loads(resp)["TextDetections"]
    DetectedText = ""
    for i in range(len(TextDetections)):
        DetectedText+=TextDetections[i]["DetectedText"]+'\n'

    # print(DetectedText)

    with open('./for_test.txt','w',encoding='utf-8') as f:
        f.write(DetectedText)

except TencentCloudSDKException as err: 
    print(err) 
```

如果对我的内容有任何问题或者有更好的解决方案的话 欢迎评论或私信哦
