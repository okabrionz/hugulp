# hugulp 2

## v2 Breaking changes

If you're using hugulp v1, please take note of the following changes:

* hugo is no longer invoked by hugulp

  use hugo as per its docs, then invoke hugulp build

* put your assets in the static folder

  for example, static/styles, static/images, static/scripts

* themes are supported out of the box

_Note: If you need sass/less/js pre-processing, read v2 docs below_

## Description

`hugulp` is a tool to optimize the assets of a [Hugo](https://gohugo.io) website.

The main idea is to recreate the famous [Ruby on Rails Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html), which minifies, concatenates and fingerprints the assets used in your website.

This leads to less and smaller network requests to your page, improving overall user experience.

Read [this blog post](https://jbrio.net/posts/mobile-friendly-website-2/) and [this article](https://medium.com/@juanbrodriguez/hugulp-a-hugo-gulp-toolchain-94f72ccc3577) for additional context.

_Note: These articles refer to v1_

It's internally driven by [Gulp](https://gulpjs.com).

This project Includes the following tools, tasks and workflows:

* [sass](https://github.com/dlmanning/gulp-sass) (super fast libsass)
* [less](https://github.com/plus3network/gulp-less)
* [autoprefixer](https://github.com/sindresorhus/gulp-autoprefixer)
* [clean-css](https://github.com/scniro/gulp-clean-css)
* [jshint](https://github.com/spalger/gulp-jshint)
* [uglify](https://github.com/terinjokes/gulp-uglify)
* [imagemin](https://github.com/sindresorhus/gulp-imagemin) (only [changed images](https://github.com/sindresorhus/gulp-changed))
* [gulp-rev](https://github.com/sindresorhus/gulp-rev), [gulp-rev-replace](https://github.com/jamesknelson/gulp-rev-replace) (fingerprinting)
* [htmlmin](https://github.com/jonschlinkert/gulp-htmlmin)
* [watch](https://github.com/floatdrop/gulp-watch)

## Installation

[Node](https://nodejs.org) needs to be installed in your system.

Then just

```bash
$ npm install -g hugulp
```

Or you can build and run using [docker](https://www.docker.com):

```bash
# Default docker setup:
$ ./scripts/create-docker-machine-and-run-it

# -- OR --

# Run with custom machine name, specific hugo version, specific node version and run docker in detached mode:
$ ./scripts/create-docker-machine-and-run-it -a app-devel -g 0.20.6 -n 6.10.0 -d
```

**Note:** You only run the `./scripts/create-docker-machine-and-run-it` if you want to create a new docker machine. Once the docker machine is created, you have to use docker commands to manage it. Please be familiar with docker in this regard.

## Getting Started

The most common usage scenario would be:

```bash
$ hugo new site yoursite
$ cd yoursite
$ hugulp init
# create content
# add images (static/images), css (static/styles) and javascript (static/scripts)
$ hugo server -D # for development
# development is done, ready to publish
$ rm -rf public # clean up public folder, it will be re-generated by hugo
$ hugo # for release/production/deployment
$ hugulp build # optimize the site by running the asset pipeline
```

Another scenario would be to include sass/less pre-processing:

```bash
$ hugo new site yoursite
$ cd yoursite
$ hugulp init
# create content
# add images (static/images), and javascript (static/scripts)
# add sass/less (assets/styles)
$ hugo server -D # for development
$ hugulp watch # to convert sass/less (assets/styles) into css (static/styles)
# development is done, ready to publish
$ rm -rf public # clean up public folder, it will be re-generated by hugo
$ hugo # for release/production/deployment
$ hugulp build # optimize the site by running the asset pipeline
```

In both cases, you could chain the last 3 commands:

```bash
$ rm -rf public && hugo && hugulp build
```

`hugulp` requires a configuration file (`.hugulprc`), which is created by the `hugulp init` command (you can create the file manually if you want).

This is the default `.hugulprc`:

```json
{
  "version": 1,
  "pipeline": ["images", "styles", "scripts", "fingerprint", "html"],
  "path": {
    "styles": "styles",
    "images": "images",
    "scripts": "scripts"
  },
  "watch": {
    "source": "assets",
    "target": "static"
  },
  "build": {
    "source": "public",
    "target": "public"
  },
  "autoprefixer": {
    "browsers": ["last 2 versions"]
  },
  "cleancss": {
    "advanced": false
  },
  "htmlmin": {
    "collapsedWhitespace": true,
    "removeEmptyElements": true
  },
  "gifsicle": { "interlaced": true },
  "jpegtran": { "progressive": true },
  "optipng": { "optimizationLevel": 5 },
  "svgo": {
    "plugins": [{ "removeViewBox": true }, { "cleanupIDs": false }]
  }
}
```

You can easily customize `hugulp`'s behavior, by modifying this configuration file, as described below.

## Available Commands

### hugulp watch

`hugulp` will assist you if you're using sass/less (and javascript), which require pre-processing.

It will watch for changes to styles or script files, process them and write them to hugo's static folder, according to the following table

| In Folder      |      Looks for       | Operation                              | Written to     |
| -------------- | :------------------: | -------------------------------------- | -------------- |
| assets/styles  | s[a\|c]ss, less, css | Convert sass/less to css               | static/styles  |
| assets/scripts |          js          | Lint javascript code (_soon babelify_) | static/scripts |

_Note: It searches the folders recursively_

The table above applies to `hugulp` run with a default `.hugulprc`.

You can customize the folder names: `resources` instead of `assets`, `js` instead of `scripts` and so on.

This is described in the `.hugulprc` section below.

### hugulp build

It optimizes the site that hugo built, by running the asset pipeline as defined in `.hugulprc` (field _pipeline_).

Additionally, files are not watched for changes

### hugulp version

Display installed version.

### hugulp init

Create a default `.hugulprc`.

## Configuration

By editing the `.hugulprc` configuration file, you can customize almost anything about `hugulp`.

Description of each field follows:

### pipeline

Defines which tasks of the asset pipeline will be executed (`hugulp build` command)

Type: array <br>
Default:

```json
"pipeline": ["images", "styles", "scripts", "fingerprint", "html"]
```

| Task        | Description                                                        |
| ----------- | ------------------------------------------------------------------ |
| images      | minify images with `imagemin`                                      |
| styles      | pre-process `sass`/`less`/css, then `clean-css`                    |
| scripts     | `jshint`, then `uglify`                                            |
| fingerprint | fingerprint with `rev`, then replace references with `rev-replace` |
| html        | minify html with `htmlmin`                                         |

Let's say you don't want to fingerprint the assets. Just set _pipeline_ to

```json
"pipeline": ["images", "styles", "scripts", "html"]
```

By removing the _fingerprint_ task, it will not be executed.

Note that tasks are executed sequentially.

### path

Defines the name of the folders where your assets are located/will be transferred to.

Type: object <br>
Default:

```json
"path": {
  "styles": "styles",
  "images": "images",
  "scripts": "scripts"
}
```

So if you prefer your styles folder to be called css, and scripts to be called js, you would change it to:

```json
"path": {
  "styles": "css",
  "images": "images",
  "scripts": "js"
}
```

### watch

Define which folders to watch for changes, for the `hugulp watch` command.

Type: object <br>
Default:

```json
"watch": {
  "source": "assets",
  "target": "static"
}
```

This field works together with the path field.

With a default `.hugulprc`, it will watch `assets/styles` and `assets/scripts` (recursively).

If you customized `path` as per above, it will watch `assets/css` and `assets/js`.

If you additionally want the `assets` folder to be called `resources`, change _source_ to `resources`

```json
"watch": {
  "source": "resources",
  "target": "static"
}
```

then it will watch `resources/css` and `resources/js`

Finally, the changes will be written to the well-known hugo static folder.

With a default `.hugulprc`, files will be written to `static/styles` and `static/scripts`.

### build

Defines the folders referenced during the `hugulp build` command

Type: object <br>
Default:

```json
build: {
  source: 'public',
  target: 'public'
}
```

This should generally be left unchanged.

`hugo` will output to the public folder by default, so `hugulp build` will process the files in-place.

### autoprefixer

Options for `autoprefixer`. Check [gulp-autoprefixer](https://github.com/sindresorhus/gulp-autoprefixer) for documentation.

Task: `styles`

Type: object<br>
Default:

```json
autoprefixer: {
  browsers: ['last 2 versions']
}
```

### cleancss

Options for `clean-css`. Check [gulp-clean-css](https://github.com/scniro/gulp-clean-css) for documentation.

Task: `styles`

Default:

```json
cleancss: {
  advanced: false
}
```

### htmlmin

Options for `htmlmin`. Check [gulp-htmlmin](https://github.com/jonschlinkert/gulp-htmlmin) for documentation.

Task: `html`

Default:

```json
htmlmin: {
  collapsedWhitespace: true,
  removeEmptyElements: true
}
```

## How to update

Whenever a new `hugulp` version becomes available, you can update it by running

```bash
$ npm update -g hugulp
```

## PR

Pull Requests are welcome :thumbsup:.

## Share

Made by [Juan B. Rodriguez](http://jbrodriguez.io), with a MIT License.

Please [share the article or leave your comments](http://jbrodriguez.io/mobile-friendly-website-2/).
