circle-docs
===========

Source code of the [Circle documentation](https://circle-rpi.readthedocs.io) on *Read the Docs*.

Local installation
------------------

This documentation can be built locally too. You will need a working python environment. Just enter in a shell:

	pip install sphinx
	pip install sphinx_rtd_theme

	cd /path/to/your/projects
	git clone https://github.com/rsta2/circle-docs.git
	cd circle-docs/docs

	make html

Then open [docs/_build/html/index.html](docs/_build/html/index.html) in your web browser.
