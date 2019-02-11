.. title: Searching and browsing CMSSW: use git
.. slug: cmssw-gitgrepview
.. date: 2019-02-11 09:15:00 UTC+01:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

CMSSW_, the software that the CMS experiment uses to process collision data,
from the detector readout to a format suitable for analysis,
is contained in one big git repository, `cms-sw/cmssw.git <https://github.com/cms-sw/cmssw>`_.
Since documentation is not always kept up to date, or missing some details,
it occurs quite often that the only way to figure out how some numbers came about
is to check the code itself |---| this is even more the case when doing development.

For saving time during development, git's `sparse checkout`_ feature is heavily used, such that
only the minimally required (modified, and depending) packages are checked out and built.
The other targets |---| mostly shared libraries, executables, and python modules |---|
are taken from the corresponding release area, which is distributed through
the `CernVM-FS (or cvmfs) <https://cvmfs.readthedocs.io/>`_ filesystem.
Since also the git objects are distributed in this way (to all CMS colleagues who have not
already done so: set ``CMSSW_GIT_REFERENCE=/cvmfs/cms.cern.ch/cmssw.git.daily`` when working
on a machine with cvmfs), the cost of setting up a working area is very small (it typically
takes a few seconds to run ``cmsrel``).

A less known feature of such a setup is that it also allows for very fast (and flexible)
searching of the full repository, parts of it, and the complete git history, with
``git grep`` and ``git show``. The rest of this post describes my favourite tricks and aliases.

git grep and ack
================

There are many online introductions to using ``git grep``, see e.g. the
`"Search a git repo like a ninja" post by Travis Jeffery <http://travisjeffery.com/b/2012/02/search-a-git-repo-like-a-ninja/>`_,
which also introduces a ``git ack`` alias to format the results like `ack`_
(if you do not know `ack`_ yet: it is like `grep`_ but with better defaults, switches to search only some file types etc.
|---| really handy, and there is a one-file install option).
I would recommend the following customization (in ``~/.gitconfig``):

.. code-block:: ini

   [alias]
     ack = grep --break --heading --line-number --color=auto
   [grep]
     extendRegexp = true
     lineNumber = true
     color = auto

The short version: run ``git grep`` from the root of the repository (``$CMSSW_BASE/src``) with one of

.. code-block:: sh

   git [grep or ack] pattern <tree> -- <path>

where both the ``<tree>`` and the ``-- path`` parts, to specify the revision, tag or branch,
and a part of the repository that should be searched, respectively, are optional.

As a final example, the following command searches for the definition of the the ``jetId``
variable in a specific release:

.. code-block:: sh

   git ack jetId CMSSW_10_2_9 -- PhysicsTools/NanoAOD

git show with vim as a pager, and syntax higlighting
====================================================

Nothing too special by itself, but a useful helper command to know about: ``git show revision:path``
can be used as the equivalent of ``cat``, ``less`` (or your favourite plain-text pager)
for any revision of any file (also when they are not in the working directory).

Of course ``cat`` and ``less`` are not the most convenient tools to browse large source files.
vim can be used as a pager (with the `less.vim <https://github.com/vim/vim/blob/master/runtime/macros/less.vim>`_
script). Syntax highlighting does not work out of the box, however, 
because ``ftplugin`` mostly relies on the filename extension for detecting the filetype.
But when calling ``git show``, we have a filename, so we can use a trick:
first ask ``ftplugin`` which filetype it would resolve, and then pass it when opening vim.
This is what is done in the `filetypeless.sh`_ script (which should be installed in 
``~/.vim/macros/``), the gist of it is

.. code-block:: sh

   ftcmd=$(vim -es --cmd "filetype on" "${1}" -c ':exec ":set filetype?" | exec ":q!"')
   vim -R --cmd 'let no_plugin_maps = 1' -c "set ${ftcmd}" -c 'runtime! macros/less.vim'

The following git alias (using the trick from
`this recipe <https://git.wiki.kernel.org/index.php/Aliases#Use_graphviz_for_display>`_)
will run the script on the output of ``git show``

.. code-block:: ini

   [alias]
     view = "!v() { git show $1 | ~/.vim/macros/filetypeless.sh $1; }; v"

and that's all: ``git view HEAD:some_file.cpp`` now shows the file in vim, with
(in this case C++) syntax highlighting.

.. _CMSSW: https://cms-sw.github.io/

.. _sparse checkout: https://git-scm.com/docs/git-read-tree#_sparse_checkout

.. _grep: https://www.gnu.org/software/grep/manual/grep.html

.. _ack: https://beyondgrep.com/

.. _filetypeless.sh: https://gist.github.com/pieterdavid/b58c586e333ab0ac00b7b7499e4a487e

.. |---| unicode:: U+2014
   :trim:
