```shell
npm init
npm install http-server
http-server
#或package.json
"scripts": {
     "start": "http-server -a 0.0.0.0 -p 8000",
 }
```

- [path] defaults to ./public if the folder exists, and ./ otherwise.
- 配置代理

```javascript
http-server -p 3000 -P https://condejs.org
//-p 本地运行端口  -P 代理地址（就是要访问的接口下的域名）
```

**参数**

```
-p 端口号 (默认 8080)
-a IP 地址 (默认 0.0.0.0)
-d 显示目录列表 (默认 'True')
-i 显示 autoIndex (默认 'True')
-e or --ext 如果没有提供默认的文件扩展名(默认 'html')
-s or --silent 禁止日志信息输出
--cors 启用 CORS via the Access-Control-Allow-Origin header
-o 在开始服务后打开浏览器
-c 为 cache-control max-age header 设置Cache time(秒) , e.g. -c10 for 10 seconds (defaults to '3600'). 禁用 caching, 则使用 -c-1.
-U 或 --utc 使用UTC time 格式化log消息
-P or --proxy Proxies all requests which can't be resolved locally to the given url. e.g.: -P http://someurl.com
-S or --ssl 启用 https
-C or --cert ssl cert 文件路径 (default: cert.pem)
-K or --key Path to ssl key file (default: key.pem).
-r or --robots Provide a /robots.txt (whose content defaults to 'User-agent: *\nDisallow: /')
-h or --help 打印以上列表并退出
```

> https://www.npmjs.com/package/http-server
