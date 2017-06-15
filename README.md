# obs_pkg_mgr

This repository hosts the obs_pkg_mgr script. This is a script that can be used in place of [`zypper`](https://github.com/openSUSE/zypper) in Dockerfiles in order to make it possible to build these Dockerfiles in the [Open Build Service](https://github.com/openSUSE/open-build-service).

## Why

Inside OBS, during building of a package, the building scripts don't have access to the internet. The dependencies for a package are provided by OBS itself. Calling `zypper in -y gcc` won't work inside OBS because this command will try to fetch the "gcc" package from a remote location.

Another reason to use this script instead of the original package manager (zypper), is that this way it is easier for OBS service to extract dependencies from commands like `RUN obs_pkg_mgr install ruby` because of the simplyfied syntax and zero options passed.

## How

### obs_pkg_mgr install

The `obs_pkg_mgr` script disables all enabled repositories temporarily and enables a local one as defined by the `OBS_REPOSITORY_URL` environment variable. That repository serves all the dependencies in the Dockerfile. After the packages are installed all disabled repositories are re-enabled.

When `OBS_REPOSITORY_URL` is not defined, this scripts doesn't disable or enable any repositories and simply delegates to `zypper`. 

### obs_pkg_mgr add_repo

In some cases the repositories already enabled in the base image do not include all the packages needed in the final image. In those cases we need to add additional repositories. OBS needs to know about these additional repositories so that the needed packages are provided at build time. For that reason, this script also provides the `obs_pkg_mgr add_repo` command. This command will enable the repository in the final image but during build time it will not try to refresh it otherwise it would fail because of the network limitations.

This command is meant to replace `zypper addrepo` commands in Dockerfiles.

## Example usage

The following Dockerfile is an example of how this script can be used:

```
FROM opensuse:42.2

# Make obs_pkg_mgr available
ARG obs_repository_url
ENV OBS_REPOSITORY_URL $obs_repository_url
ADD obs_pkg_mgr /usr/bin/obs_pkg_mgr
RUN chmod +x /usr/bin/obs_pkg_mgr

RUN obs_pkg_mgr add_repo http://download.opensuse.org/repositories/Cloud:/Platform:/scf/openSUSE_Leap_42.2/ "Cloud:Platform:scf (openSUSE_Leap_42.2)"
RUN obs_pkg_mgr install dumb-init ruby2.3-rubygem-configgin
```

## Future work

- Make this scripts capable of delegating to other package managers as well in order to serve other distributions (e.g. `apt-get`).
- Move from a bash script to some "real" programming language.
