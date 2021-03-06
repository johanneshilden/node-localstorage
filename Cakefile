fs            = require('fs')
{execSync, spawnSync} = require('child_process')

isWindows = (process.platform.lastIndexOf('win') == 0)

runSync = (command) ->  # !TODO: Upgrade to spawn a shell so we can run gsutil like https://github.com/cbarrick/gulp-run/blob/master/lib/command.js
  if isWindows
    shell = 'cmd.exe'
    args = ['/c', command]
  else
    shell = 'sh'
    args = ['-c', command]
  {status, stdout, stderr} = spawnSync(shell, args, {encoding: 'utf8'})
  if stderr?.length > 0 or status > 0
    console.error("Error running: '#{command}'\n#{stderr}\n")
    process.exit(status)
  else
    console.log("Output of running '#{command}'\n#{stdout}\n")
    return stdout

task('compile', 'Compile CoffeeScript source files to JavaScript', () ->
  process.chdir(__dirname)
  fs.readdir('./', (err, contents) ->
    files = ("#{file}" for file in contents when (file.indexOf('.coffee') > 0))
    command = ['coffee', '-c'].concat(files).join(' ')
    runSync(command)
  )
)

task('test', 'Run the CoffeeScript test suite with nodeunit', () ->
  {reporters} = require('nodeunit')
  process.chdir(__dirname)
  reporters.default.run(['test'], undefined, (failure) -> 
    if failure?
      console.log(failure)
      process.exit(1)
  )
)


task('publish', 'Publish to npm and add git tags', () ->
  process.chdir(__dirname)
  runSync('cake test')  # Doing this externally to make it synchronous
  process.chdir(__dirname)
  runSync('cake compile')
  console.log('checking git status --porcelain')
  stdout = runSync('git status --porcelain')
  if stdout.length > 0
    console.error('`git status --porcelain` was not clean. Not publishing.')
  else
    console.log('checking origin/master')
    stdout = runSync('git rev-parse origin/master')

    console.log('checking master')
    stdoutOrigin = stdout
    stdout = runSync('git rev-parse master')
    stdoutMaster = stdout

    if stdoutOrigin == stdoutMaster

      console.log('running npm publish')
      runSync('npm publish .')

      if fs.existsSync('npm-debug.log')
        console.error('`npm publish` failed. See npm-debug.log for details.')
      else

        console.log('creating git tag')
        runSync("git tag v#{require('./package.json').version}")
        runSync("git push --tags")
    else
      console.error('Origin and master out of sync. Not publishing.')
)


