Currently, whenever I wanted to update the repositories stored on the server serving the web app, I had to SSH into the server and pull from the Bitbucket repository. It would be nice if I could push to the server as a remote instead.

---

## Gitolite

To integrate a Git server with the web app, I've selected Gitolte. The installation instructions in the repo were enough to get started and are easy to understand so I won't repeat them all here, but there are two things I wanted to note.

 #1. Making the SSH key in a file not `~/.ssh/id_rsa`.

On the machine I'm working from, I already have an SSH key that's in the `/home/name/.ssh/id_rsa` slot, so I made my key in a separate file, `/home/name/.ssh/gitserver`. This is perfectly acceptable, of course, but then I couldn't just call git push and git pull to the Gitolite repositories without trying to specify the SSH key to use.

The solution here was the SSH configuration file at /home/name/.ssh/config. I added this entry:

```
Host gitserver
    Hostname example.com
    IdentityFile ~/.ssh/gitserver
    User git
```

So, when I went to grab the gitolite-admin repo from the server to my machine, I used the command git clone git@gitserver:gitolite-admin. This prompted me for the passphrase to the SSH key, and then I was up and running.

 #2. Different umask for the repositories

By default, Gitolite sets the file permissions for the repositories it holds to 700. This is perfectly fine unless you have, say, a web app trying to reach that path. To change that, SSH to your server, change into the user under which you installed Gitolite, and edit the configuration file (vim ~/ssh/.gitolite.rc). Find the UMASK line. The default, as VonC points out on Stack Overflow, is 0077, which gives the files that 700 permission. According to the Gitolite documentation for the configuration file, the correct way to handle navigating into the repositories from any other user is to change the umask to 0027 and add the desired user to the files' permissions group. Do so. You may have to stop your app and log in and out of that user on the server to get the group change to take effect.

Continue on with installation as documented, then push one of your repositories to Gitolite. For me, because of how I configured my SSH config, that was `git remote add gitserver git@gitserver:repo-name` followed by `git push gitserver master -u`.

## App updates

The only thing, if installation went well, we have to do for Codewall is update the paths of the repositories that we're showing. You can do this through your database's GUI/CLI tools, or through SQLAlchemy:

```language-python
from codewall.app import app, db  
from codewall.models import Repository

for repo in Repository.query.all():  
    repo.path = '/path/to/gitolite_installation/%s.git' % repo.name
db.session.commit()  
```

Exit the shell and navigate to your server. Attempt to browse a repository. If all went well, you should be browsing Gitolite's copy of the repository.