## 手拉手实现一套多端业务解决方案

### 1. 前言
就前端而言，RN、Web、小程序（微信、QQ、百度、头条等）各端的出现使得业务有多样的展示形式和用户体验。但对于研发需要针对不同端编写多套代码，成本显然非常高。一套代码适配多端显得比较棘手，市面上也有各种多端框架，本文使用taro框架结合业务来和大家分享如何实现一套多端（小程序和H5）业务解决方案。
### 2. 流程图
   ![框架图](https://storage.360buyimg.com/wximg/tarodocs/taro%E4%B8%9A%E5%8A%A1%E6%9E%B6%E6%9E%84%E5%9B%BE.png)
    使用<a href="#cli">脚手架工具</a>初始化<a href="#taro">taro项目</a>完成业务开发，然后编译成小程序和H5两端代码，H5部分直接发布到线上，小程序部分可通过执行代码复制<a href="#copy">自动融合</a>到<a href="#mp">小程序原生项目</a>，接下来让我们通过一个简单的demo一步步实现一套多端业务解决方案。

### 3. <a id="taro">搭建taro项目模板</a>
- 使用@tarojs/cli初始化项目
 
    ```javascript
    // 使用 npm 安装 CLI (使用的是1.3.0版本)
    $ npm install -g @tarojs/cli
    // 初始化项目
    taro init myApp

    // 初始化完项目后的目录结构
    myApp/
    |-- config/
    |   |-- index.js
    |   |-- dev.js
    |   |-- prod.js
    |
    |-- src/
    |   |-- pages/
    |   |   |-- index/
    |   |       |-- index.js
    |   |       |-- index.css
    |   |
    |   |-- app.js
    |   |-- app.css
    |   |-- index.html
    |
    |-- package.json
    ```

- 项目模板化
    ```javascript
    1. 以上taro只是生成简单的项目，如果要进行业务开发，还需要进一步做修改，这里修改后将作为业务开发的项目模板。

    // 目录结构如下：
    myApp/  
    |-- config/
    |   |-- index.js
    |   |-- dev.js
    |   |-- prod.js
    |
    |-- src/
    |   |-- common-t/ // 此处加-t是为了与原生小程序目录做区分
    |   |   |-- css/ // 公共样式
    |   |   |-- js/ // 暴露统一的方法（不同端差异化处理）
    |   |       |-- storage/ // 缓存相关方法
    |   |           |-- index.h5.js // 提供给h5使用的方法
    |   |           |-- index.weapp.js // 提供给小程序使用的方法
    |   |
    |   |-- components-t/ // 此处加-t是为了与原生小程序目录做区分
    |   |
    |   |-- pages/
    |   |   |-- index-t/ // 此处加-t是为了与原生小程序目录做区分
    |   |       |-- index.js
    |   |       |-- index.css
    |   |
    |   |-- app.js
    |   |-- app.css
    |   |-- index.html
    |
    |-- package.json
    |
    |-- copy/
        |-- index.js // 本地小程序融合
        |-- index.config.js // 小程序融合配置

    ```
    ```javascript
    2. 相应文件内容添加

    /**
     * storage差异化封装
     */

    // index.weapp.js文件
    export const setStorageSync = (key, value) => {
        try {
            wx.setStorageSync(key, value)
        } catch (e) {

        }
    }

    // index.h5.js文件
    export const setStorageSync = (key, value) => {
        localStorage.setItem(key, value)
    }


    /**
     * 修改配置文件，根据不同平台打包对应的dist
     */
    // config -> index.js文件（以下是修改部分）
    const env = process.argv[3];
    const config = {
        // 根据编译不同端输出相应代码
        outputRoot: `dist${env}` ,
        weapp: {
            // 编译小程序不输出app.js app.json app.wxss 
            // 由于融合到原生小程序当中，以上打包可以不输出
            appOutput: false
        }
    }
    ```
- 执行构建命令
  ```javascript
  // 正式环境小程序构建命令
  npm run build:weapp 
  // 开发环境小程序构建命令
  npm run dev:weapp

  // 正式环境H5构建命令
  npm run build:h5
  // 开发环境H5构建命令
  npm run dev:h5

  编译完成后，项目根目录下会多出disth5（H5代码）和 distweapp（小程序）目录
  ```

    按以上修改完后基本可以作为项目模板，这里只是简单的模板，对于业务开发还需要添加静态资源域、封装更多的js库&组件库、根据项目添加更多环境配置、h5发布部署等。
### 4. <a id="mp">小程序原生项目</a>
- 通过微信开发者工具生成原生小程序项目
   ```javascript
    //目录结构如下（删除了不必要的文件，以免干扰）。
    myMp/
    |-- pages
    |   |-- index
    |       |-- index.js
    |       |-- index.json
    |       |-- index.wxml
    |       |-- index.wxss
    |
    |-- app.js
    |-- app.json
    |-- app.wxss

    // 添加多端项目融合进来的页面路径
    // app.json 文件(这里只展示添加的部分，这里路径是放主包，建议放分包里，除非一定要放在主包)
    {
        "pages": [
            "pages/index-t/index"
        ]
    }


   ```
### 5. <a id="copy">小程序融合</a>
- copy --> index.js 文件
    ```javascript
    const fs = require('fs');
    const path = require('path');
    const config = require('./index.config');
    //不复制的目录
    const exceptFolder = config.exceptFolder;
    //不复制的文件
    const exceptFile = config.exceptFile;
    //当前的生成的小程序目录
    const dist = config.dist;
    //原生小程序目录路径（融合目标）
    const mpPath = config.miniProgramProjectPath;

    // 复制文件夹
    function copyFolder(copiedPath, resultPath, direct) {
        if (!direct) {
            copiedPath = path.join(__dirname, copiedPath)
        }
        let except = false
        exceptFolder.forEach((item) => {
            if (copiedPath.includes(item)) {
                log('red', '不复制' + item)
                except = true
            }
        })
        if (except) {
            return false
        }
        function log(index, logtxt) {
            const styles = {
                'bold': ['\x1B[1m', '\x1B[22m'],
                'italic': ['\x1B[3m', '\x1B[23m'],
                'underline': ['\x1B[4m', '\x1B[24m'],
                'inverse': ['\x1B[7m', '\x1B[27m'],
                'strikethrough': ['\x1B[9m', '\x1B[29m'],
                'white': ['\x1B[37m', '\x1B[39m'],
                'grey': ['\x1B[90m', '\x1B[39m'],
                'black': ['\x1B[30m', '\x1B[39m'],
                'blue': ['\x1B[34m', '\x1B[39m'],
                'cyan': ['\x1B[36m', '\x1B[39m'],
                'green': ['\x1B[32m', '\x1B[39m'],
                'magenta': ['\x1B[35m', '\x1B[39m'],
                'red': ['\x1B[31m', '\x1B[39m'],
                'yellow': ['\x1B[33m', '\x1B[39m'],
                'whiteBG': ['\x1B[47m', '\x1B[49m'],
                'greyBG': ['\x1B[49;5;8m', '\x1B[49m'],
                'blackBG': ['\x1B[40m', '\x1B[49m'],
                'blueBG': ['\x1B[44m', '\x1B[49m'],
                'cyanBG': ['\x1B[46m', '\x1B[49m'],
                'greenBG': ['\x1B[42m', '\x1B[49m'],
                'magentaBG': ['\x1B[45m', '\x1B[49m'],
                'redBG': ['\x1B[41m', '\x1B[49m'],
                'yellowBG': ['\x1B[43m', '\x1B[49m']
            };
            console.log(styles[index][0], logtxt, styles[index][1])
        }

        function createDir(dirPath) {
            log('green', `创建目录中【${dirPath}】`)
            fs.mkdirSync(dirPath)
        }

        if (!fs.existsSync(resultPath)) {
            createDir(resultPath)
        }
        if (fs.existsSync(copiedPath)) {
            const files = fs.readdirSync(copiedPath, {withFileTypes: true});
            for (let i = 0; i < files.length; i++) {
                const cf = files[i]
                const ccp = path.join(copiedPath, cf.name)
                const crp = path.join(resultPath, cf.name)
                if (cf.isFile()) {
                    if(exceptFile.includes(cf.name)){
                        log('red','此文件不复制：'+cf.name)
                        continue;
                    }
                    //创建文件,使用流的形式可以读写大文件
                    const readStream = fs.createReadStream(ccp)
                    const writeStream = fs.createWriteStream(crp)
                    readStream.pipe(writeStream)
                    log('cyan', `复制文件【${cf.name}】==>【${crp}】`)
                } else {
                    try {
                        //判断读(R_OK | W_OK)写权限
                        fs.accessSync(path.join(crp, '..'), fs.constants.W_OK)
                        copyFolder(ccp, crp, true)
                    } catch (error) {
                        log('red', 'folder write error:' + error);
                    }

                }
            }
        } else {
            log('red', '目录不存在: ' + copiedPath);
        }
    }

    copyFolder(dist, mpPath);

    ```
- copy --> index.config.js 文件
    ```javascript
    module.exports = {
        // 不复制的目录名
        exceptFolder: [],
        //不复制的文件名
        exceptFile: [],
        // 打包后的小程序目录名
        dist: '../distweapp',
        // 原生小程序项目本地路径(终端进入项目执行pwd查看路径)
        // window：'E:/workSpace/mpProject'
        // mac：'/Users/管理员名称/mpProject'
        miniProgramProjectPath: ''
    }
    ```
- 在package.json文件里添加scripts
  ```javascript
  // 这里只展示添加部分
  {
    "scripts": {
        // 小程序融合
        "copy:weapp": "node ./copy/index.js",
        // 生产环境构建并融合
        "build:copy:weapp": "taro build --type weapp && npm run copy:weapp"
    }
  }
  ```
- 代码融合
  ```javascript
  // 编译taro项目后执行融合命令（也可以执行上面构建并融合命令）
  npm run copy:weapp

  执行完，可以查看微信开发者工具看是否生效。
  ```
### 6. <a id="cli">脚手架工具</a>
- 脚手架目录
  ```javascirpt
  taro-cli/
  |-- bin/ 
  |   |-- index.js // 读取终端命令并执行
  |
  |-- command/
  |   |-- init.js // 初始化项目
  |
  |-- package.json
  |
  |-- templates.json // 模板配置
  ```
- package.json文件
  ```javascirpt
  {
    "name": "djtaro-cli",
    "version": "1.0.0",
    "description": "this is taro-cli",
    "keywords": [
        "taro",
        "javascript",
        "taro.js"
    ],
    "main": "index.js",
    "bin": {
        "djtaro": "./bin/index.js"
    },
    "directories": {
        "doc": "docs"
    },
    "dependencies": {
        "chalk": "^2.4.2",
        "co": "^4.6.0",
        "co-prompt": "^1.0.0",
        "commander": "^2.20.0"
    },
    "devDependencies": {},
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "repository": {
        
    },
    "author": "demo",
    "license": "MIT"
  }
  ```
- index.js文件
    ```javascript
    #! /usr/bin/env node

    const program = require('commander');
    const package = require('../package');
    const init = require('../command/init');
    const list = require('../command/list');

    program
        .version(package.version)
        .usage('<command> [options]');
    program.command('init (template)')
        .description("Generate a new project")
        .alias('i')
        .action(template => {
            init(template)
        });
    program.parse(process.argv);
    if(program.args.length==0){
        program.help()
    }

    ```
- templates.json文件
    ```javascript
    {
        "tpl": {
            "djtaro-simple": {
                "branch": "master",
                "url":"", // 上面模板项目的git库地址
                "desc": "A simple djtaro for quick prototyping"
            }
        }
    }
    ```
- init.js文件
    ```javascript
    const exec = require('child_process').exec;
    const co = require('co');
    const prompt = require('co-prompt');
    const templates = require('../templates');
    const chalk = require('chalk');

    module.exports = template => {
        co(function *() {
            // 处理用户输入
            let tplName = template;
            let projectName = yield prompt('Project name: ');
            let gitUrl;
            let branch;

            if (!templates.tpl[tplName]) {
                console.log(chalk.red('\n × Template does not exit!'))
                process.exit()
            }
            gitUrl = templates.tpl[tplName].url;
            branch = templates.tpl[tplName].branch;

            // git命令，远程拉取项目并自定义项目名
            let cmdStr = `git clone ${gitUrl} ${projectName} && cd ${projectName} && git checkout ${branch}`

            console.log(chalk.white('\n Start generating...'));

            exec(cmdStr, (error, stdout, stderr) => {
                if (error) {
                    console.log(error);
                    process.exit()
                }
                console.log(chalk.green('\n √ Generation completed!'));
                console.log(`\n cd ${projectName} && npm install \n`);
                process.exit()
            })
        })
    };
    ```
- 脚手架工具本地调试
  ```javascript
  // 依赖安装
  npm install

  // 本地调试安装
  npm link

  // 初始化项目
  djtaro [init|i] djtaro-simple

  本地测试通过后可以托管到npm上，具体如何发布这里不做详细介绍，自行查询文档。
  ```
### 7. 问题
问题1：按照以上步骤做完后，可能会捏一把汗，为啥花这么大的精力实现转多端，直接使用H5不就行了（H5不存在跨端）。
回答1：这里出于以下几个原因：1. H5嵌套到小程序里，微信提供的JSSDK功能有限，很难满足业务需求 2. H5在小程序里的体验不是很好 3. 小程序支持原生组件。

问题2：一套代码开发不同平台业务，代码比较难维护，因为要考虑多端需求。
回答2：在开发多端业务时，差异化功能封装好，暴露统一的API。对业务进行组件化，开发业务尽力少的做差异化判断。

问题3：直接用多端开发整个项目，为啥还要和小程序原生进行融合？
回答3：对于基础业务（不怎么迭代，功能较固定）使用原生开发能避免多端框架功能跟不上和兼容问题，而且小程序开发相对效率较高。这里的业务多端解决方案适用于快速开发迭代业务，比如：拉新、裂变、促活工具类。能很好的降低维护和试错成本。

问题4：这里只提到了小程序和H5,转其他小程序支持吗？RN支持吗？
回答4：多端业务解决方案同样适用于其他小程序，目前taro框架是支持的。RN目前支持的不是很好。

### 8. 总结与思考
以上多端业务解决方案在到家小程序业务上得到较好的实践。目前京东到家小程序上看到的大部分拉新、裂变、促活、游戏等业务都是使用多端项目开发，通过一套代码开发维护，也大大降低了我们的人力成本。
对于上面技术本身也有很多值得探究的地方，比如如何做到远程自动化融合，如何进行多端和原生小程序公共库的抽离。怎么去优化多端项目等。