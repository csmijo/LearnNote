# Appium 笔记

`Appium`

---

## 1. Appium Android平台支持

* **版本**：2.3 及以上版本
    * 2.3 至 4.2 版本是通过 Appium 绑定的基于 Instrumentation框架的 Selendroid实现的自动化。Selendroid 的命令设置与默认的 Appium 有点不同， 支持的配置文件也同样不同。要获得在后台运行自动化的权限，需要环境配置中将 automationName 的值为 Selendroid。
    * 4.2 以及更高的版本是通过 Appium 自己的 UiAutomator 库实现。这是默认的自动化后台。
* **设备**：Android 模拟器以及 Android 真机
* **是否支持 Native 应用**：支持
* **是否支持移动端浏览器**：支持（除了使用 `Selendroid` 后台的时候）。Appium 绑定了一个 `Chromedriver` 服务，使用这个代理服务进行自动化测试。
    * 在 `4.2` 和 `4.3` 版本，只能在官方的 `Chrome` 浏览器或者 `Chromium` 执行自动化测试。
    * 在 `4.4` 及更高版本，可以在内置的 “浏览器” 应用上进行自动化了。更多介绍请查看 [mobile web doc](https://github.com/appium/appium/blob/master/docs/cn/writing-running-appium/mobile-web.md)。
* **是否支持 Hybrid 应用**：支持。更多介绍请查阅 hybrid doc。
    * 默认的 Appium 自动化后台：支持 4.4 以及更高版本
    * Selendroid 自动化后台：支持 2.3 以及更高版本
* **是否支持多个 app 在同一个 session 中自动化**：支持（除了使用 Selendroid 后台的时候）
* **是否支持多个设备同时进行自动化**：支持，即使 Appium 在开始的时候，使用不同的端口号作为服务器参数，`--port`, `--bootstrap-port` (或者 `--selendroid-port`) 还有 `--chromedriver-port`. 更多介绍请查看 [server args doc](https://github.com/appium/appium/blob/master/docs/cn/writing-running-appium/server-args.md)。
* **支持第三方引用**：支持（除了使用 Selendroid 后台的时候）
* **支持自定义的、非标准的 UI 控件的自动化**：**不支持**

## Appium Girls 学习指南





---

[参考文献]
* [Appium 支持的平台](https://github.com/appium/appium/blob/master/docs/cn/appium-setup/platform-support.md)
* [Appium Girls 学习指南](https://anikikun.gitbooks.io/appium-girls-tutorial/content/index.html)