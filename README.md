# Using Grunt

## Installing Grunt local
Create the package.json file in the conFusion folder with the following content:

```
{
  "name": "conFusion",
  "private": true,
  "devDependencies": {},

  "engines": {
    "node": ">=0.10.0"
  }
}

```
Install Grunt to use within your project. To do this, go to the app folder and type the following at the prompt:

```bash
npm install grunt --save-dev

```
## Grunt file
You need to create a Grunt file containing the configuration for all the tasks to be run when you use Grunt. To do this, create a file named Gruntfile.js in the app folder. Add the following code to Gruntfile.js to set up the file to configure Grunt tasks:

```javascript
'use strict';

module.exports = function (grunt) {
  // Define the configuration for all the tasks
  grunt.initConfig({

  });
};
```

This sets up the Grunt module ready for including the grunt tasks inside the function above.

## Configuring the project
You need to configure the html files. Open the html file and update the CSS import lines in the head of the page as follows:

```html
<!-- Bootstrap -->

<!-- build:css styles/main.css -->
  <link href="../bower_components/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="../bower_components/bootstrap/dist/css/bootstrap-theme.min.css" rel="stylesheet">
  <link href="../bower_components/font-awesome/css/font-awesome.min.css" rel="stylesheet">
  <link href="styles/bootstrap-social.css" rel="stylesheet">
  <link href="styles/mystyles.css" rel="stylesheet">
<!-- endbuild -->
```

Place all of the javascript in a file in your script directory.

```html
<!-- build:js scripts/main.js -->
  <script src="../bower_components/angular/angular.min.js"></script>
  <script src="scripts/app.js"></script>
<!-- endbuild -->
```

## JSHint
JSHint helps validate JavaScript code and points out errors and gives warnings about minor violations. Install the following modules by typing the following at the prompt:

```bash
npm install grunt-contrib-jshint --save-dev
npm install jshint-stylish --save-dev
npm install time-grunt --save-dev
npm install jit-grunt --save-dev
```
1. jshint-stylish: one adds further to print out the messages from JSHint in a better format.
2. time-grunt: generates time statistics about how much time each task consumes.
3. jit-grunt: includes the necessary downloaded Grunt modules when needed for the tasks.

### Update Grunt.js for the new modules.

```javascript
// Time how long tasks take. Can help when optimizing build times
require('time-grunt')(grunt);

// Automatically load required Grunt tasks
require('jit-grunt')(grunt);

// Define the configuration for all the tasks
grunt.initConfig({
  pkg: grunt.file.readJSON('package.json'),

  // Make sure code styles are up to par and there are no obvious mistakes
  jshint: {
    options: {
      jshintrc: '.jshintrc',
      reporter: require('jshint-stylish')
    },

    all: {
      src: [
        'Gruntfile.js',
        'app/scripts/{,*/}*.js'
      ]
    }
  }
});

grunt.registerTask('build', [
  'jshint'
]);

grunt.registerTask('default',['build']);
```

In the above, we have set up time-grunt and jit-grunt to time the tasks, and load Grunt modules when needed. The JSHint task is set to examine all the JavaScript files in the app/scripts folder, and the Gruntfile.js and generate any reports of JS errors or warnings. We have set up a Grunt task named build, that runs the jshint task and the build task is also registered as the default task.

## jshintrc
 create a file named .jshintrc (Don't forget the . in front of jshintrc) in the app folder, and include the following in the file:

```
{
  "bitwise": true,
  "browser": true,
  "curly": true,
  "eqeqeq": true,
  "esnext": true,
  "latedef": true,
  "noarg": true,
  "node": true,
  "strict": true,
  "undef": true,
  "unused": true,
  "globals": {
    "angular": false
  }
}
```
Run the grunt command and check for errors.

## Copying the Files and Cleaning Up the Dist Folder
No we want to copy over files to a distribution folder named dis, and clean up the dist folder when needed. To do this install:

```bash
npm install grunt-contrib-copy --save-dev
npm install grunt-contrib-clean --save-dev
```
Update Gruntfile.js

```javascript
copy: {
  dist: {
    cwd: 'app',
    src: [ '**','!styles/**/*.css','!scripts/**/*.js' ],
    dest: 'dist',
    expand: true
  },

  fonts: {
    files: [
      {
        //for bootstrap fonts
        expand: true,
        dot: true,
        cwd: 'bower_components/bootstrap/dist',
        src: ['fonts/*.*'],
        dest: 'dist'
      }, {
        //for font-awesome
        expand: true,
        dot: true,
        cwd: 'bower_components/font-awesome',
        src: ['fonts/*.*'],
        dest: 'dist'
      }
    ]
  }
},

clean: {
  build: {
    src: [ 'dist/']
  }
}
```
Update the Grunt build task in the file as follows:
```javascript
grunt.registerTask('build', [
  'clean',
  'jshint',
  'copy'
]);
```
Run grunt. You should now have a dist folder with subfolders for your files.

## Get ugly
 Use the Grunt usemin module together with concat, cssmin, uglify and filerev to prepare the distribution folder. To do this, install the following Grunt modules:

```bash
npm install grunt-contrib-concat --save-dev
npm install grunt-contrib-cssmin --save-dev
npm install grunt-contrib-uglify --save-dev
npm install grunt-filerev --save-dev
npm install grunt-usemin --save-dev
```
Update the task configuration within the Gruntfile.js with the following additional code to introduce the new tasks:

```javascript
useminPrepare: {
  html: 'app/menu.html',
  options: {
    dest: 'dist'
  }
},

// Concat
concat: {
  options: {
    separator: ';'
  },

  // dist configuration is provided by useminPrepare
  dist: {}
},

// Uglify
uglify: {
  // dist configuration is provided by useminPrepare
  dist: {}
},

cssmin: {
  dist: {}
},

// Filerev
filerev: {
  options: {
    encoding: 'utf8',
    algorithm: 'md5',
    length: 20
  },

  release: {
    // filerev:release hashes(md5) all assets (images, js and css )
    // in dist directory
    files: [{
      src: [
        'dist/scripts/*.js',
        'dist/styles/*.css',
      ]
    }]
  }
},

// Usemin
// Replaces all assets with their revved version in html and css files.
// options.assetDirs contains the directories for finding the assets
// according to their relative paths
usemin: {
  html: ['dist/*.html'],
  css: ['dist/styles/*.css'],
  options: {
    assetsDirs: ['dist', 'dist/styles']
  }
},
```

Update the jit-grunt configuration as follows, to inform it that useminPrepare task depends on the usemin package:

```javascript
require('jit-grunt')(grunt, {
    useminPrepare: 'grunt-usemin'
});
```

 Update the Grunt build task as follows:

```javascript
grunt.registerTask('build', [
  'clean',
  'jshint',
  'useminPrepare',
  'concat',
  'cssmin',
  'uglify',
  'copy',
  'filerev',
  'usemin'
]);
```
Now if you run Grunt, it will create a dist folder with the files structured correctly to be distributed to a server to host your website.

## Watch, Connect and Serve Tasks
To spin up a web server and keep a watch on the files and automatically reload the browser when any of the watched files are updated, install the following grunt modules:
```bash
npm install grunt-contrib-watch --save-dev
npm install grunt-contrib-connect --save-dev
```
Configure the connect and watch tasks by adding the following code to the Grunt file:

```javascript
watch: {
  copy: {
    files: [ 'app/**', '!app/**/*.css', '!app/**/*.js'],
    tasks: [ 'build' ]
  },

  scripts: {
    files: ['app/scripts/app.js'],
    tasks:[ 'build']
  },

  styles: {
    files: ['app/styles/mystyles.css'],
    tasks:['build']
  },

  livereload: {
    options: {
      livereload: '<%= connect.options.livereload %>'
    },

    files: [
      'app/{,*/}*.html',
      '.tmp/styles/{,*/}*.css',
      'app/images/{,*/}*.{png,jpg,jpeg,gif,webp,svg}'
    ]
  }
},

connect: {
  options: {
    port: 9000,
    // Change this to '0.0.0.0' to access the server from outside.
    hostname: 'localhost',
    livereload: 35729
  },

  dist: {
    options: {
      open: true,
      base:{
        path: 'dist',
        options: {
          index: 'menu.html',
          maxAge: 300000
        }
      }
    }
  }
},
```
Add the following task to the Grunt file:
```javascript
grunt.registerTask('serve',['build','connect:dist','watch']);
```

Runt `grunt serve` at the command line and grunt will open the web page, watch files in the app folder, and rebuild the project and load the page if you update any of the files.
