# NEU 校园网自动连接

连接已保存的 NEU Wi-Fi，并打开校园网认证页面。

- `NEU_connect.zip`：Python 源码。如果电脑已经有 Python 环境，可以优先使用这个版本。需要安装 Playwright 和 Microsoft Edge，具体步骤见压缩包里的 README。
- `NEUConnect.zip`：免 Python 版，解压后运行，无需另外安装 Python 或 Playwright，但仍需要 Microsoft Edge。

两个版本的连接和认证逻辑相同，账号填写方式有所不同：源码版在 `neu_connect.py` 中填写；免 Python 版在 `config.json` 中填写。压缩包内提供的是空白账号配置，请不要分享填有自己密码的文件。

如果页面需要验证码或其他验证，请手动完成。页面显示“网络已连接”才表示认证成功。
