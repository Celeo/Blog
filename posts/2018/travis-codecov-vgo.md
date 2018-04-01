# Travis CI, Codecov, and vgo

[vgo](https://github.com/golang/vgo) is the (newest) official way to managed versioned dependencies in Go. In my [most recent](https://github.com/Celeo/dnd) project, I’ve decided to use it instead of dep or glide, which I’ve used previously. Taking a leaf out of the recent work I’ve been doing _at_ work, I’ve hooked that repository up to some free CI tools, including [Travis CI](travis-ci.org).

## Getting it all to work

The chief problem I had was getting vgo to play nice on Travis. You can’t tell now because I rebased my commit history, but I had a lot of trouble getting Travis to build and test what I actually wanted it to without throwing errors about Go dependencies and GitHub errors.

### Getting vgo installed in the build

First off, getting vgo installed, as it doesn’t exist by default on Travis (obviously). Travis has an **install** step that I’m using to install vgo:

```yml
install:
  - go get -u golang.org/x/vgo
```

That should look very familiar - it’s just installing vgo like any other **go get** command would.

### Testing via vgo

By default, Travis runs the test command against the **go** binary, so I’m overriding the **script** step to use vgo instead:

```yml
script:
  - vgo test -v -race -coverprofile=coverage.txt -covermode=atomic ./...
```

Note that command also runs the race condition checked (not actually needed for this library) and saves the coverage to a file called **coverage.txt**. That’s up in a bit.

If you have *just this* for your vgo test command, it’ll fail, citing the GitHub API rate limits for unauthorized requests. The way to get around this on your dev machine is to edit **~/.netrc** with a generated GitHub Personal Access Token. Travis recommends doing this too in the build in their [docs](https://docs.travis-ci.com/user/languages/go/#Installing-Private-Dependencies) for private dependencies, and it can work for vgo’s problems with GitHub rate limits.

I didn’t want to put a key in a file in my repository, but I still needed that **.netrc** file to be created during my build, so I’m using an **echo** command and output redirection in the **before_install** step in Travis to *echo* that token into the **.netrc** file. I’m taking the token (and my GitHub login) as hidden environment variables in the job configuration so I don’t have to save them in plain text in the actual repo. The setup for the **.netrc** file looks like this:

```yml
before_install:
  - echo -e "machine api.github.com\n  login ${GITHUB_LOGIN}\n  password ${GITHUB_TOKEN}" > ~/.netrc
  - chmod 600 ~/.netrc
```

With this, vgo has the system configuration it needs to get past GitHub’s rate limits.

### Coverage

If you *just* wanted to run unit tests in vgo, then the command would just be `vgo test` with any other parameters you want to supply, but I’m also calculating coverage, as [http://codecov.io](Codecov) is free for open-source repositories (just like Travis) and I’d like to have my repo have a coverage percentage badge as well as a “build: passing” badge.

Codecov has an [example repo](https://github.com/codecov/example-go) for uploading public (or private) coverage metrics as part of the Travis build, so the end of my **.travis.yml** file looks like:

```yml
after_success:
  - bash <(curl -s https://codecov.io/bash)
```

### Full config

My full **.travis.yml** file at time of writing is this:

```yml
language: go
go:
  - "1.10.x"
before_install:
  - echo -e "machine api.github.com\n  login ${GITHUB_LOGIN}\n  password ${GITHUB_TOKEN}" > ~/.netrc
  - chmod 600 ~/.netrc
install:
  - go get -u golang.org/x/vgo
script:
  - vgo test -v -race -coverprofile=coverage.txt -covermode=atomic ./...
after_success:
  - bash <(curl -s https://codecov.io/bash)
```

And my Travis repo settings has environment variables for `CODECOV_TOKEN`, `GITHUB_LOGIN`, and `GITHUB_TOKEN`.

## Summary

vgo was annoying to get running properly, but Travis runs great and Codecov was a breeze. Now that both sites are configured, I get builds and coverage metrics whenever I push, all for free.

