#### 语法

```python
assert_exists # 断言图片存在
assert_not_exists # 断言图片不存在
assert_equal # 断言相等
assert_not_equal # 断言不等

# 例如 模拟输入并获得20分之后
value = Poco("分数按钮").attr("num")
assert_equal(value, 20, "获到20分")

# 在Python脚本中，airtestIDE中会自动将Template(xxxx)渲染为图片形式

# 脚本运行命令行有两种形式，命令行中的参数包含device、log等：
# 命令行运行Airtest脚本的示例：>airtest run untitled.air --device Android:///手机设备号 --log log/。

# 在很多接口中，支持传入Template图片对象作为参数，在运行时将会去点击图片在画面中的所在位置，类似这样：
# 等价于 touch((x, y)), (x, y)是图片所在的中心点
touch(Template(r"tpl1556019871196.png", record_pos=(0.204, -0.153), resolution=(1280, 720)))

# Template对象是一个图片类，Airtest会先尝试在当前画面中寻找能够匹配这张图片的位置，如果找到了，将对这个坐标进行点击操作，如果找不到，将抛出识别异常。我们将在后文对Template图片类进行更加详细的介绍。

# 在Android中，有一个平台独有的接口list_app可以列出所有安装过的应用
dev = device()  # 先获取到当前设备对象，这里的dev即是一个Android对象
print(dev.list_app())  # 然后就可以调用平台独有接口了

# Android平台下的touch接口支持额外的参数duration来控制点击屏幕的时长
# 翻阅airtest.core.android.android中的Android包含的touch方法来获取更多参数信息
touch((600, 500), duration=1)

```

#### Airtest常见窗口

```
touch ：点击某个位置，可以设定被点击的位置、次数、按住时长等参数
swipe ：从一个位置滑动到另外一个位置
text ：调用输入法输入指定内容
keyevent ：输入某个按键响应，例如回车键、删除键
wait ：等待某个指定的图片元素出现
snapshot ：对当前画面截一张图
```



#### 简单实例

```python
# 初始化 -- 将Airtest的主要API都 import 进来
from airtest.core.api import *

# 是一个用来初始化环境的接口
# 5个参数，脚本所在的路径、指定运行脚本的设备、设置默认的log路径、设置脚本父路径、屏幕截图的压缩比率
# auto_setup不传入任何参数的话，即 auto_setup(__file__)）Airtest将会读取运行时命令行中传入的各项参数，来对环境进行初始化
# 意思是将脚本文件作为脚本路径传入，其他参数内容将默认读取运行命令行传入的参数。
# 请尽量在脚本初始化期间调用auto_setup接口，这样能保证尽可能正确地初始化环境、并生成log文件，否则默认是不生成log内容的
# 如果没有在初始化时连上设备，可以在auto_setup(__file__,devices=["Android://127.0.0.1:5037/5PZTQWQOGES8RWUG"])接口中指定运行脚本的设备，或者使用connect_device接口来连接设备。
auto_setup(__file__)



```



