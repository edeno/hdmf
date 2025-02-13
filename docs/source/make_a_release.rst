=====================
How to Make a Release
=====================

A core developer should use the following steps to create a release ``X.Y.Z`` of **hdmf**.

.. note::

  Since the hdmf wheels do not include compiled code, they are considered
  *pure* and could be generated on any supported platform.

  That said, considering the instructions below have been tested on a Linux system,
  they may have to be adapted to work on macOS or Windows.

-------------
Prerequisites
-------------

* All CI tests are passing on `GitHub Actions`_.

* You have a `GPG signing key`_.

* Dependency versions in ``requirements.txt``, ``requirements-dev.txt``, ``requirements-opt.txt``,
  ``requirements-doc.txt``, and ``requirements-min.txt`` are up-to-date.

* Legal information and copyright dates in ``Legal.txt``, ``license.txt``, ``README.rst``,
  ``docs/source/conf.py``, and any other files are up-to-date.

* Package information in ``setup.py`` is up-to-date.

* ``README.rst`` information is up-to-date.

* The ``hdmf-common-schema`` submodule is up-to-date. The version number should be checked manually in case syncing the
  git submodule does not work as expected.

* Documentation reflects any new features and changes in HDMF functionality.

* Documentation builds locally.

* Documentation builds on the `ReadTheDocs project`_ on the "dev" build.

* Release notes have been prepared.

* An appropriate new version number has been selected.

-------------------------
Documentation conventions
-------------------------

The commands reported below should be evaluated in the same terminal session.

Commands to evaluate starts with a dollar sign. For example::

  $ echo "Hello"
  Hello

means that ``echo "Hello"`` should be copied and evaluated in the terminal.


-------------------------------------
Publish release on PyPI: Step-by-step
-------------------------------------

1. Make sure that all CI tests are passing on `GitHub Actions`_.


2. List all tags sorted by version.

  .. code::

    $ git tag -l | sort -V


3. Choose the next release version number and store it in a variable.

  .. code::

    $ release=X.Y.Z

  .. warning::

      To ensure the packages are uploaded on PyPI, tags must match this regular
      expression: ``^[0-9]+.[0-9]+.[0-9]+$``.


4. Download the latest sources.

  .. code::

    $ cd /tmp && git clone --recurse-submodules git@github.com:hdmf-dev/hdmf && cd hdmf


5. Tag the release.

  .. code::

    $ git tag --sign -m "hdmf ${release}" ${release} origin/dev

  .. warning::

      This step requires a `GPG signing key`_.


6. Publish the release tag.

  .. code::

    $ git push origin ${release}

  .. important::

      This will trigger the "Deploy release" GitHub Actions workflow which will automatically upload the wheels
      and source distribution to both the `HDMF PyPI project page`_ and a new `GitHub release`_
      using the hdmf-bot account.


7. Check the status of the builds on `GitHub Actions`_.


8. Once the builds are completed, check that the distributions are available on `HDMF PyPI project page`_ and that
   a new `GitHub release`_ was created.


9. Copy the release notes from ``CHANGELOG.md`` to the newly created `GitHub release`_.


10. Create a clean testing environment to test the installation.

  On bash/zsh:

  .. code::

    $ python -m venv hdmf-${release}-install-test && \
      source hdmf-${release}-install-test/bin/activate

  On other shells, see the `Python instructions for creating a virtual environment`_.


11. Test the installation:

  .. code::

    $ pip install hdmf && \
      python -c "import hdmf; print(hdmf.__version__)"


10. Cleanup

  On bash/zsh:

  .. code::

    $ deactivate && \
      rm -rf dist/* && \
      rm -rf hdmf-${release}-install-test


.. _GPG signing key: https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key
.. _ReadTheDocs project: https://readthedocs.org/projects/hdmf/builds/
.. _GitHub Actions: https://github.com/hdmf-dev/hdmf/actions
.. _GitHub release: https://github.com/hdmf-dev/hdmf/releases
.. _HDMF PyPI project page: https://pypi.org/project/hdmf
.. _Python instructions for creating a virtual environment: https://docs.python.org/3/library/venv.html#creating-virtual-environments


--------------------------------------------
Publish release on conda-forge: Step-by-step
--------------------------------------------

.. warning::

   Publishing on conda requires you to have the corresponding package version uploaded on
   PyPI. So you have to do the PyPI and GitHub release before you do the conda release.

.. note::

   Conda-forge maintains a bot called "regro-cf-autotick-bot" that regularly monitors PyPI for new releases of
   packages that are also on conda-forge. When a new release is detected, usually within 24 hours of publishing
   on PyPI, the bot will create a Pull Request with the correct modifications to the version and sha256 values
   in ``meta.yaml``. If the requirements in ``setup.py`` have been changed, then you need to modify the
   requirements/run section in ``meta.yaml`` manually to reflect these changes. Once tests pass, merge the PR,
   and a new release will be published on Anaconda cloud. This is the easiest way to update the package version
   on conda-forge.

In order to release a new version on conda-forge manually, follow the steps below:

1. Store the release version string (this should match the PyPI version that you already published).

  .. code::

    $ release=X.Y.Z


2. Fork the `hdmf-feedstock <https://github.com/conda-forge/hdmf-feedstock>`_ repository to your GitHub user account.


3. Clone the forked feedstock to your local filesystem.

   Fill the YOURGITHUBUSER part.

   .. code::

      $ cd /tmp && git clone https://github.com/YOURGITHUBUSER/hdmf-feedstock.git


4. Download the corresponding source for the release version.

  .. code::

    $ cd /tmp && \
      wget https://github.com/hdmf-dev/hdmf/releases/download/$release/hdmf-$release.tar.gz


5. Create a new branch.

   .. code::

      $ cd hdmf-feedstock && \
        git checkout -b $release


6. Modify ``meta.yaml``.

   Update the `version string <https://github.com/conda-forge/hdmf-feedstock/blob/main/recipe/meta.yaml#L2>`_ and
   `sha256 <https://github.com/conda-forge/hdmf-feedstock/blob/main/recipe/meta.yaml#L3>`_.

   We have to modify the sha and the version string in the ``meta.yaml`` file.

   For linux flavors:

   .. code::

      $ sed -i "2s/.*/{% set version = \"$release\" %}/" recipe/meta.yaml
      $ sha=$(openssl sha256 /tmp/hdmf-$release.tar.gz | awk '{print $2}')
      $ sed -i "3s/.*/{$ set sha256 = \"$sha\" %}/" recipe/meta.yaml

   For macOS:

   .. code::

      $ sed -i -- "2s/.*/{% set version = \"$release\" %}/" recipe/meta.yaml
      $ sha=$(openssl sha256 /tmp/hdmf-$release.tar.gz | awk '{print $2}')
      $ sed -i -- "3s/.*/{$ set sha256 = \"$sha\" %}/" recipe/meta.yaml

  If the requirements in ``setup.py`` have been changed, then modify the requirements/run list in
  the ``meta.yaml`` file to reflect these changes.


7. Push the changes to your fork.

   .. code::

      $ git push origin $release


8. Create a Pull Request.

   Create a pull request against the `main feedstock repository <https://github.com/conda-forge/hdmf-feedstock/pulls>`_.
   After the tests pass, merge the PR, and a new release will be published on Anaconda cloud.
