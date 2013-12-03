fs        = require 'fs'
path      = require 'path'
{compile} = require 'coffee-script'
less      = require 'less'
uglify    = require 'uglify-js'
cssmin  = require 'cssmin'
jsdom     = require 'jsdom'
autoprefixer = require 'autoprefixer'

option '-t', '--themes [NAME]', 'theme for compiled chocolate code'
option '-b', '--basedir [DIR]', 'directory with css, js, image folders'

task 'build', 'Build chocolate.js', (options) ->
  theme   = options.themes  or 'default'
  basedir = options.basedir or "/dist/#{theme}"
  current = path.dirname __filename

  dist = path.normalize "#{current}/dist/#{theme}"
  src  = path.normalize "#{current}/themes/#{theme}"

  fs.stat dist, (error, stat) ->
    if stat
      results = fs.readdirSync dist

      for folder in results
        folder = path.normalize "#{dist}/#{folder}"
        files  = fs.readdirSync folder
        if files and files.length > 0
          fs.unlinkSync path.normalize "#{folder}/#{file}" for file in files
        fs.rmdirSync folder

      fs.rmdirSync path.normalize dist

    fs.mkdirSync dist
    fs.mkdirSync "#{dist}/js"
    fs.mkdirSync "#{dist}/css"

    if fs.statSync "#{src}/images"
      fs.mkdirSync "#{dist}/images"
      images = fs.readdirSync "#{src}/images"
      for image in images
        from = fs.createReadStream path.normalize "#{src}/images/#{image}"
        to   = fs.createWriteStream path.normalize "#{dist}/images/#{image}"

        from.pipe to

    compileCssContent dist, src, basedir
    compileJsContent dist, src, basedir

compileTemplate = (src, basedir, fn) ->
  html = fs.readFileSync "#{src}/templates.html", 'utf8'
  jsdom.env html, [], (error, window) ->
    templates = {}

    [].forEach.call window.document.querySelectorAll('script'), (script) ->
      key = script.getAttribute 'id'

      if key
        html = script.innerHTML
        html = html.replace /\{\{basedir\}\}/gi, basedir

        lines = html.split "\n"
        html  = ''
        for line in lines
          line = line.trim()
          html += line

        templates[key] = html.trim()

    window.close()
    fn templates

compileJsContent = (dist, src, basedir) ->
  current = path.dirname __filename
  sources = [
    path.normalize "#{current}/src/utils.coffee"
    path.normalize "#{current}/src/storage.coffee"
    path.normalize "#{current}/src/chocolate.coffee"
    path.normalize "#{current}/src/touch.coffee"
  ]

  options = "defaultOptions = `" + fs.readFileSync("#{src}/options.json", 'utf8') + "`"

  compileTemplate src, basedir, (templates) ->
    templates = "templates = `" + JSON.stringify(templates) + "`"

    chocolate = ''
    for source in sources
      chocolate += fs.readFileSync source, 'utf8'
      chocolate += '\n'

    js = compile "#{options}\n\n#{templates}\n\n#{chocolate}", bare: true
    js = "(function(window, document) {\n\n#{js}\n\n})(window, document);"

    fs.writeFileSync path.normalize("#{dist}/js/chocolate.js"), js

    source = uglify.minify js, fromString: true
    source = source.code

    fs.writeFileSync path.normalize("#{dist}/js/chocolate.min.js"), source

    console.log 'done'

compileCssContent = (dist, src, basedir) ->
  parser = new less.Parser
    paths:    ["#{src}/css"]
    filename: 'chocolate.less'

  source = path.normalize "#{src}/css/chocolate.less"

  css = fs.readFileSync source, 'utf8'
  css = css.replace '{{basedir}}', basedir

  parser.parse css, (error, tree) ->
    css = tree.toCSS()

    css = autoprefixer.compile css

    fs.writeFileSync path.normalize("#{dist}/css/chocolate.css"), css
    fs.writeFileSync path.normalize("#{dist}/css/chocolate.min.css"), cssmin css
