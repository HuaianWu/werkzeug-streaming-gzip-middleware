我们想让 JSON、HTML 和静态文本响应按客户端 Accept-Encoding 自动使用 gzip，大响应还要保持流式输出。协商必须处理质量值、通配符和 identity；已压缩内容、不可压缩响应、HEAD、204、304 都不能误处理，同时要正确维护 Content-Length、ETag、Vary，并在客户端提前断开时清理响应迭代器，帮我完成这个功能的新增
