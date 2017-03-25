In this article, I'll be going over the commit and inspect pages. The commit page is responsible for showing the changes made in a commit, and the inspect page is responsible for showing the contents of a file.

---

Keep in mind that this is a very simple web app by design - I wanted to get it stable, then write about it, then update it, then write about the updates.

## Commit page

#### Backend method

This time, I'll start with the backend method as it's not as simple as the one for browsing the repository. The method decoration and definition is:

```language-python
@app.route('/repo/<repo_name>/<commit_sha>')
def commit(repo_name, commit_sha):  
```

Coming in, I get the name of the repository so I can get the DB model Repository, and the SHA of the commit to get the commit object from GitPython.

First, I'll check the `repo_name` argument:

```language-python
repo = Repository.query.filter_by(name=repo_name).first()  
if not repo:  
    abort(404)
```

Pretty simple, and pretty jarring to the user (should definitely be updated later, probably with a `flash` of a message).

Next, I'll grab some information from GitPython:

```language-python
commit = repo.git_repo.commit(commit_sha)  
all_commits = list(repo.git_repo.iter_commits())  
current_commit_index = all_commits.index(commit)  
```

Here, I grab the commit object, the list of all commits, and the index of the commit that the URL supplied. Next, I need to look forward and backward for the next and previous commits, respectively, for navigation:

```language-python
previous_commit, next_commit = None, None  
if current_commit_index + 1 < len(all_commits):  
    previous_commit = all_commits[current_commit_index + 1]
if current_commit_index > 0:  
    next_commit = all_commits[current_commit_index - 1]
```

I'm just taking the index that I got before and getting the next and previous items in the list. Next, get the data from the commit:

```language-python
diff_raw = Markup.escape(repo.git_repo.git.diff(previous_commit, commit))  
```

I'm calling GitPython's `repo.git.diff`, passing the previous_commit and the commit that the user is actually attempting to see in order to get the changes made by that commit. This data comes as a string, then passed to Flask's Markup (from flask import Markup) to escape any HTML, as I'll be adding HTML to the string before passing it to the template to be rendered (and making use of the |safe template filter).

Here's that "markup":

```language-python
diff, color = '', None  
for line in diff_raw.splitlines():  
    if line.startswith('+++') or line.startswith('---'):
        continue
    elif line.startswith('+'):
        color = 'add'
    elif line.startswith('-'):
        color = 'remove'
    elif line.startswith('diff --git'):
        color = 'divider'
    elif line.startswith('@@') or line.startswith('index ') or \
            line.startswith('deleted file mode ') or line.startswith('new file mode'):
        color = 'info'
    else:
        color = 'none'
    diff += '<div class="code_row diff_%s">%s</div>' % (color, line)
```

I'm wrapping each of the lines in the output with a div with a class dependent on the contents of the line. If the line starts with a +, denoting that it was added in the commit, then I color the line green. Reversely, if the line starts with a -, denoting removal, then the line becomes red. "Information" lines, lines that denote where and which file was changed, are in gray. All other lines are not colored.

Finally, the call to render:

```language-python
return render_template('commit.html', repo=repo, commit=commit,  
    previous_commit=previous_commit, next_commit=next_commit, diff=diff)
```

This is just passing the data the method has collected to render_template.

#### The template

The template, though not as large as the `repo.html` template, is still a bit lengthy. Here is that file in full.

The first item of note is the title:

```language-html
<h3><span title="{{ commit.hexsha }}">{{ commit.hexsha|sha }}</span> by {{ commit.author }} on {{ commit.authored_date|date }}</h3>  
```

This uses two custom template filters. The first to condense the full SHA into 7 characters, and the second to parse the timestamp into a date:

```language-python
@app.template_filter('sha')
def filter_sha(s):  
    return s[:7]

@app.template_filter('date')
def filter_date(s):  
    return datetime.fromtimestamp(int(s)).strftime('%b %d, %Y at %I:%M:%S')
```

Next, there're the previous and next commit links that were found in the backend method:

```language-html
<div class="row">  
    <h4 class="pull-left">
    {% if previous_commit %}
        <a href="{{ url_for('commit', repo_name=repo.name, commit_sha=previous_commit.hexsha) }}">&lt; Previous commit</a>
    {% endif %}
    </h4>
    <h4 class="pull-right">
    {% if next_commit %}
        <a href="{{ url_for('commit', repo_name=repo.name, commit_sha=next_commit.hexsha) }}">Next commit &gt;</a>
    {% endif %}
    </h4>
</div>  
```
Note the if checks - if the variable is None, then the link isn't shown at all. The next bit is a table of the commit data which is pretty simple: just getting the data from the Commit object from GitPython.

Yes, that's it. A pre block (which is formatted by Bootstrap) and the data passed through Jinja's |safe filter. The classes added in the backend are defined in a CSS file:

```language-css
div.code_row {  
    margin: 0;
    padding: 0;
}

div.diff_add {  
    padding: 2px;
    background-color: rgba(0, 255, 0, 0.5);
}

div.diff_remove {  
    padding: 2px;
    background-color: rgba(255, 0, 0, 0.5);
}

div.diff_divider {  
    padding: 2px;
    margin-top: 2em;
    background-color: rgba(99, 99, 99, 0.5);
}

div.diff_info {  
    padding: 2px;
    background-color: rgba(99, 99, 99, 0.5);
}
```

## Inspect page

#### Backend method

The backend method here is a bit shorter:

```language-python
@app.route('/repo/<repo_name>/inspect/<path:code_path>')
def inspect(repo_name, code_path):  
    repo = Repository.query.filter_by(name=repo_name).first()
    if not repo:
        abort(404)
    f = repo.git_repo.head.commit.tree[code_path]
    data = f.data_stream.read().decode('UTF-8')
    up_directory = code_path[:code_path.rfind('/')]
    return render_template('inspect.html', repo=repo, code_path=code_path, up_directory=up_directory, data=data)
```

First up, there's the check for the valid DB model entry. Next, I get the blob for the file that the user is trying to see with the call to repo.git_repo.head.commit.tree[code_path]. This f variable now points to a Blob object. I make use if the .data_stream.read().decode method to get the contents of the file. up_directory is used for navigating back to the user's previous place in the repository. The data is passed to the render call, and off to the template.

#### The template

The inspect.html template is pretty small; it's only a few elements. First, I grab the Code-Prettify JavaScript file from a CDN to add syntax highlighting to the code:

```language-html
{% block head %}
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" defer></script>  
{% endblock head %}
```

This isn't in the content block like all the other templates (except base.html) have been - I'm adding this script to the HTML head.

Next, the Bootstrap breadcrumb so the user can go back:

```language-html
<ol class="breadcrumb">  
    <li><a href="{{ url_for('repo', repo_name=repo.name, code_path=up_directory) }}" class="active">Back to repo</a></li>
</ol>  
```

Then I show the path to the current file and its contents:

```language-html
<h3>{{ code_path }}</h3>

<div class="code_block">  
    <pre class="prettyprint">
{{ data }}
    </pre>
</div>  
```

Code-Prettify requires the prettyprint class to be added to the pre element, and the code_block class to the parent is defined in a custom CSS file:

```language-css
div.code_block > pre {  
    padding: 1em;
}
```

The formatting of the `pre` block here, without the padding added through the code_block class, is very tight and the edges of the code are a bit difficult to see. A little padding solves this problem.

## Conclusion

That's the bulk of the app! I've noted several places where improvements need to be made, but it's stable as-is. For a bit of a "Yo, dawg", I'm currently running the app at https://git.celeodor.com and you can browse it's source from the app itself.

# Running

If you want to run the app, just do the following:

```language-bash
$ git clone http://bitbucket.org/Celeo/CodeWall
$ cd CodeWall
$ virtualenv env
$ . env/bin/activate
$ pip install -r requirements.txt
$ ./run.sh
```

This will run the app through gunicorn on port 5000. Production/public running should include a reverse proxy to serve the static files. I use Nginx.