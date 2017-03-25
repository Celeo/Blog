In the last post I set up the app, the DB model, and the Git integration through the GitPython library. In this post, I'll be looking at the templates that will render what the user sees.

---

## Base template

First up is the base template from which all other pages will extend. As I wrote before, I decided to use the Bootstrap CSS framework. I nearly always use jQuery, and I'll add in a custom CSS file as well. That all makes the <head>:

```language-html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="CodeWall">
    <meta name="author" content="Celeo">
    <title>{% if title %}{{ title }} | CodeWall{% else %}CodeWall{% endif %}</title>
    <link href="{{ url_for('static', filename='css/bootstrap.min.css') }}" rel="stylesheet">
    <link href="{{ url_for('static', filename='css/bootstrap-theme.min.css') }}" rel="stylesheet">
    <link href="{{ url_for('static', filename='css/style.css') }}" rel="stylesheet">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.2.0/jquery.min.js" defer></script>
    <script src="{{ url_for('static', filename='js/bootstrap.min.js') }}" defer></script>
    {% block head %}{% endblock head %}
</head>  
```

I'll add a simple navbar for navigating back to the index to the top of the page:

```language-html
<body role="document">  
    <nav class="navbar navbar-default navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a class="navbar-brand" href="{{ url_for('index') }}">CodeWall</a>
            </div>
            <div id="navbar" class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li {% if request.path == '/' %}class="active"{% endif %}><a href="{{ url_for('index') }}">Home</a></li>
                </ul>
            </div>
        </div>
    </nav>
```

We'll finish off with a block for the content of the page and another for end-of-body code:

```language-html
    <div class="container" role="main">
        {% block content %}{% endblock content %}
    </div>
    {% block bodyend %}{% endblock bodyend %}
</body>  
</html>  
```

## Index page

With a lot of the boilerplate tucked away in base.html, we're able to write pretty small HTML templates for the other pages. index.html is only 19 lines.

The goal of this page is very simple: show the user a list of the repositories in the database and let them click on the names to go browse.

The Bootstrap list-group helps here:

```language-html
{% extends 'base.html' %}

{% block content %}

<div class="panel panel-default">  
    <div class="panel-heading">Repositories</div>
    <ul class="list-group">
    {% for repo in repositories %}
        <li class="list-group-item"><a href="{{ url_for('repo', repo_name=repo.name) }}">{{ repo.name }}</a></li>
    {% else %}
    </ul>
    <div class="panel-body">
        <p>No repositories found</p>
    </div>
    {% endfor %}
    </ul>
</div>

{% endblock content %}
```

That for loop in the middle is the bulk of the HTML on the page (depending on how many repositories you have configured) - it lists all of the repositories, with links, as items in the list. The link leads to the repo backend method, passing the name of the repository (this is the Repository.name field from the DB model): `{{ url_for('repo', repo_name=repo.name) }}`.

Since the only variable that the index.html template needs is repositories, the backend method for index is only 3 lines:

```language-python
@app.route('/')
def index():  
    return render_template('index.html', repositories=Repository.query.filter_by(public=True).all())
```

In the last post, I noted how Repository.public controls whether or not the repository is shown on the index page. I'm not currently limiting access to public repositories; it's like an unlisted video on YouTube.

We'll follow the link here and look next at the repository page.

## Repository page

This is the largest page in the app, as it has to show the user the following:

* the code in the current path
* the commits
* the branches

In the current state, I'm only showing the branches, not actually doing anything with them, so we'll pass over that. The files and commits are larger tasks. Commits are a bit simpler, so we'll start there.

## Showing commits

If you thought that showing the commits would be as simple as a for loop in the template, you're half right. For younger repositories, the commit log may only be 20-40 commits long, which isn't too big to show on one page. For older repositories, though, the commit log is several hundred entries, which is too long to show at once. The solution is pagination, which GitPython already supports!

Back in article 1, I showed that the Repository DB model has a commit method:

```language-
def commits(self, branch='master', offset=0, per_page=20):  
    offset, per_page = int(offset), int(per_page)
    return list(Repo(self.path).iter_commits(branch, max_count=per_page, skip=offset * per_page))
```

Note that the method itself takes an offset parameter and the `git.Repo.iter_commits` method takes a `skip` parameter. I use the template-accessed offset to paginate the commits, 20 per page by default, shown in the `repo.html` template at once.

In the repo backend method, I use a GET parameter to keep track of this and pass to the template:

```language-python
commit_offset = request.args.get('offset', 0)  
```

##Showing files

The last thing (which is really the first thing) that the repo page has to do is show files. This is done through the DB model's files method (`def files(self, path='', commit='HEAD')`).

Here, path is what you'd expect: empty for top-level and `/src/com/app/templates` to lead down into that directory in the code. GitPython lets use browse files through `git_repo.tree(repo.commit(commit))[path]` (where `git_repo` is an instance of `git.Repo`).

I take the blob and tree objects that GitPython gives us and wrap them in the custom CodeFile class from article 1 to give them the properties is_directory, name, and path for use in displaying on the template.

So, taking that all together, calling `{{ repo.files(code_path) }}` gives a list of objects that can be shown (`{{ file.name }}`) to the user.

## The call to render_template

Here's the backend method. It gets `commit_offset` and `show_tab` from the GET parameters, and repo from the URL:

```language-python
@app.route('/repo/<repo_name>')
@app.route('/repo/<repo_name>/browse/<path:code_path>')
def repo(repo_name, code_path=''):  
    commit_offset = request.args.get('offset', 0)
    show_tab = request.args.get('showtab', 'code')
    repo = Repository.query.filter_by(name=repo_name).first()
    if not repo:
        abort(404)
    return render_template('repo.html', repo=repo, code_path=code_path, commit_offset=commit_offset, show_tab=show_tab)
```

## The template

That's a lot of data to present to the user, so the template will be a bit lengthy (136 lines). Instead of pasting it all here, I'll direct you to the file on Bitbucket and talk about specific parts.

To display the data, I'm using Bootstrap's nav-tabs to show the three tabs: Code, Commits, and Branches, which correspond with what I've written above.

To display the code, I make use of the repo.files call:

```language-html
<table class="table table-hover">  
    <thead>
        <tr>
            <th>Path</th>
            <th>Author</th>
            <th>Date</th>
        </tr>
    </thead>
    <tbody>
    {% for file in repo.files(code_path) %}
        <tr>
        {% if file.is_directory %}
            <td><a href="{{ url_for('repo', repo_name=repo.name, code_path=file.path) }}">{{ file.name }}</a></td>
        {% else %}
            <td><a href="{{ url_for('inspect', repo_name=repo.name, code_path=file.path) }}">{{ file.name }}</a></td>
        {% endif %}
            <td>{{ file.last_commit_mesage or "foo" }}</td>
            <td>{{ file.last_commit_date or "foo" }}</td>
        </tr>
    {% endfor %}
    </tbody>
</table>  
```

I have a for loop to iterate through the files in the current path and display them in the table. You'll see that, in the current state, the `last_commit_message` and `last_commit_date` fields of the CodeFile object don't yet exist - they'll come later.

Files that are directories lead to the same backend route, with their modified versions of the code_path. Files that aren't directories lead to the inspect page (next article).

Showing the commits is a bit simpler. Here's the pagination code at the top of the page:

```language-html
{% if repo.commit_count() > 20 %}
    <nav>
        <ul class="pagination">
        {% for index in (repo.commit_count() / 20)|int|commit_range %}
            {% if index == commit_offset|int + 1 %}
                <li class="active">
                    <a href="?offset={{ index - 1 }}&amp;showtab=commits">{{ commit_offset|int + 1}} <span class="sr-only">(current)</span></a>
                </li>
            {% else %}
                <li><a href="?offset={{ index - 1 }}&amp;showtab=commits">{{ index }}</a></li>
            {% endif %}
        {% endfor %}
        </ul>
    </nav>
{% endif %}
```

We only create the pagination section if the repository has more than 20 commits. Now, for displaying the commits:

```language-html
<table class="table table-hover">  
    <thead>
        <tr>
            <th>SHA</th>
            <th>Author</th>
            <th>Authored on</th>
            <th>Committer</th>
            <th>Committed on</th>
            <th>Summary</th>
        </tr>
    </thead>
    <tbody>
    {% for commit in repo.commits(offset=commit_offset) %}
        <tr>
            <td><a href="{{ url_for('commit', repo_name=repo.name, commit_sha=commit.hexsha) }}">{{ commit.hexsha|sha }}</a></td>
            <td>{{ commit.author }}</td>
            <td>{{ commit.authored_date|date }}</td>
            <td>{{ commit.committer }}</td>
            <td>{{ commit.committed_date|date }}</td>
            <td>{{ commit.summary }}</td>
        </tr>
    {% endfor %}
    </tbody>
</table>  
```

Another table! This table only has 1 type of link: to the commit page, passing the name of the repository (to fetch the DB model from the database) and the commit's SHA, which is also displayed on the page to the user, filtered to show the first 7 characters:

```language-python
@app.template_filter('sha')
def filter_sha(s):  
    return s[:7]
```

Finally, the branches:

```language-html
<table class="table table-hover">  
    <thead>
        <tr>
            <th>Name</th>
        </tr>
    </thead>
    <tbody>
    {% for branch in repo.branches() %}
        <tr>
            <td>{{ branch.name }}</td>
        </tr>
    {% endfor %}
    </tbody>
</table>  
```

As noted previously, I'm not currently doing anything more with branches.

Check the link to the repo page source for the full page, but here's the bulk of the rendering.

Next, the inspect and commit pages.