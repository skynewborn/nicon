## Nicon

Nicon 是一个集图标上传、展示、使用于一身的字体图标管理平台，流程简单，符合日常开发使用习惯，适合企业在内部部署使用。采用 Iconfont 字体图标替换项目中图片图标的使用，以达到缩减体积、风格统一、提高开发效率等目的。若配合设计师使用，设计师可在平台上管理图标，复用图标，减少设计图标耗费的时间，而开发只负责使用设计师维护好的图标库，减少了与设计师的交流成本。

## 优势
与其他字体图标管理平台相比，它拥有以下优势：

* 使用流程简单，符合日常开发使用习惯，无需在审核管理流程中耗费时间
* 部署简单，平台自带注册、登录功能，还有静态资源路由，只需数据库配置就可部署使用
* 支持接入三方登录、资源上传到三方CDN服务器。使用更安全，资源更稳定
* 支持导出资源多样化，符合多种使用场合，更有配套的导出工具[nicon-tookit](https://github.com/bolin-L/nicon-toolkit), 方便快捷


## 结构设计图
![结构设计图](./docs/图标管理平台结构设计.png)

## 使用流程图
**使用流程图**

![结构设计图](http://edu-image.nosdn.127.net/0a712d18-fd7c-40ea-94ef-ef43dd2a5d88.png?imageView&quality=100)


**开发使用流程图**

![开发使用流程](http://edu-image.nosdn.127.net/2436a363-6a6d-401e-95ba-f45b2d51d263.jpeg)

**设计师参与使用流程图**

![开发使用流程](http://edu-image.nosdn.127.net/99297908-a6e8-4340-8df5-0b0d65b54f07.jpeg)

如果设计师与开发协同参与使用维护图标库，不仅使设计师可以有一个可视化管理字体图标的平台。还可以减少开发与设计的交流时间。


## 服务安装部署
**系统要求**

- linux/unix/windows

**环境要求**

- nodejs 7.0+
- npm 3.10.8+
- mogondb 3.2+
- redis 3.2+

在启动工程之前，必须确保数据库已经启动，且已经把相应的数据库创建好。

1、 克隆项目到本地|服务器

```
git clone git@github.com:bolin-L/nicon.git
```

2、  进入到项目工程nicon安装依赖

```
cd nicon && npm install
```

3、配置数据库信息

在nicon同级文件夹(or 任何地方，但是如果放在nicon文件夹中，名称必须为`start.sh` 或 `start.bat`, 会被ignore掉，更新不会被覆盖)创建一个启动脚本`start.sh`, 用于配置数据库及其他的环境变量配置，可参考[examle](./bin/env_config_example.sh)

```
#!/bin/bash

# DB config

# mongodb
export MONGODB_NAME=iconRepo;
export MONGODB_HOST=127.0.0.1;
export MONGODB_PORT=27017;
export MONGODB_USERNAME='';
export MONGODB_PASSWORD='';


# redis
export REDIS_FAMILY=4;
export REDIS_HOST=127.0.0.1;
export REDIS_PORT=6379;
export REDIS_PASSWORD='';
export REDIS_DB=0;


# config your website host
export productHost='icon.bolin.site';

# start server
npm run publish
```

4、进入到nicon文件夹，执行启动脚本命令

```
sh yourStartConfigPath/start.sh
```

如果不出什么意外，这个时候应该已经启动好了。服务监听的端口是4843, 当然这只是个纯服务，具体的页面还需要部署前端工程[nicon-front](https://github.com/bolin-L/nicon-front)。

## 前端静态资源部署
图标管理平台采用的是前后端完全分离的开发方式，前端代码放在独立的[icon-front](https://github.com/bolin-L/nicon-front)。前端只需要提供完整的静态html页面与其他静态资源即可。静态资源的访问通过配置nginx代理实现页面的访问，跟服务端工程毫无关系，服务端只负责提供异步接口。

1、克隆前端项目到本地, 与nicon文件夹同级

```
git clone git@github.com:bolin-L/nicon-front.git
```

2、进入到nicon-front工程，安装依赖

```
cd nicon-front && npm install 
```

3、运行打包命令、打包输出到nicon-front/dist文件夹下

```
npm run build
```

现在服务已经启动，静态资源已经输出，接下来需要配置nginx让请求可以访问到静态资源，异步接口可以访问到服务。

## Nginx配置
当前端静态资源与后端服务部署在同一台机器，并且使用本地管理图标库字体文件资源时的nginx配置如下


```
server {
    listen 80;
    listen [::]:80;

    server_name icon.bolin.site;

    access_log   /var/log/nginx/bolin_sites_icon.access.log main;
    error_log    /var/log/nginx/bolin_sites_icon.error.log;

    include common.conf;
    # 静态资源请求
    location / {
        root /home/liaobolin/app/nicon-front/dist;
        index index.html index.htm;
    }
    
    location ^~ /static {
        root /home/liaobolin/app/nicon-front/dist;
    }
	# 配置异步接口请求到服务器
    location /api {
		proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass http://127.0.0.1:4843;
    }
	# 服务端的字体文件资源请求
    location ^~ /resource {
        root /home/liaobolin/app/nicon;
    }
}
```

配置到此，平台基本就可以运行起来使用了，浏览器访问`icon.bolin.site`就可以访问到首页

## 三方服务接入

虽然该平台已经可以提供完成的登录、注册，图标库样式文件等静态资源的访问。但是对于企业来说，内部的工具平台最好只接受内部人或只能内网访问，对于静态资源最理想的就是放到自家的cdn服务器上，让平台操作更安全，访问所速度更快等等...

基于这样的需求，Nicon支持接入三方登录与字体文件资源上传到三方服务器，登录或上传需要的密钥等需要通过环境变量设置，比如放在`start.sh`文件中，登录、上传等接入代码需要自己按照规范开发完成提PR，检查完毕后合并到工程`master`分支中。目前代码中支持一下三方服务：

**三方登录**

- 网易openId登录
- github授权登录

**三方上传**

- 网易NOS服务
- 七牛云存储服务

三方服务接入文件夹结构如下

```
├── service
│   ├── login
│   │   ├── default
│   │   │   ├── config.js
│   │   │   └── index.js
│   │   ├── github
│   │   │   ├── config.js
│   │   │   └── index.js
│   │   ├── github_qiniu
│   │   │   ├── config.js
│   │   │   └── index.js
│   │   ├── index.js
│   │   └── netease
│   │       ├── config.js
│   │       └── index.js
│   └── upload
│       ├── default
│       │   ├── config.js
│       │   └── index.js
│       ├── github_qiniu
│       │   ├── config.js
│       │   └── index.js
│       ├── index.js
│       ├── netease
│       │   ├── config.js
│       │   ├── index.js
│       │   ├── nosService.js
│       │   └── upload2NosService.js
│       └── qiniu
│           ├── config.js
│           └── index.js

```


### 三方登录、上传服务接入步骤
1、在接入三方服务之前，必须先配置产品类型环境变量 `productType`。加入到`start.sh`文件中

```
export productType='github_qiniu';
```

比如我需要接入github三方登录与qiniu上传存储服务，那么我的`productType`就设置为`github_qiniu`。 那么在开发接入服务时文件夹名称必须为 `github_qiniu`。


2、 在`service/login/` 文件夹下创建文件夹 `github_qiniu`, 然后在该文件夹下创建`config.js` , 与`index.js`, 在index.js文件中必须暴露出async `login`方法, 调用方法后需要返回指定格式的数据

```
// index.js

require('request');
let rp = require('request-promise');
let config = require('./config');

class GithubOpenIdLogin {
    async login (ctx) {
        return this.getUserBaseInfo(ctx);
    }

    async getUserBaseInfo (ctx) {
        // your code
        
        // login 方法返回的数据格式
		return {
		    userName: tokenInfo.sub, // 必须且唯一
		    password: tokenInfo.sub,
		    email: openIdUserInfo.email,
		    nickName: openIdUserInfo.nickname,
		    fullName: openIdUserInfo.fullname
		}
    }
}

let loginIns = new GithubOpenIdLogin();
module.exports = loginIns.login.bind(loginIns);

```

3、在`service/upload/` 文件夹下创建文件夹 `github_qiniu`, 然后在该文件夹下创建`config.js` , 与`index.js`, 在index.js文件中必须暴露出async `upload`方法, 调用方法后需要返回指定格式的数据

```
// index.js

let config = require('./config');
let qiniu = require('qiniu');

class QiniuUpload {
    async upload (dirPath) {
        let fontMap = await this.uploadFonts(dirPath);
        // 上传font完毕后替换css中的引用
        let cssContent = await this.replaceFontsInCss(dirPath, fontMap);
        let cssUrl = await this.uploadCss(dirPath, cssContent);
        
        // 上传返回数据格式
        return {
            url: cssUrl, // 必须
            cssContent: cssContent // 必须
        };
    }
}

let uploadIns = new QiniuUpload();
module.exports = uploadIns.upload.bind(uploadIns);

```

至此就已经配置完毕，启动工程就好了。

## 单元测试
待补充....

## License
MIT



