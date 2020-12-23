---
title: 开发 - Vagrant
sort: 11
contributors:
  - SpaceK33z
  - chrisVillanueva
  - byzyk
  - wizardofhogwarts
---

如果你在开发一个更加高级的项目，并且使用 [Vagrant](https://www.vagrantup.com/) 来实现在虚拟机(Virtual Machine)上运行你的开发环境，那你可能会需要在虚拟机中运行 webpack。

## 配置项目 {#configuring-the-project}

首先，确保 `Vagrantfile` 拥有一个静态 IP。

```ruby
Vagrant.configure("2") do |config|
  config.vm.network :private_network, ip: "10.10.10.61"
end
```

然后，在项目中安装 `webpack` 和 `webpack-dev-server`；

```bash
npm install --save-dev webpack webpack-dev-server
```

确保提供一个 `webpack.config.js` 配置文件。如果还没有准备，下面的示例代码可以作为起步的最简配置：

```js
module.exports = {
  context: __dirname,
  entry: './app.js',
};
```

然后，创建一个 `index.html` 文件。其中的 `script` 标签应当指向你的 bundle。如果没有在配置中指定 `output.filename`，其默认值是 `bundle.js`。

```html
<!doctype html>
<html>
  <head>
    <script src="/bundle.js" charset="utf-8"></script>
  </head>
  <body>
    <h2>Heey!</h2>
  </body>
</html>
```

注意，你还需要创建一个 `app.js` 文件。

## 启动服务器 {#running-the-server}

现在，我们启动服务器：

```bash
webpack serve --host 0.0.0.0 --public 10.10.10.61:8080 --watch-poll
```

默认只允许从 localhost 访问服务器。所以我们需要修改 `--host` 参数，
以允许在我们的宿主 PC 上访问。

`webpack-dev-server` 会在包中引入一个脚本，此脚本连接到 WebSocket，这样可以在任何文件修改时，触发重新加载应用程序。
`--public` 标记可以确保脚本知道从哪里查找 WebSocket。默认情况下，服务器会使用 `8080` 端口，因此也需要在这里指定。

`--watch-poll` 可以确保 webpack 能够检测到文件更改。webpack 默认会监听文件系统触发的相关事件，
但是 VirtualBox 使用默认配置会有许多问题。

现在服务器应该能够通过 `http://10.10.10.61:8080` 访问了。修改 `app.js`，应用程序就会实时重新加载。

## 配合 nginx 的高级用法 {#advanced-usage-with-nginx}

为了更好的模拟类生产环境（production-like environment），还可以用 nginx 来代理 `webpack-dev-server`。

在你的 nginx 配置文件中，加入下面代码：

```nginx
server {
  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    error_page 502 @start-webpack-dev-server;
  }

  location @start-webpack-dev-server {
    default_type text/plain;
    return 502 "Please start the webpack-dev-server first.";
  }
}
```

`proxy_set_header` 这几行配置很重要，因为它们能够使 WebSocket 正常运行。

然后 `webpack-dev-server` 启动命令可以修改为：

```bash
webpack serve --public 10.10.10.61 --watch-poll
```

现在只能通过 `127.0.0.1` 访问服务，这点关系不大，因为 ngnix 能够使得你的 PC 电脑能访问到服务器。

## 结论 {#conclusion}

我们能够从静态 IP 访问 Vagrant box，然后使 `webpack-dev-server` 可以公开访问，以便浏览器可以访问到它。然后，我们解决了 VirtualBox 不发送到文件系统事件的常见问题，此问题会导致服务器无法重新加载文件更改。
