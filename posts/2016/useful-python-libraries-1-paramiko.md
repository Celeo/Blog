Paramiko is an SSH library.

----

**Update**: As I reviewed the code that I had written and its shortcomings, I found other people using Fabric instead of Paramiko for this high-level remote control. I'll write up a post on Fabric soon.

* [GitHub link](https://github.com/paramiko/paramiko)
* [PyPi link](https://pypi.python.org/pypi/paramiko/)
* [Docs]()

> Paramiko is a module for Python 2.6+ that implements the SSH2 protocol for secure (encrypted and authenticated) connections to remote machines.

-source: readme

For me, that means a scriptable method to communicate with a networked device running Linux with more control than using a bash script.

## Installation
First off, installation (you are using some sort of virtual environment, right?):

```language-bash
$ pip install paramiko
```

You may need a supporting library in order to install PyCrypto. For Debian (specifically, Linux Mint 17.2) I needed python-dev:

```language-bash
$ sudo apt-get update
$ sudo apt-get install python-dev
```

## Making the connection
Now that we have the basics setup, we can make the connection. We need an IP address, port (default for SSH is 22), username, and password. Paramiko does support using a SSH key file, but I don't need that functionality. So, here is the basic code:

```language-python
import paramiko
ssh = paramiko.SSHClient()  
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())  
```

The first line is the import - no fuss. Next, we create an instance of the `SSHClient` class. Then, we set the connection to ignore unknown hosts. If this code were going to a client, I'd make sure to properly handle known hosts, but for code connecting to many systems in a development environment, it's fine.

Now, the connection call:

```language-python
ssh.connect(ip_address, port=port, username=username, password=password)
```

You can simply ask the user for the IP address:

```language-python
ip_address = raw_input('Enter the IP address to connect to: ')  
```

For the password, you may want to have it entered like passwords are in the shell where the keys are not echoed to the terminal as the user types. This can be done throuh the getpass module:

```language-python
from getpass import getpass
password = getpass('Enter the password: ')
```

Now that we have the connection, we'll open an interactive shell with invoke_shell:

```language-python
shell = ssh.invoke_shell()  
```

We'll use this to write commands to the device and read the responses.

## Sending commands

```language-python
from time import sleep  
shell.send(command + '\n')  
sleep(1)  
buff = ''  
while shell.recv_ready():  
    buff += shell.recv(1024)
print buff  
```

Let's break that down.

First, we'll import the sleep function from the time module. Quick note: unlike some languages, like Java and C#, Python's sleep takes the time to wait in seconds, not milliseconds.

Then, we send the command down through the connection with an appended newline character to simulate the user pressing Enter. We sleep for a second to allow processing (you may want to consider making the sleep duration dependent on your command; you'll have to play with it to see what you need) and read the response from the connection. If you make this code into a separate method (recommended), return the buff variable. Otherwise, here we'll just print it.

## Authenticating as the root user
The device I am connecting to doesn't allow root login, so we login to a user with the ability to use sudo. sudo requires that users's password, though, so we have to send the password immediately after the sudo command. Additionally, I don't want to store that password in the script, so we'll use the previously-entered password the user entered to start the connection.

Let's assume we put the previous section's code into its own method:

```language-python
def command(command):  
    buff = ''
    while shell.recv_ready():
        buff += shell.recv(1024)
    shell.send(command + '\n')
    sleep(1)
    while shell.recv_ready():
        buff += shell.recv(1024)
    return buff
```

To become the root user on the device I'm using, the command is `$ sudo su`. Through paramiko, we'll do:

```language-python
command('sudo su')  
command(password) 
```
 
This sends the command through the connection, immediately followed by the password required to confirm the command.

## SCP
Well, we have the ability to connect, send commands, and authenticate as the root user so far. What about moving files?

We'll use scp to move files, one at a time, from the remote device to our current machine. You may have used a command-line utility of the same name previously.

Paramiko doesn't come with its own scp capabilties, but there's a module that adds this feature, appropriately called scp:

> The scp.py module uses a paramiko transport to send and recieve files via the scp1 protocol.

-source: readme

It's a single file, so "installation" can be just grabbing the file from the repository and having it in your working directory:

```language-bash
$ wget https://github.com/jbardin/scp.py/blob/master/scp.py
```

To use it, first import:

```language-python
import scp  
```

We'll instantiate a `SCPClient` class using with:

```language-python
with scp.SCPClient(ssh.get_transport()) as _scp:  
```

Inside that block, we can use _scp to get file from the device and put files onto the device. Those functions, respectively, are

```language-python
_scp.get('/path/to/filename')  
_scp.put('/path/to/local/filename', '/path/to/remote/target')  
```

## Review
Using Paramiko, we made a connection to a device, setup how to send commands, authenticated as the root user, and got an add-on module to handle file transfer. Adding that together yields fully scriptable SSH connections to one or more devices.
