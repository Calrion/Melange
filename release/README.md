One-step Builds for Xcode-based Projects
========================================

The `release` script makes building an Xcode-based project as easy as:

    release ProjectName buildtype

That command will:

1. Download a pristine copy of your project's repository from GitHub.

2. Download any Git submodules.

3. Checkout the appropriate branch.

4. Run `xcodebuild` for the scheme specified by `buildtype`.

That's all it does, and that's all it's designed to do. This is with the full 
intention that _anything your build needs to do should be done **within** 
Xcode's build process_.

As an example of this, I configure my Xcode projects to include versioning 
information based on the repository state, add iTunes artwork, collect debug 
symbols, upload builds to TestFlight, and archive everything to Amazon S3.

By having all this done within Xcode (using scripts), releases can be made 
either by hitting the big button in Xcode or from the command-line in a 
_single step_. A single-step build process means builds happen the _same way_ 
every, single time.


## Requirements
The `release` script needs to know a whole bunch of things, so there's a few 
requirements. I've taken the Ruby on Rails approach of reasonable defaults, 
that can be overridden if necessary. You may not have anything much to do at 
all.

### Xcode
You'll need to have Apple Xcode 4.1 or later installed. The Xcode 
command-line tools are **not** required.

### GitHub credentials
The script expects that the Git configuration variable `github.name` will 
be set to the GitHub username to use when cloning the repository. It also 
expects that the ssh subsystem will know how to authenticate to `github.com` 
(i.e. that you have an authorised key and have configured ssh to use it 
in `~/.ssh/config`).

### Project format
The script expects your repository to have a folder named `Project`, and for 
that folder to contain an Xcode workspace named `Release.xcworkspace`. The 
release workspace needs to have a scheme named `Buildtype Release` for each 
build type you want to make. So if you want to run `release ProjectName 
Daily` then you'll need a scheme named `Daily Release`. For now, the case 
_is important_.

The script will use the scheme to have `xcodebuild` build an _archive_ of 
the project; you'll need to make sure that building an archive of the 
`Buildtype Release` scheme in the release workspace will do whatever you 
need it to. The release script will make sure that the build is made with 
a pristine, complete copy of the latest version of your repository.

Just to be clear, if your GitHub username is `Freddie` and you run:

    release MyThing Daily

The release script will clone `ssh://git@github.com/Freddie/MyThing.git` and have Xcode produce an archive of the `Daily Release` scheme in `/Project/Release.xcworkspace`.


## Known Issues

### Organisation repositories are unsupported
This script may not (or may!) work if your repository is within a GitHub 
organisation rather than a user account. It's untested and unsupported (at 
this time).

### Repositories are downloaded in full, every time
There's no caching of repositories, yet. Every time you run `release`, it 
creates a new temp folder and downloads a complete copy of your repository, 
including complete copies of all submodules.

### What, me check errors!?
This script was scraped together by trial and error, and is still quite rough 
around the edges. If your project is properly formatted, your configuration 
is right, and you invoke the script using the proper syntax, then you 
shouldn't have any problems; if not, a train wreck could well result. You've 
been warned.
