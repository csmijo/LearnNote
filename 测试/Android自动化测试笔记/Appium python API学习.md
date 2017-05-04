# Appium python API 学习

`Appium` `python API`

--

## 0. Appium server arguments

* [Appium server arguments](https://appium.io/slate/en/master/?ruby#appium-server-capabilities)
* [学习 DesiredCapabilities](https://anikikun.gitbooks.io/appium-girls-tutorial/content/desired_caps.html)

## 1. find element

### 1.1 driver 的方法

　　**注意：** 这几个方法只能通过 `self.driver` 调用
　　

1. `find_element_by_android_uiautomator(self,uia_string)`
2. `find_elements_by_android_uiautomator(self,uia_string)`
3. `find_element_by_accessibility_id(self,id)`
4. `find_elements_by_accessibility_id(self,id)`

### 1.2 element 的方法 

　　**注意：** 这几个方法通过 `driver`或`element`都可以调用。`element` 调用时可以用来定位该元素的子元素

1.  `find_element_by_id(self,id_)`
2. `find_elements_by_id(self,id_)`
3. ~~`find_element_by_name(self,name)`~~   **Appium 1.5版本之后废弃**
4. ~~`find_elements_by_name(self,name)`~~  **Appium 1.5版本之后废弃**
5. `find_element_by_link_text(self,link_text)`
6. `find_elements_by_link_text(self,link_text)`
* `find_element_by_partial_link_text(self, link_text)`
* `find_elements_by_partial_link_text(self, link_text)`
* `find_element_by_tag_name(self, name)`
* `find_elements_by_tag_name(self, name)`
* `find_element_by_xpath(self, xpath)`   **通过Xpath定位元素，详细方法可参阅`http://www.w3school.com.cn/xpath/`**
* `find_elements_by_xpath(self, xpath)`
* `find_element_by_class_name(self, name)`
* `find_elements_by_class_name(self, name)`


## 2. remote/webdriver

　　**注意：**　`selenium` 原生方法，调用方法：`self.driver.xxxx()`

1. `page_source(self)`
* `get_screenshot_as_file(self, filename)`
* `get_window_size(self, windowHandle='current')`
* `orientation(self)`   获取方向
* `orientation(self, value)`  设置方向
* `desired_capabilities(self)`   返回 设定的运行参数

## 3. appium/webdriver

此类方法的调用方式均为 `self.driver.xxxx()`

### 3.1 输入操作

#### 3.1.1 点击/触摸操作

1. `scroll(self, origin_el, destination_el)`
* `drag_and_drop(self, origin_el, destination_el)`
* `tap(self, positions, duration=None)`  模拟手指点击（最多五个手指），可设置按住时间长度（毫秒）
* `swipe(self, start_x, start_y, end_x, end_y, duration=None)`    从A点滑动至B点，滑动时间为毫秒
* `flick(self, start_x, start_y, end_x, end_y)`  按住A点后快速滑动至B点
* `pinch(self, element=None, percent=200, steps=50)`   在元素上执行模拟双指捏（缩小操作）
* `zoom(self, element=None, percent=200, steps=50)`   在元素上执行放大操作，双指放大
* `open_notifications(self)`   打系统通知栏（**仅支持API 18 以上的安卓系统**）
* `shake(self)`  摇一摇手机

#### 3.1.2 按键 

1. `press_keycode(self, keycode, metastate=None)`   发送按键码（安卓仅有），按键码可以上网址中找到`http://developer.android.com/reference/android/view/KeyEvent.html`
* `long_press_keycode(self, keycode, metastate=None)`

#### 3.1.3 输入法


1. `hide_keyboard(self, key_name=None, key=None, strategy=None)`   隐藏键盘,iOS使用key_name隐藏，安卓不使用参数

* `available_ime_engines(self)`   **Android only**. 返回安卓设备可用的输入法
* `is_ime_active(self)`  **Android only**. 检查设备是否有输入法服务活动。返回真/假
* `activate_ime_engine(self, engine)`   **Android only**.激活安卓设备中的指定输入法，设备可用输入法可以从“available_ime_engines”获取
* `deactivate_ime_engine(self)`    **Android only**.关闭安卓设备当前的输入法
* `active_ime_engine(self)`  **Android only**. 返回当前输入法的包名

### 3.2 app控制
1.  `reset(self)`  重置应用(相当于卸载重装应用)
* `background_app(self, seconds)`  后台运行app多少秒
* `is_app_installed(self, bundle_id)`  检查app是否有安装
* `install_app(self, app_path)`   安装app,app_path为安装包路径
* `remove_app(self, app_id)`
* `launch_app(self)`        根据服务关键字 (`desired capabilities`) 启动会话 (`session`) 。**请注意**这必须在设定 `autoLaunch=false` 关键字时才能生效。这不是用于启动指定的 `app/activities` ————你可以使用 `start_activity` 做到这个效果————这是用来继续进行使用了 `autoLaunch=false` 关键字时的初始化 (`Launch`) 流程的。
 
* `close_app(self)`


### 3.3 contexts/activity

1. `contexts(self)`  返回当前会话中的上下文，使用后可以识别H5页面的控件
* `current_context(self)`  返回当前会话的当前上下文
* `context(self)`  返回当前会话的当前上下文。
* `current_activity(self)`   获取当前的activity
* `wait_activity(self, activity, timeout, interval=1)`  **Android only**.  等待指定的`activity` 出现直到超时，`interval` 为扫描间隔1秒
即每隔几秒获取一次当前的 `activity` 
* `start_activity(self, app_package, app_activity, **opts)`  **Android only**.  在测试过程中打开任意活动。如果活动属于另一个应用程序，该应用程序的启动和活动被打开  

### 3.4 network

 
1. `network_connection(self)`  **Android only**.  返回网络类型  数值
* `set_network_connection(self, connectionType)`  **Android only**. 

    |Value (Alias)      | Data | Wifi | Airplane Mode|
    |:--|:--|:--|:--|
    |0 (None)           | 0    | 0    | 0|
    |1 (Airplane Mode)  | 0    | 0    | 1|
    |2 (Wifi only)      | 0    | 1    | 0|
    |4 (Data only)      | 1    | 0    | 0|
    |6 (All network on) | 1    | 1    | 0|

### 3.5 位置服务

 
1. `toggle_location_services(self)`  打开安卓设备上的位置定位设置
* `set_location(self, latitude, longitude, altitude)`   设置设备的经纬度

### 3.6 文件 pull/push
 
1. `pull_file(self, path)`  从设备的指定路径获取文件
* `pull_folder(self, path)`
* `push_file(self, path, base64data)`

### 3.7 其他

1. `end_test_coverage(self, intent, path)`  **Android only**.测试覆盖率相关，目前不能提供完整支持。 `https://github.com/appium/appium/blob/master/docs/en/android_coverage.md`
* `set_value(self, element, value)`   设置某元素的值
* `create_web_element(self, element_id)`
* `app_strings(self, language=None, string_file=None)`  解析被测app的strings.xml

## 4. remote/webelement

　　页面元素的方法，调用方法：`element.xxxx()`
 
1. `tag_name(self)`   返回元素的tagName属性
* `text(self)`  返回元素的文本值
* `click(self)` 
* `submit(self)`   提交表单
* `clear(self)`  清除输入的内容
* `get_attribute(self, name)`
* `is_selected(self)`
* `is_enabled(self)`
* `send_keys(self, *value)`  在元素中模拟输入（开启appium自带的输入法并配置了appium输入法后，可以输入中英文）
* `size(self)`  获取元素的大小（高和宽）
* `location_once_scrolled_into_view(self)`  **不稳定，不推荐使用**
* `location(self)`   获取元素左上角的坐标
* `rect(self)`  元素的大小和位置的字典


--
[参考文献]

* [Appium python API 总结](https://testerhome.com/topics/3711)
* [Appium Python API 中文版 By-HZJ](https://testerhome.com/topics/3711)
* [Appium 客户端库](http://appium.readthedocs.org/en/stable/cn/writing-running-appium/appium-bindings.cn/)