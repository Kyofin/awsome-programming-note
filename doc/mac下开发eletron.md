# mac下开发Eletron 

下载quickstart

```
git clone https://github.com/electron/electron-quick-start
```

下载完后进入目录，安装依赖。

```
~/front-code/electron-quick-start(master) » npm install
```

由于网络问题，会在安装eletron时报错。

![](http://image-picgo.test.upcdn.net/img/20200113092504.png)

跟踪该`install.js`,可以在`node_modules/@electron/get/dist/cjs/artifact-utils.js`文件中找到默认下载地址。

![](http://image-picgo.test.upcdn.net/img/20200113092729.png)

需要修改这个文件中的`getArtifactRemoteURL`方法，改成淘宝的地址，并且根据`package.json`中的eletron依赖版本改成对应版本，这里用的是7.1.7。

![](http://image-picgo.test.upcdn.net/img/20200113092823.png)

修改完后，再执行。

```
~/front-code/electron-quick-start(master) » node node_modules/electron/install.js
```

安装完后，就可以启动项目了。

```
~/front-code/electron-quick-start(master) » npm start                                                                                                        
```

![](http://image-picgo.test.upcdn.net/img/20200113093141.png)