Create New Web Development Project
==================================

> **NOTE:** This script is deprecated. My main workflow involves Git and 
GitHub now, so I don't really use this script anymore. Nevertheless, it 
remains here for posterity. Patches, via pull request, are still welcome.

I got sick of creating folder structures and Subversion repositories by 
hand, each and every time I had a new project. So I wrote this script 
that automates:

* Creates a new folder for the project
* Initialises a new Subversion repository
* Creates `doc`, `img`, `res`, and `www` folders
* Does a checkout to the working folder
* Creates a symlink in `~/Sites` enabling `http://localhost/~user/project`

There's also a `-r` option (or you can invoke as `rmwebproj`) to remove the 
symbolic link and archive the site to a tarball.


## Usage

	mkwebproj [-f] [--no-link|-r] project ...

**`-f`** Forces operations, despite existing files, permissions, etc.

**`--no-link`** Skips creation of the symbolic link in `~/Sites/`.

**`-r`** Archive, rather than create. Removes symbolic link and archives 
everything to a tarball (`projectname.tar.gz`).

**`project ...`** The name of the project to create or archive. Multiple 
projects may be specified using multiple arguments.


## Known issues

Archive operations fail in the event that not all files in the project 
folder can be read by the user running the script. The project folder 
is **not** deleted in this case.

Project names containing spaces are not supported. They should work, but 
you might run into issues that haven't been considered.

Only Subversion is supported as the revision control repository.