---
title: Gulp合并requirejs并MD5文件
date: 2016-12-11 16:46:51
categories:
- FrontEnd
---

## 项目目录结构
{% asset_img 目录结构.jpg 目录结构 %}

可以从https://github.com/muchstudy/GulpDemo 这里下载样例代码，求Star!  

说明：  
1. js的的依赖关系为`main.build.js`-->`three.js`-->`two.js`-->`one.js`
2. `main-build.js`为构建的入口
3. 从github上拿下来的项目是可以直接运行的。可执行`gulp clean`与`gulp`观察结果。

## 文件合并

1. 在requirejs目录下运行`npm init`，初始化package.json
2. 运行`npm install gulp --save-dev`、`npm install gulp-requirejs-optimize --save-dev`、``npm install gulp-rename --save-dev`

文件合并代码如下:
```javascript
var gulp            = require('gulp');
var reqOptimize     = require('gulp-requirejs-optimize');   //- requireJs文件合并所需模块，选择该模块的原因为相对于其它模块活跃度较高
var rename          = require("gulp-rename");               //- 文件重命名

gulp.task("optimize",function () {
    gulp.src("app/main-build.js")
        .pipe(reqOptimize({
            optimize:"none",                                //- none为不压缩资源
            //findNestedDependencies: true,                 //- 解析嵌套中的require
            paths:{
                "PDAppDir":"",                              //- 所有文件的路径都相对于main-build.js，所以这里为空即可
                "jquery":"empty:"
            }
        }))
        .pipe(rename("main.min.js"))
        .pipe(gulp.dest('app'));                            //- 映射文件输出目录
});
```

说明：  
1. 运行`gulp optimize`即可把`main-build.js`所依赖的文件合并到`main.min.js`中。
2. 在代码中有一个`findNestedDependencies`参数，意思是代码中如果是使用`require`方式而不是`define`方式依赖文件，默认不解析
3. `:empty`代表忽略该文件
4. reqOptimize的详细参数见requirejs的<a href="https://github.com/requirejs/r.js/blob/master/build/example.build.js">官方文档</a>

## 文件md5

运行：`npm install gulp-rev --save-dev`

### MD5
文件MD5代码如下：
```javascript
gulp.task("md5",function (cb) {
    gulp.src("app/main-build.js")
        .pipe(reqOptimize({
            optimize:"none",                                //- none为不压缩资源
            //findNestedDependencies: true,                 //- 解析嵌套中的require
            paths:{
                "PDAppDir":"",                              //- 所有文件的路径都相对于main-build.js，所以这里为空即可
                "jquery":"empty:"
            }
        }))
        .pipe(rev())                                        //- 文件名加MD5后缀
        .pipe(gulp.dest("app"))                             //- 生成MD5后的文件
        .pipe(rev.manifest({merge:true}))                   //- 生成一个rev-manifest.json
        .pipe(gulp.dest(''))                                //- 映射文件输出目录
        .on('end', cb);
});
```

运行`gulp md5`，此时就会生成类似于这样的文件`main-min-c144065c18.js`。现在问题就来了，文件名变了，就需要动态替换首页上的引用文件名。

### 路径替换
运行：`mpn install gulp-rev-collector --save-dev`、`npm install through2 --save-dev`
代码如下:

```javascript
function modify(modifier) {
    return through2.obj(function(file, encoding, done) {
        var content = modifier(String(file.contents));
        file.contents = new Buffer(content);
        this.push(file);
        done();
    });
}
function replaceSuffix(data) {
    return data.replace(/\.js/gmi, "");
}
//去掉.js后缀，因为requirejs的引用一般都不带后缀
gulp.task("replaceSuffix",function (cb) {
    gulp.src(['rev-manifest.json'])
        .pipe(modify(replaceSuffix))            //- 去掉.js后缀
        .pipe(gulp.dest(''))
        .on('end', cb);
});

gulp.task("replaceHomePath",function (cb) {
    gulp.src(['rev-manifest.json', 'index-build.html'])
        .pipe(revCollector())                   //- 替换为MD5后的文件名
        .pipe(rename("index.html"))
        .pipe(gulp.dest(''))
        .on('end', cb);
});
```
依次运行`gulp replaceSuffix`、`gulp replaceHomePath`即可生成可运行的首页。相对于原始首页，依赖的文件路径为md5后的路径。

## 整合
从上面可以看到，这一套下来需要运行一系列的task，而且这些task的运行还有先后顺序。下面尝试使用一个task来完成整个任务。由于gulp的task都是异步运行的，所以需要使用到`run-sequence`  
安装`npm install gulp-clean --save-dev`、`npm install run-sequence --save-dev`

```javascript
//删除掉上一次构建时创建的资源
gulp.task("clean",function () {
    return gulp.src([
        'rev-manifest.json',
        '**/*-build-*.js',
        'index.html'
    ]).pipe(clean());
});

//构建总入口
gulp.task('default', function(callback) {
    runSequence(
        "clean",                //- 上一次构建的结果清空
        "md5",                  //- 文件合并与md5
        "replaceSuffix",        //- 替换.js后缀
        "replaceHomePath",      //- 首页路径替换为md5后的路径
        callback);
});
```
说明:  
1. 可以看到上面的task中最后都有一句`.on('end', cb)`这个是为了解决task任务异步运行的问题
2. 此时，只需要运行`gulp`即可完成所有的构建任务。
