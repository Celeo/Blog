__Prest__ is a new library for accessing [EVE's CREST](https://developers.eveonline.com/).

---

* [git.celeodor.com](https://git.celeodor.com/Celeo/Prest)
* [GitHub](https://github.com/Celeo/Prest)
* [PyPI](https://pypi.python.org/pypi/EVEPrest)

For the EVE XML API, I've always used [eveapi](https://github.com/ntt/eveapi). For CREST, however, I took the opportunity of making my own library to interface with the API.

This was the first time that I've submitted a package to PyPI, and I used two good guides to do so:

* http://peterdowns.com/posts/first-time-with-pypi.html
* https://hynek.me/articles/sharing-your-labor-of-love-pypi-quick-and-dirty/

Making HTTP requests is done with the excellent [requests](http://docs.python-requests.org/en/master/) library.

Unit testing is done with [pytest](http://pytest.org/latest/xunit_setup.html).

PEP8 linting is done with [pep8](https://pypi.python.org/pypi/pep8), though I tend to ignore the restriction on line length - I prefer longer (to an extent, of course) lines as I think they improve readability instead of breaking the line up and working within the line continuation indent PEPs. Sorry.

This was a really fun (though sometimes frustrating) project that gave me the opportunity to do thing as "by the book" as I could. I'm still getting used to writing unit tests but I think what's there is a decent representation of the easily-testable parts of the library.

For now, I'm moving on to actually using the library in another project, so I'll likely be making tweaks or fixes to it as I go along so it'll stay updated.