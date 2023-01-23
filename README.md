
# K8s and GitSecOps

This workshops walks you through setting up a model for using GitSecOps for a cloud native application deployed on Kubernetes 

## Repository Structure

The repository structure follows the [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) recommended layout.

```
.
├── README.md
├── docs/             ## Folder containing the documentation
├── mkdocs.yml        ## MKdocs configuration file
├── overrides/        ## Folder with theme layout customization
├── support/          ## Folder with support and resources for the initiative   
├── archive           ## Archived content
└── requirements.txt  ## Python libraries required to build the site
```

The **docs** folder contains the markdown files with the documentation. The folder structure within it is used to build the layout menu and the URL paths of the pages. [Here](https://www.mkdocs.org/user-guide/configuration/#docs_dir) you'll find more information about how the docs folder works.

## Running Local

It is highly recommended that you view your changes locally before committing and pushing. [MkDocs](https://www.mkdocs.org/) is built in python so you need to have `python3` and `pip3` installed in your environment. The `requirements.txt` file contains the python libraries required to build the site.

On the root folder:

```sh
$ pip3 uninstall -r requirements.txt
$ pip3 install -r requirements.txt
$ mkdocs serve -a 0.0.0.0:8000
```

Go to [http://localhost:8000](http://localhost:8000). 

The dev-server supports auto-reloading and it will rebuild the documentation whenever anything in the configuration file, documentation directory, or theme directory changes.

## What we are working on

Please see the TODO.md file if you want a really quick look at what we are working.  


