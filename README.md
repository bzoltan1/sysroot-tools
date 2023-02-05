# Creating sysroot with development APIs for SDK

This tool is to bootstrap a  minimal zypper/rpm based system and installes a defined set of API/development packages.
Additionally it can be used to hook up the sysroot as chroot.

## Warning

This tool operates with chroots. It means that essential system resources (/dev, /tmp, etc.) are hooked up to the working directory.

DO NOT EVER TERMINATE THE SYSROOT CREATION.

If the tool breaks or terminates for whatever reason please check with mount command if there is any mount point left in the working directory.

Please note that the json file may need to be updated and adjusted as the content of the repositories change. Packages may get removed, 
renamed and  updated. This will cause the download or installation processes fail. All failures should be graceful but if not then the 
tool may leave artifacts and mounted system directories behind. In that case please be careful when removing directories created by this tool

## Requirements

The following tools must be available on the system where the sysroot tool is used:

* openssl
* rpm2cpio
* cpio
* chroot
* tar
* qemu-user-static

## How to use

The sysroot tool is using chroot and mount commands so it must be executed with sudo

Create a new sysroot:
```
$ sudo sysroot create -f [sysroot template json file] [--keep]
```
Mount a sysroot file for using it with chroot:
```
$ sudo sysroot mount -f [sysroot json file]
```
Decouple the mounted sysroot file:
```
$ sudo sysroot umount -f [sysroot json file] [--keep]
```
Log in to an existing sysroot:
```
$ sudo sysroot login -f [sysroot json file]
```
Build an rpm project in the sysroot
```
$ sudo sysroot build -p [project diretory] -f [sysroot json file] 
```

## Options and parameters of the sysroot tool

* -f [ JSON file]
	+ This parameter defines what JSON file should be taken as configuration
* -d [ directory name ]
	+ The directory name should be an existing directory where the sysroot files and the sysroot chroots will be placed  
* -p [ project directory name ]
        + The directory name where the source tree of the rpm project is located

* create
	+ Command to create a sysroot file
        + When the --keep is used the sysroot directory is only decoupled from the host system but not removed.
* mount
	+ Command to mount a sysroot.tar.xz file. Both the tar.xz and its JSON file must be present
        + If the sysroot was created or umounted with --keep option then the existing sysroot directory will be used
* umount
	+ Command to decouple the mounted sysroot file
        + When the --keep is used the sysroot directory is only decoupled from the host system but not removed.
* login
        + Command to mount the sysroot file and log in with chroot
* build
        + Command to mount up the sysroot file and build an rpm project against it in a chroot environment
        + The [project name]-[ timestamp].buildlog file is placed under the work directory (configurable with -d)
        + The binary packages are placed under the $HOME/rpmbuild/RPMS/[ target architecture ]/ directory

## How the sysroot template json file looks

There is a `sysroot-template.json` what can be used to create new sysroot definitions.

Description of the sysroot json file:

* sysroot_directory : This is how the sysroot will be named (for example openSUSE-current-x86)
* qemu_static: (optional) The name(s) of the qemu binary in case the target arch is different from the host's arch. Can be a single string or an array of strings.
* target_arch : The target arch can be any arch what QEMU supports on the host system
* ssl_certificate: SSL certificate for the package repositories if they require such
* keep_zypp_settints: The tool can clean up or leave the zypp credentials and setting in the sysroot (boolean)
* target_certs: If the repositories require certification they should be appended to these files
* repositories: List of repositories where each repository must have the following fields:

Description of the repository record

* name: Unique identifier of the repository 
* type: The repositories are either cross or native. Native repositories are used by zypper, the packages from cross repositories are directly downloaded and unpacked to a specific path on the image root
* packages: Packages from the repository to be installed on the image. The packages may contain a version number separated by a space from the package name.

For cross repositories
* target: The absolut path on the image rootfs where the packages are unpacked
* url : The server address where the individual packages can be direct downloaded
* repository : The path on the server to the repository
* username : User name for the repository if it requires authentication
* password" : Password for the repository if it requires authentication

For native repositories
* repository: This the repository where all the target packages are located
* zypp_repos: The /etc/zipp/repo.d/ repositories where the SDK packages are installed from
* zypp_credentials: Credentials for the package repositories if they require authentication

## Output

The sysroot tool produces a 
* [sysroot_directory]-sysroot.tar.xz file what is a root filesystem of chrootable system
* [sysroot_directory]-sysroot.json file what contains information about the sysroot

The [sysroot_directory]-sysroot.json has the following fields:
* size: The size of the [sysroot_directory]-sysroot.tar.xz  in bytes
* created: The creation time of the [sysroot_directory]-sysroot.tar.xz in a human readable format 
file_name: The filename of the  [sysroot_directory]-sysroot.tar.xz
architecture:  The target arch if the sysroot 
combined_sha256: Combined MD5 and SHA256 checksum of the [sysroot_directory]-sysroot.tar.xz
sha256: SHA256 checksum of the [sysroot_directory]-sysroot.tar.xz
md5: MD5 checksum of the [sysroot_directory]-sysroot.tar.xz

## Supplementary tools

### Dependency listing tool

In  zypper/rpm based systems packages require resources like executable binaries, libraries or whatever files on the systems. It is not self evident what exact package dependecies a package needs.
In order to build a minimal core system one needs to know the exact and dependency tree of the core packages (glibc, bash, rpm and zypper)

```
$ dependency [package name]
```

The dependency tool goes through the requirement list of the package [package name] and systematically checks what package provides the resource. The dependency tool creates a dependencies.txt and a requirements.txt files each contains the full package dependency and required resource list of the package [package name].
