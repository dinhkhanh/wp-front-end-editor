fs = require('fs')
path = require('path')
mkdirp = require('mkdirp').sync
{spawn, exec} = require('child_process')

io = (callback, inputPath, outputPath) ->
	mkdirp path.dirname outputPath

	input = fs.readFileSync inputPath, 'utf8'

	if input is ''
		console.warn "Empty file: #{inputPath}"

	callback input, (output) ->
		fs.writeFileSync outputPath, output, 'utf8'

compress_js = (input, cb) ->
	{parser, uglify} = require('uglify-js')

	ast = parser.parse input
	ast = uglify.ast_squeeze ast

	cb uglify.gen_code ast

launch = (cmd, options=[], callback) ->
	app = spawn cmd, options
	app.stdout.pipe(process.stdout)
	app.stderr.pipe(process.stderr)
	app.on 'exit', (status) -> callback?() if status is 0

coffee_invoke = (watch) ->
	options = ['-c', '-b', '-o', 'js', 'coffee']
	options.unshift '-w' if watch
	launch 'coffee', options

task 'watch', 'Watch coffee/ directory and compile into js/', (options) ->
	coffee_invoke true

task 'watch:coffee', 'Watch coffee/ directory and compile into js/ (using Jitter)', (options) ->
	launch 'jitter', ['coffee/', 'js/']

task 'dev:js', 'Generate separate JS files', (options) ->
	coffee_invoke false

task 'build:js', 'Generate compressed JS', (options) ->
	exec 'cd coffee; cat core.coffee hover.coffee init.coffee fields/*.coffee | coffee -cs > ../build/editor.js', (err, stdout, stderr) ->
		throw err if err

		io compress_js, 'build/editor.js', 'build/editor.min.js'

task 'build:aloha', 'Generate Aloha plugin(s)', (options) ->
	plugin = 'wpImage'

	dir = "aloha-plugins/#{plugin}/lib/"

	mkdirp dir

	exec "coffee -b $2 -o #{dir}/ coffee/aloha/#{plugin}-plugin", (err, stdout, stderr) ->
		throw err if err

task 'build', 'Generate a build for wp.org', (options) ->
	mkdirp 'build'
	invoke 'build:js'
	invoke 'build:aloha'
