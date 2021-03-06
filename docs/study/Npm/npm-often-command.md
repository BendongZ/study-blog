# npm常用命令

升级NodeJS:
```bash
$ npm i n -g
$ n stable
```

全局安装卸载package:
```bash
$ npm install forever --global (-g)
$ forever
$ npm uninstall forever --global
$ forever
```

本地安装package:
```bash
$ cd ~/desktop
$ mkdir gp-project
$ cd gp-project
$ npm install underscore
$ tree
$ npm list (ls)
```

安装指定版本的包：
```bash
$ pwd
$ npm list
$ npm info underscore
$ npm install underscore@1.8.0
$ npm list
$ npm uninstall underscore
$ npm list
```

package.json初始化：
```bash
$ pwd
$ npm init
$ ls
$ cat package.json
```

使用package.json:
```bash
$ npm install underscore --save
$ cat package.json
$ npm install babel-cli --save-dev
$ cat package.json
$ rm -rf node_modules
$ ls
$ npm install  使用package.json文件安装包
$ npm uninstall underscore --save
$ npm list |grep underscore
$ cat package.json
```

更新本地安装的包：
```bash
$ npm info gulp
$ npm install gulp@3.8.0 --save-dev
$ npm list | grep gulp
$ npm outdated
$ atom ./     ~2.0.0表示patch, ^2.0.0表示minor   * 表示xx最新版本
$ npm outdated
$ npm list | grep gulp
$ npm update
```

使用快速的安装源：
```bash
$ npm install nrm --global
$ nrm ls
$ nrm test
$ nrm use taobao
```

清除缓存：
```bash
npm cache clean
```

### 上传自己的包：
- 1、编写模块
```js
exports.sayHello =function(){
    return 'Hello World';
}
```
保存为index.js

- 2、初始化包描述文件
```bash
$ npm init
```

package.json
```json
{
    "name": "wu_xx",
    "version": "1.0.1",
    "description": "wu_xx first demo",
    "main": "index.js",
    "scripts": {
        "test": "make test"
    },
    "repository": {
        "type": "Git",
        "url": "git+https://github.com/blackwuxin/testdemo.git"
    },
    "keywords": [
        "demo"
    ],
    "author": "wu_xx",
    "license": "ISC",
    "bugs": {
        "url": "https://github.com/blackwuxin/testdemo/issues"
    },
    "homepage": "https://github.com/blackwuxin/testdemo#readme",
}
```
- 3、注册npm仓库账号
https://www.npmjs.com 上面的账号
```bash
$ npm adduser
```
- 4、上传包
```bash
$ npm publish
```
4.1 卸载npmjs官网上的包
```bash
$ npm unpublish wu_xx --force
```

- 5、安装包
```bash
$ npm install wu_xx
```

- 6、管理包权限
```bash
// 查看模块拥有者
$ npm owner ls <package_name>

// 添加一个发布者
$ npm owner add <user> <package_name>

// 删除一个发布者
$ npm owner rm <user> <package_name>
```

- 7、分析包
```bash
// 查看当前项目引用了哪些包
npm ls
```

- 8、使用引入包
```js
var hello = require('wu_xx');
hello.sayHello();
```
