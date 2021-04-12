1、谈谈你对工程化的初步认识，结合你之前遇到过的问题说出三个以上工程化能够解决问题或者带来的价值。

  答案：工程化是根据业务特点，将前端开发流程规范化，标准化，它包括了开发流程，技术选型，代码规范，构建发布等，用于提升前端工程师的开发效率和代码质量。

优点：
	
	1、制定各项规范，编码规范，开发流程规范，前后端接口规范等等
	2、使用合适前端技术和框架，提高生产效率，降低维护难度，采用模块化，组件化，数据分离等
	3、代码可测试，单元测试，端到端测试等   
	4、开发部署自动化 

2、你认为脚手架除了为我们创建项目结构，还有什么更深的意义？    

	1、减少重复性的工作，不需要复制其他项目再删除无关代码，或者从零创建一个项目和文件。  
	2、可以根据交互动态生成项目结构和配置文件。  
	3、多人协作更为方便，不需要把文件传来传去。  


3、概述脚手架实现的过程，并使用 NodeJS 完成一个自定义的小型脚手架工具

	3.1、首先创建目录 初始化 yarn init 创建出 package.json
	3.2、在package.json中 输入 bin入口



```
	{
 	  "bin": "lib.js",
	}
```

	3.3、在根目录创建 lib.js文件 添加bin 入口标识 
	#!/usr/bin/env node

	3.4、引入inquirer 模块 创建用户与命令行交互的工具 编写所需问题及字段

	3.5、创建模板目录templates 将项目文件导入到目录中

	3.7、引入ejs模块 结合所需功能问题变量 改写 templates 下项目文件 达到所需功能

	3.8、在inquirer回调中 结合nodejs 读写功能 和ejs 模块将问题变量 重写到项目中

	3.9、然后发布到npm上


```
#!/usr/bin/env node

const fs = require('fs')
const path = require('path')
const inquirer = require('inquirer')
const ejs = require('ejs')

inquirer.prompt([
  {
    type: 'input',
    name: 'name',
    message: 'Project name?'
  },
  {
    type: 'list',
    name: 'theme',
    message: 'Select the theme color',
    choices: ['Dark', 'Light'],
    filter: function (val) {
      return val.toLowerCase();
    },
  },
  {
    type: 'checkbox',
    message: 'Select what to include',
    name: 'content',
    choices: [
      {
        name: 'Header',
      },
      {
        name: 'Body',
      },
      {
        name: 'Footer',
      },
    ],
    validate: function (answer) {
      if (answer.length < 1) {
        return 'You must choose at least one content.';
      }

      return true;
    },
  },
  
])
.then(anwsers => {
  const tmplDir = path.join(__dirname, 'templates')
  const destDir = process.cwd()

  fs.readdir(tmplDir, (err, files) => {
    if (err) throw err
    files.forEach(file => {
      ejs.renderFile(path.join(tmplDir, file), anwsers, (err, result) => {
        if (err) throw err

        fs.writeFileSync(path.join(destDir, file), result)
      })
    })
  })
})
```

4、尝试使用 Gulp 完成项目的自动化构建
```
const path = require('path');
const sass = require('sass');

const del = require('del');
const browserSync = require('browser-sync');

const loadGruntTasks = require('load-grunt-tasks');
const autoprefixer = require('autoprefixer');
const stylelint = require('stylelint');
const scss = require('postcss-scss');
const reporter = require('postcss-reporter');
const minimist = require('minimist');

const bs = browserSync.create();
const cwd = process.cwd();

const args = minimist(process.argv.slice(2));

const isProd = process.env.NODE_ENV ? process.env.NODE_ENV === 'production' : args.production || args.prod || false;

const bsInit = {
notify: false,
port: args.port || 2080,
open: args.open || false,
};

let config = {
build: {
src: 'src',
dist: 'dist',
temp: 'temp',
public: 'public',
paths: {
styles: 'assets/styles//.scss',
scripts: 'assets/scripts//.js',
pages: '/.html',
images: 'assets/images//.{jpg,jpeg,png,gif,svg}',
fonts: 'assets/fonts/*/.{eot,svg,ttf,woff,woff2}',
},
},
};

try {
const loadConfig = require(${cwd}/pages.config.js);
config = { ...config, ...loadConfig };
} catch (e) { }

module.exports = (grunt) => {
grunt.initConfig({
sass: {
options: {
sourceMap: !isProd,
implementation: sass,
outputStyle: 'expanded',
},
main: {
expand: true,
cwd: config.build.src,
src: [config.build.paths.styles],
dest: config.build.temp,
ext: '.css',
},
},
postcss: {
main: {
options: {
processors: [
autoprefixer(),
],
},
expand: true,
cwd: config.build.temp,
src: ['assets/styles/*/.css'],
dest: config.build.temp,
},

  lint: {
    options: {
      processors: [
        stylelint({ fix: args.fix }),
        reporter(),
      ],
    },
    src: `${path.join(config.build.src, config.build.paths.styles)}`,
  },
},

eslint: {
  options: {
    fix: args.fix,
  },
  main: `${path.join(config.build.src, config.build.paths.scripts)}`,
},
babel: {
  options: {
    sourceMap: !isProd,
    presets: ['@babel/preset-env'],
  },
  main: {
    expand: true,
    cwd: config.build.src,
    src: [config.build.paths.scripts],
    dest: config.build.temp,
    ext: '.js',
  },
},
html_template: {
  options: {
    cache: false,
    locals: config.data,
  },
  main: {
    expand: true,
    cwd: config.build.src,
    src: [config.build.paths.pages, '!layouts/**', '!partials/**'],
    dest: config.build.temp,
    ext: '.html',
  },
},
imagemin: {
  image: {
    expand: true,
    cwd: config.build.src,
    src: [config.build.paths.images],
    dest: config.build.dist,
  },
  font: {
    expand: true,
    cwd: config.build.src,
    src: [config.build.paths.fonts],
    dest: config.build.dist,
  },
},
copy: {
  main: {
    expand: true,
    cwd: config.build.public,
    src: ['**'],
    dest: config.build.dist,
  },
  html: {
    expand: true,
    cwd: config.build.temp,
    src: [config.build.paths.pages],
    dest: config.build.dist,
  },
},
useminPrepare: {
  main: {
    expand: true,
    cwd: config.build.temp,
    src: [config.build.paths.pages],
  },
  options: {
    dest: config.build.dist,
    root: [config.build.temp, '.', '..'],
  },
},

usemin: {
  main: {
    expand: true,
    cwd: config.build.dist,
    src: [config.build.paths.pages],
  },
  options: {

  },
},
'gh-pages': {
  options: {
    base: config.build.dist,
    branch: 'gh-pages-grunt',
  },
  main: ['**'],
},
watch: {
  script: {
    files: [`${path.join(config.build.src, config.build.paths.scripts)}`],
    tasks: ['babel'],
  },
  style: {
    files: [`${path.join(config.build.src, config.build.paths.styles)}`],
    tasks: ['style'],
  },
  page: {
    files: [`${path.join(config.build.src, config.build.paths.pages)}`],
    tasks: ['html_template'],
  },
},
browserSync: {
  dev: {
    bsFiles: {
      src: [
        config.build.temp,
        config.build.dist,
      ],
    },
    options: {
      ...bsInit,
      watchTask: true,
      server: {
        baseDir: [config.build.temp, config.build.dist, config.build.public, config.build.src],
        routes: {
          '/node_modules': 'node_modules',
        },
      },
    },
  },
  prod: {
    bsFiles: {
      src: config.build.dist,
    },
    options: {
      ...bsInit,
      server: {
        baseDir: config.build.dist,
      },
    },
  },
},
});

loadGruntTasks(grunt);

grunt.registerTask('reload', () => {
bs.reload();
});

grunt.registerTask('stylesLint', []);

grunt.registerTask('scriptsLint', []);

grunt.registerTask('clean', () => {
del.sync([config.build.dist, config.build.temp, '.tmp']);
});

grunt.registerTask('style', ['sass', 'postcss:main']);

grunt.registerTask('compile', ['style', 'babel', 'html_template']);

grunt.registerTask('build', [
'clean',
'compile',
'copy',
'useminPrepare',
'concat:generated',
'cssmin:generated',
'uglify:generated',
'usemin',
'imagemin',
]);

grunt.registerTask('serve', ['compile', 'browserSync:dev', 'watch']);

grunt.registerTask('start', ['build', 'browserSync:prod']);

grunt.registerTask('deploy', ['build', 'gh-pages']);

grunt.registerTask('lint', ['postcss:lint', 'eslint']);
};

```






























