# Multipass installer

This script will install Docker via Multipass from scratch on MacOS.

Visual Studio Code is not needed for the install, but needed to make use of it.

## Prerequisites

The script should need absolutely nothing to run from a fresh install. However
you will need git to download it in the first place, so open a terminal and
enter:

```sh
xcode-select --install
```

## Usage

```sh
./install.sh
```

Enter your password when prompted.

Now just clone a repository containing a devcontainer configuration and select
`Reopen in Container` to open it in the Docker VM.

## Troubleshooting

When initially running the installer script, make sure you cloned to somewhere
other than a protected MacOS directory (such as `Desktop`). The recommended
option is to create `$HOME/Developer` (which MacOS automatically assigns a nice
icon) and clone to there. Otherwise you may get mounting issues where Docker
complains that the current directory doesn't exist.

Likewise, if you're cloning to a protected directory (e.g. under `Documents`)
then you may find that your containers fail to mount the directory. Either move
your files to a non-protected path or enable full disk access for `multipassd`
in your MacOS System Settings under Privacy & Security -> Full Disk Access.

If you get errors about `docker-credential-osxkeychain` not existing, either
install the binary or remove the `credsStore` line from `~/.docker/config.json`
(which was probably put there by Rancher Desktop).

If the Docker VM ever gets into a state where it won't launch, then delete it
and launch it again. This means you'll need to re-add keys (and remove the old
ones from your keystore) and re-mount your home directory:

```sh
multipass delete docker
multipass purge
multipass launch docker
multipass stop docker
multipass mount --type native "$HOME" docker
multipass start docker
multipass <~/.ssh/id_ed25519.pub exec docker -- bash -c 'cat -- >> ~/.ssh/authorized_keys'
ssh-keygen -R docker.local
```

## FAQs

### _How can I customise the amount of CPUs and memory allocated to the Virtual Machine?_

You can do this by setting the values in Multipass:

```sh
multipass stop docker
multipass set local.docker.cpus=4
multipass set local.docker.memory=8GiB
multipass start docker
```

### _Does the script need to mount my entire home directory onto the Virtual Machine?_

You don't _need_ to have the `$HOME` directory mounted, but it's recommended.

The script has been tested with specific non-protected mounts and has worked without errors. Refer to the Troubleshooting section of this README.

You can change mountpoints like so:

```sh
multipass stop docker
multipass umount docker:"$HOME"
multipass mount --type native "$HOME/my_developer_dir" docker # /my_developer_dir would contain all your devcontainer-supported repositories
multipass start docker
```

## TODO

-   Add a test suite.

## License

(C) CTS 2023

MIT License, see [LICENSE](LICENSE) for details.
