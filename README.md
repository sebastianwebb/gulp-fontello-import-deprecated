### sebcus ###

Needed to add function to catch potential 400 errors when connecting to fontello because it doesn't like the format of the config.json file:

function getIconFont(options, cb) {
 
  return apiRequest(options,

      // success callback
      function(sessionUrl) {
        var zipFile;
        zipFile = needle.get("" + sessionUrl + "/get", function(error, response, body) {
          if (error) {
            throw error;
          }
        });
        if (options.css && options.font) {
          return zipFile.pipe(unzip.Parse()).on('entry', (function(entry) {
            var cssPath, dirName, fileName, fontPath, pathName, type, _ref;
            pathName = entry.path, type = entry.type;
            if (type === 'File') {
              dirName = (_ref = path.dirname(pathName).match(/\/([^\/]*)$/)) != null ? _ref[1] : void 0;
              fileName = path.basename(pathName);
              switch (dirName) {
                case 'css':
                  cssPath = path.join(options.css, fileName);
                  return entry.pipe(fs.createWriteStream(cssPath));
                case 'font':
                  fontPath = path.join(options.font, fileName);
                  return entry.pipe(fs.createWriteStream(fontPath));
                default:
                  return entry.autodrain();
              }
            }
          })).on('finish', (function() {
            console.log('Install complete.\n');
            cb();
            return;
          }));
        } else {
          return zipFile.pipe(unzip.Extract({
            path: 'icon-example'
          })).on('finish', (function() {
            console.log('Install complete.\n');
            cb();
            return;
          }));
        }
      },

      // error callback (sebcus)
      // When I tried to add the font awesome star icon, I got an error.
      // "data.glyphs.356 no (or more than one) schemas match"
      // maybe because in config.json the width was set to null for some unknown reason
      // the height="1em" in the SVG possibly...
      function(response){
        console.log('There was an error connecting to fontello', response.statusMessage, '('+response.statusCode+')');
        console.log(response.body);
      }

  );
}




gulp-fontello-import
====================

Import svg files to fontello icon font project, use svg filename as glyph name. Also provide task for auto download exported css and font files into desinated folder.

_This plugin currently is not utilizing streams for gulp, and used a lot of sync operation. This will be improved later with a pure gulp stream solution._

## Recommended Structure

<pre>
---project folder
   |--css (location can be specified in task options, output icon font css)
   |--fonts (location can be specified in task options, output icon fonts)
   |--svg-src (location can be specified in task options, source svg files)
   |--gulpfile.js (write your task here, can take the example gulpfile as reference)
   |--config.json (fontello config file, this path can be specified in task options)
</pre>

## Install

Add this in your pacakge.json:

``` javascript
"devDependencies": {
    "gulp": "^3.8.10",
    "gulp-fontello-import": "fireball-x/gulp-fontello-import"
}
```
And run:

``` bash
npm install
```

## Usage

You should create a `config.json` file somewhere in your project, with the following content:
``` javascript
{
    "name": "font-name",
    "css_prefix_text": "icon-",
    "css_use_suffix": false,
    "hinting": true,
    "units_per_em": 1000,
    "ascent": 850,
    "glyphs": []
}
```
Customization on name, prefix and units are available, just edit the file.
You can also copy the `config.json` file included in this plugin as a starting point.

Next, you should have a folder with all your source svg files. You should manage this folder so that your svg icons are identified by their filename. **We will use svg file name as the base of css classname of generated icon font.**

You can add, replace svg files in that folder. Just make sure the naming of svg files are consistent. **NOTE: if you remove a svg icon from the source folder, you have to remove the corresponding entry manually in your config.json file. The importer does not handle icon removal automatically.**

### Add Fontello Icons

There are icon font sets on <http://fontello.com> website you can add to your project. To do this, just open fontello website and drag your `config.json` file to the webpage. And follow instructions on fontello website.

Your imported Svg files will appears in **Custom Icons** section of the webpage.

You can also edit css class and code for all icon font glyphs in your project.

### Update source SVG to config.json

Add the following code to your `gulpfile.js`:
``` javascript
var gulp = require('gulp');
var importer = require('gulp-fontello-import');

gulp.task('import-svg', function(cb) {
    importer.importSvg({
        config : 'config.json',
        svgsrc : 'svg-src'
    }, cb);
});
```
The importer api accept an option object that specified path to `config.json` and svg source folder. Run this task will update the `config.json` file you specified in the option.

### Download generated icon font

Add the following code to your `gulpfile.js`:

``` javascript

gulp.task('get-icon-font', ['import-svg'], function(cb) {
    importer.getFont({
        host           : 'http://fontello.com',
        config         : 'config.json',
        css : 'css',
        font : 'fonts'
    },cb);
});

gulp.task('get-example', ['import-svg'], function(cb) {
    importer.getFont({
        host : 'http://fontello.com',
        config: 'config.json'
    }, cb);
});
```

There are two kind of options you can pass into `importer.getFont` task. The one with `css` and `font` will output generated icon font file and css to the path you specified.

The one without those will download the whole fontello project to "icon-example" folder in your project. A demo html file is included in the fontello project so you can check if all icons work correctly before you put them in use in your project.
