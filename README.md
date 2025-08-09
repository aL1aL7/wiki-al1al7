# Repository for my personal wiki/blog and digital notepad

The website is hosted at [wiki.al1al7.de](https://wiki.al1al7.de) and serves as a personal repository for code snippets, commands, and technical notes. It is not intended for public blogging or commercial use.

This website is built with [MkDocs](https://www.mkdocs.org/) and uses the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

To build this site, the only requirement is to have [Python](https://www.python.org/) installed.

To build the site, create a virtual environment, install the required packages, and run the build command:

1. Clone the repository

    ```bash
    git clone https://github.com/aL1aL7/wiki-al1al7.git
    cd wiki-al1al7
    ```
2. Create a virtual environment and activate it

    ```bash
    python -m venv venv
    source venv/bin/activate  # Linux/macOS
    # or
    venv\Scripts\activate  # Windows
    ```
3. Install the required packages    

    ```bash
    pip install -r requirements.txt
    ```
4. Build the site

    ```bash
    cd src
    mkdocs build
    ```
5. Or serve the site locally (optional)

    ```bash
    cd src
    mkdocs serve
    ```
