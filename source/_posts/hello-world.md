---
title: Hello World
categories:
- 技术
tags:
- 工具
- 示例
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>demo</title>
</head>
<body>
    <button type="button" id="alertBtn">Alert</button>
    <script type="text/javascript" src="alert.js"></script>
    <script type="text/javascript">
        document.getElementById('alertBtn').onclick = function(){
            //调用 Chef.alert() 方法
            Chef.alert({
                'title':'标题',
                'content':'内容',
                'firm':'确定',
                'width':'300px',
                'shade':0.4
            });
        };
    </script>
</body>
</html>

```
