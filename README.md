# iCloud Photos Downloader for DSM

* A command-line tool to download all your iCloud photos.
* Works on Mac, Linux, Windows, and Synology DSM.
* Run as a scheduled task in Synology to keep a local backup of your photos and videos.

### Installation to Synology DSM

    # Clone the repo somewhere
    git clone https://github.com/skarppi/icloud_photo_station.git
    cd icloud_photo_station

> If you need to install Python, see the [Requirements](#requirements) section for instructions.

    # Create a SPK installation package containing virtualenv, python scripts and all necessary dependencies.

    cd spk
    sh build.sh

Manually install resulting [icloud_photo_station-0.2.0.spk](https://github.com/skarppi/icloud_photo_station/releases/download/0.2.0/icloud_photo_station-0.2.0.spk) in your DSM `Package Station`. Now you can set up `User-defined script` into `Task Scheduler` and set up scheduling and notification emails for script output.

    source /volume1/@appstore/icloud_photo_station/env/bin/activate
    python /volume1/@appstore/icloud_photo_station/app/icloudpd.py \
        --username '<YOUR ICLOUD USERNAME>' \
        --password '<YOUR ICLOUD PASSWORD>' \
        --auto-delete \
        --until-found 10 \
        --download output

If your iCloud account has two-factor authentication enabled, SSH to Synology box and run the script manually first time in order to input the verification code.

## Usage

    $ icloudpd <download_directory>
               --username <username>
               [-p, --password <password>]
               [-d, --directory <directory>]
               [--cookie-directory </cookie/directory>]
               [--size (original|medium|thumb)]
               [--live-photo-size (original|medium|thumb)]
               [--recent <integer>]
               [--until-found <integer>]
               [-a, --album <album>]
               [-l, --list-albums]
               [--skip-videos]
               [--skip-live-photos]
               [--force-size]
               [--auto-delete]
               [--only-print-filenames]
               [--folder-structure ({:%Y/%m/%d})]
               [--set-exif-datetime]
               [--smtp-username <smtp_username>]
               [--smtp-password <smtp_password>]
               [--smtp-host <smtp_host>]
               [--smtp-port <smtp_port>]
               [--smtp-no-tls]
               [--notification-email <notification_email>]
               [--notification-script PATH]
               [--log-level (debug|info|error)]
               [--no-progress-bar]

    Options:
        --username <username>           Your iCloud username or email address
        --password <password>           Your iCloud password (default: use PyiCloud
                                        keyring or prompt for password)
        --cookie-directory </cookie/directory>
                                        Directory to store cookies for
                                        authentication (default: ~/.pyicloud)
        --size [original|medium|thumb]  Image size to download (default: original)
        --live-photo-size [original|medium|thumb]
                                        Live Photo video size to download (default:
                                        original)
        --recent INTEGER RANGE          Number of recent photos to download
                                        (default: download all photos)
        --until-found INTEGER RANGE     Download most recently added photos until we
                                        find x number of previously downloaded
                                        consecutive photos (default: download all
                                        photos)
        -a, --album <album>             Album to download (default: All Photos)
        -l, --list-albums               Lists the avaliable albums
        --skip-videos                   Don't download any videos (default: Download
                                        both photos and videos)
        --skip-live-photos              Don't download any live photos (default:
                                        Download live photos)
        --force-size                    Only download the requested size (default:
                                        download original if size is not available)
        --auto-delete                   Scans the "Recently Deleted" folder and
                                        deletes any files found in there. (If you
                                        restore the photo in iCloud, it will be
                                        downloaded again.)
        --only-print-filenames          Only prints the filenames of all files that
                                        will be downloaded. (Does not download any
                                        files.)
        --folder-structure <folder_structure>
                                        Folder structure (default: {:%Y/%m/%d})
        --set-exif-datetime             Write the DateTimeOriginal exif tag from
                                        file creation date, if it doesn't exist.
        --smtp-username <smtp_username>
                                        Your SMTP username, for sending email
                                        notifications when two-step authentication
                                        expires.
        --smtp-password <smtp_password>
                                        Your SMTP password, for sending email
                                        notifications when two-step authentication
                                        expires.
        --smtp-host <smtp_host>         Your SMTP server host. Defaults to:
                                        smtp.gmail.com
        --smtp-port <smtp_port>         Your SMTP server port. Default: 587 (Gmail)
        --smtp-no-tls                   Pass this flag to disable TLS for SMTP (TLS
                                        is required for Gmail)
        --notification-email <notification_email>
                                        Email address where you would like to
                                        receive email notifications. Default: SMTP
                                        username
        --notification-script PATH      Runs an external script when two factor
                                        authentication expires. (path required:
                                        /path/to/my/script.sh)
        --log-level [debug|info|error]  Log level (default: debug)
        --no-progress-bar               Disables the one-line progress bar and
                                        prints log messages on separate lines
                                        (Progress bar is disabled by default if
                                        there is no tty attached)
        --version                       Show the version and exit.
        -h, --help                      Show this message and exit.

Example:

    $ icloudpd ./Photos \
        --username testuser@example.com \
        --password pass1234 \
        --recent 500 \
        --auto-delete

## Requirements

- Python 2.7 or Python 3.4+
  - _Python 2.6 is not supported._
- pip

### Install Python & pip

#### Windows

- [Download Python 3.7.0](https://www.python.org/ftp/python/3.7.0/python-3.7.0.exe)

#### Mac

- Install [Homebrew](https://brew.sh/) (if not already installed):

```
which brew > /dev/null 2>&1 || /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- Install Python (includes `pip`):

```
brew install python
```

> Alternatively, you can [download the Python 3.7.0 installer for Mac](https://www.python.org/ftp/python/3.7.0/python-3.7.0-macosx10.9.pkg).

#### Linux (Ubuntu)

```
sudo apt-get update
sudo apt-get install -y python
```

## Authentication

If your Apple account has two-factor authentication enabled,
you will be prompted for a code when you run the script.

Two-factor authentication will expire after an interval set by Apple,
at which point you will have to re-authenticate. This interval is currently two months.

Authentication cookies will be stored in a temp directory (`/tmp/pyicloud` on Linux, or `/var/tmp/...` on MacOS.) This directory can be configured with the `--cookie-directory` option.

You can receive an email notification when two-factor authentication expires by passing the
`--smtp-username` and `--smtp-password` options. Emails will be sent to `--smtp-username` by default,
or you can send to a different email address with `--notification-email`.

If you want to send notification emails using your Gmail account, and you have enabled two-factor authentication, you will need to generate an App Password at https://myaccount.google.com/apppasswords

## Error on first run

When you run the script for the first time, you might see an error message like this:

```
Bad Request (400)
```

This error often happens because your account hasn't used the iCloud API before, so Apple's servers need to prepare some information about your photos. This process can take around 5-10 minutes, so please wait a few minutes and try again.

If you are still seeing this message after 30 minutes, then please [open an issue on GitHub](https://github.com/ndbroadbent/icloud_photos_downloader/issues/new) and post the script output.

## Cron Task

Follow these instructions to run `icloudpd` as a scheduled cron task.

```
# Clone the git repo somewhere
git clone https://github.com/ndbroadbent/icloud_photos_downloader.git
cd icloud_photos_downloader

# Copy the example cron script
cp cron_script.sh.example cron_script.sh
```

- Update `cron_script.sh` with your username, password, and other options

- Edit your "crontab" with `crontab -e`, then add the following line:

```
0 */6 * * * /path/to/icloud_photos_downloader/cron_script.sh
```

Now the script will run every 6 hours to download any new photos and videos.

> If you provide SMTP credentials, the script will send an email notification
> whenever two-step authentication expires.

## Docker

This script is available in a Docker image: `docker pull ndbroadbent/icloudpd`

Usage:

```bash
# Downloads all photos to ./Photos

$ docker pull ndbroadbent/icloudpd
$ docker run -it --rm --name icloud -v $(pwd)/Photos:/data ndbroadbent/icloudpd:latest \
    icloudpd /data \
    --username testuser@example.com \
    --password pass1234 \
    --size original \
    --recent 500 \
    --auto-delete
```

## Contributing

Install dependencies:

```
sudo pip install -r requirements.txt
sudo pip install -r requirements-test.txt
```

Run tests:

```
pytest
```

Before submitting a pull request, please check the following:

- All tests pass on Python 2.7 and 3.6
  - Run `./scripts/test`
- 100% test coverage
  - After running `./scripts/test`, you will see the test coverage results in the output
  - You can also open the HTML report at: `./htmlcov/index.html`
- Code is formatted with [autopep8](https://github.com/hhatto/autopep8)
  - Run `./scripts/format`
- No [pylint](https://www.pylint.org/) errors
  - Run `./scripts/lint` (or `pylint icloudpd`)
- If you've added or changed any command-line options,
  please update the [Usage](#usage) section in the README.

If you need to make any changes to the `pyicloud` library,
`icloudpd` uses a fork of this library that has been renamed to `pyicloud-ipd`.
Please clone my [pyicloud fork](https://github.com/ndbroadbent/pyicloud)
and check out the [pyicloud-ipd](https://github.com/ndbroadbent/pyicloud/tree/pyicloud-ipd)
branch. PRs should be based on the `pyicloud-ipd` branch and submitted to
[ndbroadbent/pyicloud](https://github.com/ndbroadbent/pyicloud).

### Building the Docker image:

```
$ git clone https://github.com/ndbroadbent/icloud_photos_downloader.git
$ cd icloud_photos_downloader/docker
$ docker build -t ndbroadbent/icloudpd .
```

### TODO

* Store photos in their albums and use default date structure YYYY/MM/DD as fallback if photo doesn't belong to any album
* Use multiple PhotoStation destinations
  * E.g. sync by default to global PhotoStation
  * Allow moving some photos to Personal PhotoStation so that they doesn't appear back to the default PhotoStation
* Enable easier way to enter two-factor authentication (2FA) code without need to SSH to your Synology box
