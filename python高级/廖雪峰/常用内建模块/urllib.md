# urllib：HTTP 请求与 URL

> **重要程度：中**｜**手练：建议**
> 重点：URL 编码、请求超时、异常与响应关闭。

## 模块分工

- `urllib.request`：构造请求和读取响应。
- `urllib.parse`：拆分 URL、编码查询参数。
- `urllib.error`：处理 `HTTPError` 和 `URLError`。

`urlopen()` 返回响应对象，可用 `with` 管理关闭；`read()` 返回字节。文本应根据协议或响应声明选择编码。默认请求方式一般由是否提供 `data` 决定；`Request` 也可显式指定方法和请求头。

## 工作中注意

设置连接超时、限制响应大小，并处理 HTTP 错误、网络错误及超时；超时等异常不一定都包装成 `URLError`。`HTTPError` 是 `URLError` 的子类，分别处理时先捕获前者。重试前先判断请求是否安全或具备幂等保障。

`urlencode()` 适合编码查询字符串或表单数据，不等于 JSON 编码。不要在 URL、日志或示例代码中写入真实密码。访问外部地址时检查协议、重定向和目标地址，防止服务端请求伪造。

手练：请求本地测试服务，设置超时；分别验证正常响应、404 和超时，并确认响应已关闭。

参考：[urllib.request](https://docs.python.org/3/library/urllib.request.html)；[urllib.error](https://docs.python.org/3/library/urllib.error.html)。
