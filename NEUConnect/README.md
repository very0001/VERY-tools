# NEU 校园网自动连接

连接已保存的 NEU Wi-Fi，并打开校园网认证页面。

- `NEU_connect.zip`：Python 源码。电脑已有 Python 环境的话，可以优先使用这个版本。需要安装 Playwright 和 Microsoft Edge，具体步骤见压缩包里的 README。
- `NEUConnect.zip`：免 Python 版，解压后运行，不需要另外安装 Python 或 Playwright，但仍需要 Microsoft Edge。

两个版本都在 `config.json` 中填写账号密码，运行时不会显示终端窗口。约 60 秒内没有连上 NEU 就会自动退出。需要验证码时请在网页上手动完成，页面显示“网络已连接”才表示认证成功。

压缩包里的账号密码都是空的，不要分享填有自己密码的文件。
