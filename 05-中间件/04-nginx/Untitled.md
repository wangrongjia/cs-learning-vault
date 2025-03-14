## ^~的作用：an

- **location ^~ /report/**：优先级更高，更安全，适合需要确保前缀匹配不被干扰的场景。
- **location /report/**：简单前缀匹配，适用于没有正则表达式竞争的简单配置。

```nginx
location ^~ /report/ {
    alias /usr/share/nginx/html/fsscreport/;
    try_files $uri $uri/ /index.html;
}
```
