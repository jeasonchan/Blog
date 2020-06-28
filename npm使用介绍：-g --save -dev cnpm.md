# 1 什么是npm
要介绍npm还要先从node.js介绍起，node.js说白了就是“优化过的v8引擎+一堆常用的js脚本”，v8本来是内嵌在浏览器中解释执行js的引擎，但是，后来有人将其移植出来病经过优化，更加适合脱离浏览器去解释并执行js脚本文件，为了方便使用，再加上一些必要的js脚本，并经过封装，最终得到nodejs，使js脚本文件脱离浏览器运行给予了js很多种玩法，因此，跟Python一样，有了很多好用的包，为了方便管理第三方js包/模块，就有了npm，因此，npm和pip十分相似，都是官方提供的管理第三方包的工具，下载的地址都可以自定义。但是，也有包括但不限于以下不同：
* pip install xxxx，都是默认直接状态Python的安装目录中的，因此，安装的库都是全局可调用的
* npm install和npm install -g 分别适用于文件夹/项目局部安装和计算机全局安装了
（**当然，只要适当的、手动的放在项目的文件夹中，不需要npm install也肯定能调用，不过很容易出错，管理起来也麻烦**）
也有以下相同点：
* 可自动处理包之间的依赖关系，并安装缺失依赖
* （……）

# 2 基本使用方法
使用方法肯定是给予项目来探讨的，node的项目以文件夹为单位，比如在"./test"文件夹中打开终端，"npm init"，和"git init"一样，会将，这个文件文件夹初始化为node项目的文件结构，此时，只会增加package.json这一个文件，这是用来描述文件夹中的项目，比如，刚初始化完，是这样的：
```json
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
内容，显而易见，比如，指定了项目入口脚本为index.js，可执行的shell脚本"test"（直接使用 npm test即可执行）。
这时候，运行一次"npm install"，会发现项目文件中多出了“node_modules”这个文件夹，但是，如果在一个空白的目录中运行"npm install"会提示你找不到package.jssn文件，因此，"npm install "其实是读了json文件中的信息，但是没读到要安装什么，就建立了一个空文件夹。
在以上基础上，开始区分--save 、-dev区别。
## 2.1 npm install
会下载**package.json中dependencies和devDependencies**中描述的模块，显而易见，依赖有开依赖和运行依赖，npm将这两种场景区分开了（但是，脚本语言，运行场景和开发场景有什么不一样吗……可能是测试用的代码会用到开发依赖，运行的用不到吧……）

当使用npm install –production或者注明NODE_ENV变量值为production时，只会下载dependencies中的模块。

npm install 单个模块，仅安装到node_modules目录中，但不会将依赖信息保存在package.json 中，之后运行npm install命令时，自然不会自动安装该模块。不过，一般git都是上传整个项目文件，一般都是会包含node_modules目录的。但是，为了项目规范，还建议安装模块时，顺便讲依赖写入到package.json当中。如何在安装模块时，自动添加依赖信息到json文件中呢？添加--save命令参数。

## 2.2 npm install --save

安装到node_modules目录中，同时将依赖信息写入到package.json中dependencies字段下。安装生产环境依赖的模块，即项目运行时的模块，例如react，react-dom,jQuery等类库或者框架。
比如，"npm install --save express"后，会在模块文件夹下下载好express的最新包，同时，讲express作为运行时依赖添加到package.json中：
```json
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```
可见，已多出key"dependencies"。
## 2.3 npm install --save-dev
安装到node_modules目录 中，同时将依赖信息写入到package.json中devDependencies字段下。安装开发环境依赖的模块，即项目开发时的模块，例如babel（转码器，可以将ES6代码转为ES5代码）等一些工具，只需在开发环境是用到。
## 2.4 npm install -g
全局安装，并写入环境变量，可全局调用，比如，cnpm就是全局安装的，同样的，还有angular-cli，正是因为全局安装了，才能直接使用angular命令创建angular项目。
## 2.4 npm配置
函数名即文档！还是比较容易理解的。
```bash
npm config list
npm config list -g
npm config get registry
npm config set registry xxxx #设置npm的下载源，不过，我设置了了阿里的源还是慢，索性直接换用cnpm
```

# 3 cnpm和npm
```bash
sudo npm install -g cnpm --registry="阿里源"  #临时更换源的方式安装cnpm
```
注意，在linux下，因为要向 /usr/bin 中写入东西是要root权限的，因此，需要sudo临时申请root权限。为什么要向/usr/bin写入东西？因为，-g 的这个参数命令就是希望安装一个js脚本，方便之后直接在终端调用。
cnpm是阿里团队开发的和npm几乎一模一样的第三方包管理软件，具体区别是，下载源换为阿里的源，同时传输过程进行了数据压缩，下载速度更快，同时，极个别命令暂时不支持，不过对平时开发完全没有区别。

