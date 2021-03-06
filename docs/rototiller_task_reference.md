# Rototiller Task Reference
* herein lies the reference to the rototiller_task API

* [#rototiller_task](#rototiller_task)
  * [#add_command](#rototiller_task:add_command)
  * [#add_env](#rototiller_task:add_env)
* [Command](#Command)
  * [#add\_env](#Command:add_env)
  * [#add\_switch](#Command:add_switch)

<a name="rototiller_task"></a>
## #rototiller_task
* behaves just like any other rake task name (see below)

<a name="rototiller_task:add_command"></a>
### #add_command
* adds a command to a rototiller_task. This command can in turn contain environment variables, switches, options and arguments
* this command (and any others) will be run with the task is executed

<a name="rototiller_task:add_env"></a>
### #add_env
* adds an arbitrary environment variable for use in the task
* if specified with a default value, and the system environment does not have this variable set, rototiller will set it, for later use in a task or otherwise
* FIXME: add a bunch more examples with messaging and default values

&nbsp;

    require 'rototiller'

    desc "parent task for dependencies test. this one also uses an environment variable"
    rototiller_task :parent_task do |task|
      # most method initializers take either a hash, or block syntax (see next task)
      task.add_env({:name     => 'RANDOM_VAR', :default => 'default value'})
      task.add_env({:name     => 'RANDOM_VAR2', :default => 'default value2'})
      task.add_command({:name => "echo 'i am testing everything with $RANDOM_VAR = #{ENV['RANDOM_VAR']}'"})
      task.add_command({:name => "echo 'some other command!'"})
    end

produces:

    # added environment variable defaults are set, implicitly, if not found
    #   this way, their value can be used in the task
    # FIXME... this is a bug?  i think it's supposed to set vars with default values even at task level??
    $) rake parent_task
    INFO: The environment variable: 'RANDOM_VAR' is not set. Proceeding with default value: 'default value':
    INFO: The environment variable: 'RANDOM_VAR2' is not set. Proceeding with default value: 'default value2':
    i am running this command with $RANDOM_VAR =
    some other command!

    $) rake parent_task RANDOM_VAR=redrum
    INFO: The environment variable: 'RANDOM_VAR' was found with value: 'redrum':
    INFO: The environment variable: 'RANDOM_VAR2' is not set. Proceeding with default value: 'default value2':
    i am running this command with $RANDOM_VAR = redrum
    some other command!
&nbsp;

<a name="Command"></a>
## Command
<a name="Command:add_env"></a>
### #add_env
* adds an arbitrary environment variable which overrides the name of the command
* if specified with a default value, and the system environment does not have this variable set, rototiller will set it, for later use in a task or otherwise
* FIXME: add a bunch more examples with messaging and default values

&nbsp;

    desc "override a command-name with environment variable"
    rototiller_task :child => :parent_task do |task|
      task.add_command({:name => 'nonesuch', :add_env => {:name => 'COMMAND_EXE1'}})
      # block syntax here. We give up some lines for more readability
      task.add_command do |cmd|
        cmd.name = 'meneither'
        cmd.add_env({:name => 'COMMAND_EXE2'})
      end
    end

produces:

    # we didn't override the command with its env_var, so shell complains about nonsuch and exits
    $ rake child RANDOM_VAR=redrum
    INFO: The environment variable: 'RANDOM_VAR' was found with value: 'redrum':
    INFO: The environment variable: 'RANDOM_VAR2' is not set. Proceeding with default value: 'default value2':
    i am running this command with $RANDOM_VAR = redrum
    some other command!
    sh: nonesuch: command not found

    # now we've overridden the first command to echo partial success
    #  but the next command was not overridden by its environment variable, which has no default
    $ rake child COMMAND_EXE1='echo partial success'
    INFO: The environment variable: 'RANDOM_VAR' is not set. Proceeding with default value: 'default value':
    INFO: The environment variable: 'RANDOM_VAR2' is not set. Proceeding with default value: 'default value2':
    i am running this command with $RANDOM_VAR =
    some other command!
    partial success
    sh: meneither: command not found

    # NOW our silly example works!
    $ rake child COMMAND_EXE1='echo partial success' COMMAND_EXE2='echo awwww yeah!'
    INFO: The environment variable: 'RANDOM_VAR' is not set. Proceeding with default value: 'default value':
    INFO: The environment variable: 'RANDOM_VAR2' is not set. Proceeding with default value: 'default value2':
    i am running this command with $RANDOM_VAR =
    some other command!
    partial success
    awwww yeah!

<a name="Command:add_switch"></a>
### #add_switch
* adds an arbitrary string to a command
  * intended to add `--switch` type binary switches that do not take arguments (see [add_option](#Command:add_option))

&nbsp;

    desc "override a command-switch with environment variable"
    rototiller_task :variable_switch do |task|
      task.add_command do |cmd|
        cmd.name = 'echo'
        cmd.add_switch do |s|
         s.name = '--switch'
         s.add_env({:name => 'CRASH_OVERRIDE'})
      end
    end

produces:

    $ rake --rakefile docs/Rakefile.example variable_switch
    --switch

    $ rake --rakefile docs/Rakefile.example variable_switch --verbose
    echo --switch
    --switch

    $ rake --rakefile docs/Rakefile.example variable_switch --verbose CRASH_OVERRIDE='and burn'
    echo and burn
    and burn
