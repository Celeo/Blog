Better than Paramiko for what I needed.

---

> "Fabric is a Python (2.5-2.7) library and command-line tool for streamlining the use of SSH for application deployment or systems administration tasks."

* [GitHub link](https://github.com/fabric/fabric)
* [PyPi link](https://pypi.python.org/pypi/Fabric)
* [Homepage](http://www.fabfile.org/)
* [Docs](http://docs.fabfile.org/en/1.11/)

A while back, I wrote about using Paramiko to control several devices through SSH and SCP. Unfortunately, the code I used had some shortcomings, most notably I had no way to determine if a command had finished. I was using some low-level communication methods to persist a login to the root user account.

There's a better way to do that, and it's with Fabric.

## Installation
First off, installation (as the homepage notes, Fabric runs on 2.5-2.7, not 3.X):

```language-
$ pip install fabric
```

You may need a supporting library in order to install PyCrypto. For Debian (specifically, Linux Mint 17.2) I needed python-dev:

```language-bash
$ sudo apt-get update
$ sudo apt-get install python-dev
```

## Making the connection
Fabric is much more than a simple tool to execute SSH commands to a target, but that's what I needed it for, so that's what we'll take a look at.

First, imports:

```language-python
from fabric.api import env, run as _run, sudo as _sudo  
from fabric.tasks import execute  
from fabric.network import disconnect_all  
from fabric.state import connections  
I imported the run and sudo methods as _run and _sudo, respectively, because I'm going to wrap them to give them some default parameters.

def run(command, **kwargs):  
    kwargs['quiet'] = kwargs.get('quiet', True)
    kwargs['combine_stderr'] = kwargs.get('combine_stderr', False)
    kwargs['pty'] = kwargs.get('pty', False)
    return _run(command, **kwargs)


def sudo(command, **kwargs):  
    kwargs['quiet'] = kwargs.get('quiet', True)
    kwargs['combine_stderr'] = kwargs.get('combine_stderr', False)
    return _sudo(command, **kwargs)
```

I've found this combination of arguments to the run and sudo methods to produce the cleanest output on the systems that I'm testing it on. With these wrappers, I can call run('df -h') without having to pass the other arguments that I usually want, though, if I wanted to, I could overwrite any of them.

Next, configuration.

Fabric uses an "environment dictionary" called `env` to hold the configuration, which we imported from `fabric.api` above. We need to add a few things to the `env` variable:

```language-python
env.hosts = ['10.10.7.100', '10.10.7.101', '10.10.7.102']  
env.user = 'admin'  
env.key_filename = 'id_rsa'  
For this example, I'm using key-based authentication for 3 target systems. The SSH login uses the username 'admin'. That's it for configuration! When we call our method to actually go and perform actions on the target systems, Fabric will perform all the instructions for the first system at 10.10.7.100, then the next at 10.10.7.101, and so on.
```

## Running commands
The syntax for executing commands is extremely simple since we've wrapped those methods with our default arguments:

```language-python
run('command')  
sudo('root command')  
sudo('command as other user', user='otheruser')  
```

That's it. When we first run a superuser command, assuming that the user we're logged into, 'admin', requires a password, Fabric will prompt the user to enter the password from the command line (and it'll hide it as the user types, just like entering the password locally).

What about running commands locally? We could import `subprocess` and execute from there, but Fabric has a wrapper for us to use:

```language-python
local('command', capture=False, shell=None)  
```

You should normally be fine omitting shell, as it defaults to '/bin/sh', unless you need bash. I set capture to True, as I'd like to capture the output from the command and display it if needed instead of it immediately being written to the console, potentially adding clutter.

## SCP
Fabric uses Paramiko, which means that we can too! Remember this from the Paramiko post?

```language-python
import scp  
with scp.SCPClient(ssh.get_transport()) as _scp:  
    _scp.get('/path/to/filename')
```

With a few modifications, we can still use this module to perform our SCP gets and puts:

```language-python
with support_scp.SCPClient(connections['%s@%s:22' % (env.user, env.host_string)].get_transport()) as _scp:  
        _scp.get('/path/to/filename')
```

Here, we don't have access to a `SSHClient` from Paramiko directly, so we use Fabric's connections variable to access those lower-level variables. connections is (effectively) a dictionary, so we can access the connection variables with the currently-targeted IP address as the key, which we get from `env.host_string`.

Fabric has built-in methods to get and put files through the existing connection, but the transport method is SFTP. I needed SCP, so I used this module. If you can work with SFTP, then here's the link to the documentation for those methods.

## Review
It seems like I didn't look hard enough when I went searching for an SSH library to control devices in Python originally. Since Fabric uses Paramiko, it wasn't a loss, though. I was able to apply what I learned with Paramiko to Fabric, as I couldn't use Fabric's SFTP protocol to move files - I had to go down a level and use Paramiko with the SCP library I found previously.

Regardless of how I got here, Fabric is definitely the better tool for this job.