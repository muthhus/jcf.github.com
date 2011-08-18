sys    = require('sys')
colors = require('colors')
util   = require('util')
spawn  = require('child_process').spawn

Process = (command, args) ->
  stdout = (data) -> process.stdout.write(data)
  stderr = (data) -> process.stdout.write(data)

  ps = spawn(command, args)
  ps.stdout.on 'data', stdout
  ps.stderr.on 'data', stderr

  ps

task 'build', 'continuously build the blog', ->
  new Process('jekyll', ['--auto', '--server'])
  new Process('compass', ['watch', '_compass'])
