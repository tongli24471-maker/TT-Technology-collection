小白入门 WebUI 自动化：框架搭建（二）｜我踩过的坑都标出来了

小白说明

主包真真以为面试造航母进来拧螺丝，面试八股文答的飞起，实际上手废，Python停留在基本知识，真写代码……难评（hello world？）但也无所谓啦，毕竟还是点点点嘛
结果！！！来了新领导，上任三把火啊，要搞AI要搞自动化，对标大厂（大哥，你真看的起我们= =），每人安排任务（也就3测试= =），主包被分到搭建webUI自动化框架……

算了，废话不多说，真勇士就是实操干起来
前一篇已经告诉大家如何进行进行环境配置了，那么我们可以开始进入正篇了

踩坑 Web UI 自动化测试框架搭建结构



毕竟现在AI时代，我们也得跟上是不，嘻嘻嘻

以下是AI给出的结构，参考参考~

web_ui_autotest/
├── config/                  # 配置文件目录
│   ├── config.yaml          # 全局配置（环境、超时时间等）
│   ├── locators/            # 元素定位表达式目录
│   │   ├── login_page.yaml
│   │   └── home_page.yaml
│   └── test_data/           # 测试数据目录
│       ├── login_data.csv
│       └── user_info.json
│
├── core/                    # 框架核心模块
│   ├── base_page.py         # 页面对象基类
│   ├── driver.py            # 浏览器驱动管理
│   ├── logger.py            # 日志处理
│   ├── exception.py         # 自定义异常
│   └── utils/               # 工具函数
│       ├── file_utils.py    # 文件处理
│       ├── time_utils.py    # 时间处理
│       └── screenshot.py    # 截图功能
│
├── pages/                   # 页面对象目录
│   ├── __init__.py
│   ├── login_page.py        # 登录页面
│   ├── home_page.py         # 首页
│   └── user_page.py         # 用户页面
│
├── testcases/               # 测试用例目录
│   ├── __init__.py
│   ├── conftest.py          # pytest配置
│   ├── test_login.py        # 登录相关测试
│   └── test_user.py         # 用户相关测试
│
├── reports/                 # 测试报告目录
│   ├── html/                # HTML报告
│   └── logs/                # 测试日志
│
├── requirements.txt         # 依赖包列表
├── run.py                   # 执行入口
└── README.md                # 框架说明文档
虽然没有完全按照这个方式，但主包还是基本按照这种结构把文件都创建好了。



踩坑 AI干活



主包心里想：既然开了个 AI 头，那就讲究个有始有终，继续用到底！

于是就兴致勃勃地让 AI 来帮忙：
“来，把浏览器封装一下吧~ 再把环境封装一下吧~ 顺手再搞个登录页吧~”

当时感觉操作一气呵成，稳得不行，心想这下真 nice。

结果现实啪啪打脸：
一是 AI 封装的方法让主包看得一头雾水；
二是主包想老老实实跑 case 的时候，报错像机关枪一样噼里啪啦冒出来。


成长 撸起袖子自己干

Python只入门、实操为零。AI甩来一坨“高阶写法”，看不懂也不敢改；想回炉补课？时间不够，还未必记得住。需求却在旁边敲碗：快点交付！
那咋办？只能撸起袖子自己干！

策略很简单：小步快跑，先把一条最小可运行用例从头写到尾，能跑通就是胜利，其它优化后面再说。

开干步骤就三个字：写清单
把要做的动作先写成 TODO，边写边勾，思路稳、心态也稳：



##  选择浏览器，Chrome、firefox，
## TODO 后续可以改为继承，选择异步还是同步，后续分开
## TODO 跳转登录页
## TODO 登录页面元素以及登录账号密码输入
## TODO 断言页面上的元素
按照这个思路慢慢往里面填东西



成长 初版

主包小脑袋瓜子+AI+playwright的官网文档，总算写出一个初版

import logging
import playwright
import pytest
from playwright.sync_api import sync_playwright, Playwright

##  选择浏览器，Chrome、firefox，
# TODO 后续可以改为继承，选择异步还是同步，后续分开
defchoosebrowser(playwright: Playwright):
browser = playwright.chromium.launch(
headless=False,  # 设置为 False 可以看到浏览器界面，True 则无头模式运行
slow_mo=500      # 每个操作延迟 500ms，方便观察
)
## TODO 跳转登录页
# 创建新页面
context = browser.new_context()
page = context.new_page()
page = browser.new_page()
page.goto("https://xxxx.com/login")
logging.log(page.title())

    ## TODO 登录页面元素以及登录账号密码输入
    page.get_by_role("textbox", name="用户名").click()
    page.get_by_role("textbox", name="用户名").fill("xxx")
    page.get_by_role("textbox", name="密码").click()
    page.get_by_role("textbox", name="密码").fill("1234")
    #滑块验证
    slider=page.locator("nc_iconfont btn_slide").bounding_box()
    # 拖动滑块到最右端
    page.mouse.move(slider['x'] + slider['width']/2, slider['y'] + slider['height']/2)
    page.mouse.down()
    page.mouse.move(slider['x'] + slider['width']/2 + 260, slider['y'] + slider['height']/2)
    # 发现滑动成功元素后再进行下一步
    try:
        page.get_by_role("class",name="nc_iconfont btn_ok").wait_for(state="visible", timeout=10_000)
    except TimeoutError:
        raise RuntimeError("未检测到关键元素，终止脚本！")
    page.get_by_role("button",name="登录").click()
    # 登录按钮
    page.get_by_role("button", name="登录").click()
## TODO 断言页面上的元素
可惜浏览器压本没起来……

别急~让我问问AI啥原因

原因总结
浏览器根本没起来的根本原因是：choosebrowser() 函数里虽然写了启动代码，但谁都没调用它。
加上你在 pytest 框架里运行，pytest 只会收集并执行以 test_ 开头的函数；你当前的函数叫 choosebrowser，既没加 @pytest.mark.* 也没被任何 test_* 调用，于是整个文件里没有任何可执行用例，pytest 直接返回 0 项测试，浏览器当然不会弹出来。

此外还有一些隐藏 BUG，先一次性列给你：

函数未被执行（主因）

多余的一行 page = browser.new_page() 导致前一个 page 被覆盖

logging.log() 用法错误

滑块选择器少了点号，应为 .nc_iconfont.btn_slide

get_by_role("class", …) 写法不存在，正确的是 locator(".nc_iconfont.btn_ok")

嚯！好家伙，都没调用，写了个寂寞，看出来上手废了~



让AI调了调，优化成以下代码

import logging
import pytest
from playwright.sync_api import sync_playwright

@pytest.mark.login
deftest_login_slider():
with sync_playwright() as p:
# 启动浏览器
browser = p.chromium.launch(headless=False, slow_mo=500)
context = browser.new_context()
page = context.new_page()
page.goto("https://xxxx.com/login")
logging.info(page.title())

        # 登录
        page.get_by_role("textbox", name="用户名").fill("xxx")
        page.get_by_role("textbox", name="密码").fill("1234")

        # 滑块
        slider = page.locator(".nc_iconfont.btn_slide").bounding_box()
        page.mouse.move(slider["x"] + slider["width"] / 2,
                        slider["y"] + slider["height"] / 2)
        page.mouse.down()
        page.mouse.move(slider["x"] + slider["width"] / 2 + 260,
                        slider["y"] + slider["height"] / 2)
        page.mouse.up()

        # 等待滑块成功
        try:
            page.locator(".nc_iconfont.btn_ok").wait_for(state="visible", timeout=10_000)
        except TimeoutError:
            raise RuntimeError("未检测到关键元素，终止脚本！")

        # 登录按钮
        page.get_by_role("button", name="登录").click()
        page.wait_for_timeout(5000)
        # TODO：断言
        # assert page.url.endswith("/dashboard")

        context.close()
        browser.close()
嗯，这下应该成了吧，结果又报找不到元素问题



问题

写法错误

截取问题代码

# 等待滑块成功
try:
page.locator("nc_iconfont btn_ok")
except TimeoutError:
raise RuntimeError("未检测到关键元素，终止脚本！")
本来是想等待class=“nc_iconfont btn_ok”这个元素出来才能进行下一步操作，但是一直找不到这个元素。

问了问AI，原来这个元素写法写错了，看看AI咋说的

记住一条：class 名里带空格表示同时拥有多个 class，定位时把空格换成点号 . 即可。

page.locator(".nc_iconfont.btn_ok").wait_for(state="visible", timeout=10_000)
常见错误

错误示例

原因

page.locator("nc_iconfont btn_ok")

少了点号，被当成标签名

page.locator(".nc_iconfont .btn_ok")

中间多了空格，变成“父子层级



一句话：把空格换成点号，前面统一加 .



踩坑

文字匹配



改好后，主包兴冲冲的执行脚本

BUT！！！又报找不到元素了

我左检查右检查，真没发现啥问题

我再与网页对比一下，你猜怎么着

网页按钮文字中间多个空格！！！

图片


原代码

# 登录按钮
page.get_by_role("button", name="登录").click()
网页登录按钮

图片
主包内心一万个动物跑过……

绝对要注意这些细节，不然真的耽误很多时间



好了，能启动了，我们的框架正式踏出了第一步！



下集再见哟~



图片
图片
点/击/蓝/字  关/注/我/们

