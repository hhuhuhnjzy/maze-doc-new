============
Installation
============


------------
Prerequisites
------------

We recommend using Python 3.10 or newer.

- Python 3.10 or newer
 

You can verify your Python and pip versions with:

.. code-block:: bash

    python --version


------------------
Install from Source
------------------
We only support installation from source now.

1. Clone the repository:

.. code-block:: bash

    git clone https://github.com/your-username/your-repo.git
    cd your-repo

2. Install the package in development mode:

.. code-block:: bash

    pip install -e .

The ``-e`` flag installs the package in "editable" (development) mode, so changes to the source code take effect immediately without reinstalling.

------------------------
Using a Virtual Environment (Recommended)
------------------------

To avoid dependency conflicts, we strongly recommend using a virtual environment:

.. code-block:: bash

    python -m venv venv
    source venv/bin/activate  # Linux/macOS
    # or
    venv\Scripts\activate     # Windows

    pip install your-package-name

------------
Verify Installation
------------

After installation, confirm it worked by running:

.. code-block:: bash

    python -c "import maze; print(maze.__version__)"

If no error occurs and the version number is printed, the installation was successful.