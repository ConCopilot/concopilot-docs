# The `conpack` CLI

## build

```shell
conpack build
        [--settings=<settings>] # The `conpack` config setting file path, absent for default
        [--working-directory=<working_directory>] # The working directory (where to place the ".runtime" folder), default to current directory
        [
            [--group-id=<group-id> --artifact-id=<artifact-id> --version=<version>]
            [--src-folder=<src_folder>] # The source folder (where the ".config" folder exists), default to current directory
            [--config-file=<config-file>] # The config.yaml file path
        ]
        [*argv]
```

This command is used to test if the specified component can be successfully built and constructed.

All required python package appeared under the `setup.pip` section in the "config.yaml" of this component and other involved component will be installed.

This command will not run the component, thus do NOT to use this for runtime test.

## run

```shell
conpack run
        [--settings=<settings>] # The `conpack` config setting file path, absent for default
        [--working-directory=<working_directory>] # The working directory (where to place the ".runtime" folder), default to current directory
        [
            [--group-id=<group-id> --artifact-id=<artifact-id> --version=<version>]
            [--src-folder=<src_folder>] # The source folder (where the ".config" folder exists), default to current directory
            [--config-file=<config-file>] # The config.yaml file path
        ]
        [--skip-build]
        [*argv]
```

This command is used to run a specific copilot.
Please note that only a `Copilot` can be run, other components, like plugins, cannot.
An exception will be raised if the target component is not a `Copilot`.

## install

```shell
conpack install
        [--settings=<settings>] # The `conpack` config setting file path, absent for default
        [--working-directory=<working_directory>] # The working directory (where to place the ".runtime" folder), default to current directory
        [--src-folder=<src_folder>] # The source folder (where the ".config" folder exists), default to current directory
        [--skip-build]
        [--recursive] # check all sub-folder of the <src_folder> for possible config files
        [*argv]
```

This command will install your components configs into your local repository.

The default path of your local repository is `<user_home>/.concopilot/repository`.

## deploy

```shell
conpack deploy
        [--settings=<settings>] # The `conpack` config setting file path, absent for default
        [--working-directory=<working_directory>] # The working directory (where to place the ".runtime" folder), default to current directory
        [--src-folder=<src_folder>] # The source folder (where the ".config" folder exists), default to current directory
        [--skip-build]
        [--recursive] # check all sub-folder of the <src_folder> for possible config files
        [--repo-user-name=<repo_user_name>] [--repo-user-pwd=<repo_user_pwd>] [--gpg-passphrase=<gpg_passphrase>] [--gnupg-home=<gnupg_home>]
        [*argv]
```

This command will deploy your components configs into the remote repository.

You need to provide your repository user name and password for login.
Please register your account on our [website](https://concopilot.org).

You also need to apply the accessibility of a group_id which correlated a domain that under your control on our [website](https://concopilot.org).
Personal group_id are also accessible, such like a Git account.
Please check [Apply a Group ID](group_id.md) for details.

A GPG is also recommended.
We follow the same instructions of the Java Maven Central Repository.
See [this](https://central.sonatype.org/publish/requirements/gpg/) for detail.

Please note that we only deploy component config files (each in a ".config" folder, especially the "config.yaml") into the remote repository.
Do NOT forget to deploy your python code separately, too.
