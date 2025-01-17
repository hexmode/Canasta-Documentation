# Canasta CLI
The Canasta command line interface, written in Go.

## Before starting
You should have Docker Engine and Docker Compose installed. This is very fast and easy to do on common Linux distros such as Debian, Ubuntu, Red Hat, and CentOS. By installing Docker Engine from `apt` or `yum`, you get Docker Compose along with it. See the following install guides for each OS:

* [Debian](https://docs.docker.com/engine/install/debian/)
* [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
* [CentOS](https://docs.docker.com/engine/install/centos/)
* More available at [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

## Installation
* Then, run the following line to install the Canasta CLI:

```
curl -fsL https://raw.githubusercontent.com/CanastaWiki/Canasta-CLI/main/install.sh | bash
``` 

## All available commands

```
A CLI tool to create, import, start, stop and backup multiple Canasta installations

Usage:
  sudo canasta [command]

Available Commands:
  list        List all Canasta installations
  create      Create a Canasta installation
  import      Import a wiki installation
  start       Start the Canasta installation
  stop        Shuts down the Canasta installation
  restart     Restart the Canasta installation
  skin        Manage Canasta skins
  extension   Manage Canasta extensions
  restic      Use restic to backup and restore Canasta
  maintenance Run maintenance update jobs
  delete      Delete a Canasta installation
  help        Help about any command

Flags:
  -h, --help      help for Canasta
  -v, --verbose   Verbose output


Use "sudo canasta [command] --help" for more information about a command.
```
## Create a new wiki
* Run the following command to create a new Canasta installation with default configurations.
```
sudo canasta create -i canastaId -n example.com -w Canasta Wiki -a admin -o docker-compose
```
* Visit your wiki at its URL, "https://example.com" as in the above example (or http://localhost if installed locally or if you did not specify any domain).
* For more info on finishing up your installation, visit [after installation](setup.md#after-installation).

## Import an existing wiki
* Place all the files mentioned below in the same directory for ease of use.
* Create a .env file and customize as needed (more details on how to configure it at [configuration](setup.md#configuration), and for an example see [.env.example](https://github.com/CanastaWiki/Canasta-DockerCompose/blob/main/.env.example)).
* Drop your database dump (in either a .sql or .sql.gz file).
* Place your existing LocalSettings.php and change your database configuration to be the following:
  * Database host: db
  * Database user: root
  * Database password: mediawiki (by default; see [Configuration](setup.md#configuration))
* Then run the following command:
```
sudo canasta import -i importWikiId -d ./backup.sql.gz -e ./.env -l ./LocalSettings.php  
```
* Visit your wiki at its URL (or http://localhost if installed locally or if you did not specify any domain).
* For more info on finishing up your installation, visit [after installation](setup.md#after-installation).

## Enable/disable an extension
* To list all [Canasta extensions](https://canasta.wiki/documentation/#extensions-included-in-canasta) that can be enabled or disabled with the CLI, run the following command:
```
sudo canasta extension list -i canastaId
```
* To enable a Canasta extension, run the following command:
```
sudo canasta extension enable Bootstrap -i canastaId
```
* To disable a Canasta extension, run the following command:
```
sudo canasta extension disable VisualEditor -i canastaId
```
* To enable/disable multiple Canasta extensions, separate the extensions with a ',':
```
sudo canasta extension enable VisualEditor,PluggableAuth -i canastaId
```
* Note: The extension names are case-sensitive.


## Enable/disable a skin
* To list all [Canasta skins](https://canasta.wiki/documentation/#skins-included-in-canasta) that can be enabled or disabled with the CLI, run the following command:
```
sudo canasta skin list -i canastaId
```
* To enable a Canasta skin, run the following command:
```
sudo canasta skin enable Vector -i canastaId
```
* To disable a Canasta skin, run the following command:
```
sudo canasta skin disable Refreshed -i canastaId
```
* To enable/disable multiple Canasta extensions, separate the extensions with a ',':
```
sudo canasta skin enable CologneBlue,Modern -i canastaId
```
* Note: the skin names are case-sensitive.

## Using restic
[restic](https://restic.net) is a very useful utility for doing automated backups to a variety of different storage types; though Canasta's usage of restic is configured for using AWS S3-based repositories.

Canasta makes use of restic's [dockerized binary](https://hub.docker.com/r/restic/restic).

### How to get started
1. Add these environment variables to your Canasta installation's `.env` file. Follow the steps at [cli-configure-quickstart](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds-create) to obtain ACCESS_KEY_ID and SECRET_ACESS_KEY.
```
AWS_S3_API=s3.amazonaws.com
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_S3_BUCKET=
RESTIC_PASSWORD=
```
2. When using restic for the first time in a Canasta installation, run the following command to initialize a restic repo in the AWS S3 bucket specified in the `.env` file:
```
sudo canata restic init -i canastaId
```
Now you should be able to use any of the available commands.

### Available restic commands
  ```
  check         Check restic snapshots
  diff          Show difference between two snapshots
  forget        Forget restic snapshots
  init          Initialize a restic repo
  list          List files in a snapshost
  restore       Restore restic snapshot
  take-snapshot Take restic snapshots
  unlock        Remove locks other processes created
  view          View restic snapshots
  ```
Use "sudo canasta restic [command] --help" for more information about a command.

## /etc/canasta/conf.json
* Canasta CLI maintains a list of installations that it manages. This information is stored at /etc/canasta/conf.json. Therefore the CLI would require permissions to read and write to the `/etc/canasta/` folder.
* The scheme of the `conf.json` file is as follows
```
{
	"Installations": {
		"wiki1": {
			"Id": "wiki1",
			"Path": "/home/user/wiki1",
			"Orchestrator": "docker-compose"
		},
		"wiki2": {
			"Id": "wiki2",
			"Path": "/home/user/canasta/wiki2",
			"Orchestrator": "docker-compose"
		}
	}
}
```
* Inside the object `Installations` there is the list of installations that the CLI currently manages.
* "Id": It is the Canasta ID of the installation, specified during the installation.
* "Path": This is the directory where the configuration files for the Canasta installation is stored.
* "Orchestrator": This is the orchestrator which runs the Canasta instance. (Currently docker-compose is the only orchestrator supported by the CLI)

## Uninstall
* To uninstall the CLI, delete the binary file from the installation folder (default: /usr/local/bin/canasta)
```
sudo rm /usr/local/bin/canasta && sudo rm /etc/canasta/conf.json
```
* Note: The argument "-i canastaId" is not necessary for any command when the command is run from the Canasta installation directory.
