Now here's a project that I've wanted to do for a while. Finally, some progress!

---

Making a web frontend to a Git repo has been on the TODO list for a while. Git is my source control of choice, and I've used a handful of different websites to display my code: GitHub, Bitbucket, GitLab, and several others I've forgotten. In Windows, I use SourceTree, a GUI application to manage Git repositories that really works well. In Linux, I'm comfortable enough with (or patient enough to learn new tricks in) the CLI.

## First steps

The first step was finding a library that would let me interface with a local Git repository. I didn't need to change anything, but I needed full read access to the repo. My first though was to just fork out the commands to the Git CLI, but this wouldn't allow for more than 1 person to view the repo on the web at once - what if a user switched branches?

So, onward to a search that, per usual, landed me on Stack Overflow. I tried a few different libraries before settling on GitPython (readthedocs, source). It seemed to have what I wanted: the ability to view the contents of the repo without changing or relying upon the current state of the repo on the drive.

I installed it and opened the Python interpreter and got to work. The second step of this project was to identify the common data I'd need from the library and the code to get those data.

I knew that I'd need the following from the start:

* Get branches
* Get a commit log
* Get commit information (author, date, message)

Of course, these weren't enough for the full web app, but it was enough to start.

## Setting up the app

I'm using Flask for the backend framework, the included Jinja for the template renderer, Bootstrap 3 for the CSS framework (I'm no web designer, I'll take any help I can get to make the page look presentable), and SQLAlchemy for my database ORM connected to a little SQLite database (I'm only keeping track of the names of and paths to the repositories presented on the app).

That DB model is the following:

```language-python
class Repository(db.Model):  
    __tablename__ = 'codewall_repository'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String, unique=True)
    path = db.Column(db.String, unique=True)
    public = db.Column(db.Boolean, default=True)

    def __init__(self, name, path, public=True):
        self.name = name
        self.path = path
        self.public = public
```

It's pretty simple, no surprises here. The name column stores the name of the repository and the path column stores the path to the physical files on the drive. Finally, the public column is a "should this repository be shown on the index page" toggle.

## Git integration

The GitPython library allows us to create a Repo object by passing its constructor that path. So, let's add that to the model:

```language-python
from git import Repo

@property
def git_repo(self):  
    git_repo = Repo(self.path)
```

This is good for processing from the backend route, but not the template, so, going off of what I noted earlier, I added some helper methods that allow the templates to get the data that they need from the model that's passed to render_template:

```language-python
    def files(self, path='', commit='HEAD'):
        repo = Repo(self.path)
        if not path:
            items = repo.tree(repo.commit(commit)).traverse(depth=1)
        else:
            path_item = repo.tree(repo.commit(commit))[path]
            if path_item.type == 'blob':
                return []
            items = list(path_item.traverse(depth=1))
        return sorted([CodeFile(item) for item in items], key=lambda x: not x.is_directory)

    def branches(self):
        return Repo(self.path).branches

    def commits(self, branch='master', offset=0, per_page=20):
        offset, per_page = int(offset), int(per_page)
        print('Getting commits for repo {}, offset={}, per_page={}'.format(self.name, offset, per_page))
        return list(Repo(self.path).iter_commits(branch, max_count=per_page, skip=offset * per_page))

    def commit_count(self, branch='master'):
        return len(list(Repo(self.path).iter_commits(branch)))


class CodeFile:

    def __init__(self, item):
        self.item = item

    @property
    def is_directory(self):
        return self.item.type == 'tree'

    @property
    def name(self):
        if self.item.type == 'tree':
            return '%s/' % self.item.name
        return self.item.name

    @property
    def path(self):
        return self.item.path
```

The files method allows the template to get all the files under a path (whether top-level or otherwise) for a specific commit. In practice, this is used to navigate the repository. branches returns the branches, commits, lists Commit objects for the navigating through the commits, and commit_count returns the total number of commits in the repository. The CodeFile class is a wrapper around the Blob and Tree objects for ease of use in the templates.

That's enough to navigate through the files and commits in the repository, so let's move to the templates.