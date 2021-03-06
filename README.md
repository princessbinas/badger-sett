# Badger Sett

> A *sett* or set is a badger's den which usually consists of a network of tunnels
  and numerous entrances. Setts incorporate larger chambers used for sleeping or
  rearing young.

This script is designed to raise young [Privacy Badgers](https://github.com/EFForg/privacybadger) by teaching them
about the trackers on popular sites. Every day, [`crawler.py`](./crawler.py) visits thousands of the top sites from the [Tranco List](https://tranco-list.eu) with the latest version of Privacy Badger, and saves its findings in `results.json`.

See the following EFF.org blog post for more information: [Giving Privacy Badger a Jump Start](https://www.eff.org/deeplinks/2018/08/giving-privacy-badger-jump-start).

## Setup

0. Prerequisites: have [docker](https://docs.docker.com/install/) installed.
   Make sure your user is part of the `docker` group so that you can build and
   run docker images without `sudo`. You can add yourself to the group with

   ```
   $ sudo usermod -aG docker $USER
   ```

1. Clone the repository

   ```
   $ git clone https://github.com/efforg/badger-sett
   ```

2. Run a scan

   ```
   $ ./runscan.sh
   ```

   This will run a scan with the latest version of Privacy Badger's master branch
   and won't commit the results.

   To run the script with a different branch of privacy badger, set the `PB_BRANCH`
   variable. e.g.

   ```
   $ PB_BRANCH=my-feature-branch ./runscan.sh
   ```

   You can also pass arguments to `crawler.py`, the python script that does the
   actual crawl. Any arguments passed to `runscan.sh` will be forwarded to
   `crawler.py`. To control the number of sites that the crawler visits, use the
   `--n-sites` argument (the default is 2000). For example:

   ```
   $ ./runscan.sh --n-sites 10
   ```
   
   To exclude any sites with a given top level domain from the scan, pass in
   the `--exclude` argument followed by the TLD you want to exclude. For example,
   if you wanted to exclude all sites with a .gov TLD:

   ```
   $ ./runscan.sh --exclude gov
   ```
   To exclude multiple TLD's from a scan, pass in each TLD separated by a comma,
   with no space between. For example, if you wanted to exclude all sites with
   .org and .net TLD:
   ```
   $ ./runscan.sh --exclude org,net
   ```

   You can load another extension to run in parallel to Privacy Badger during a scan.
   Use the `--load-extension` flag and pass along the filepath for the `.crx` or `.xpi`
   file that you want to load. For example:
   ```
   $ ./runscan.sh --load-extension parallel-extensions/ublock.crx
   ```

3. Monitor the scan

   To have the scan print verbose output about which sites it's visiting, use
   the `--log-stdout` argument.

   If you don't use that argument, all output will still be logged to
   `docker-out/log.txt`, beginning after the script outputs "Running scan in
   Docker..."

### Automatic crawling

To set up the script to run periodically and automatically update the
repository with its results:

1. Create a new ssh key with `ssh-keygen`. Give it a name unique to the
   repository.

   ```
   $ ssh-keygen
   Generating public/private rsa key pair.
   Enter file in which to save the key (/home/USER/.ssh/id_rsa): /home/USER/.ssh/id_rsa_badger_sett
   ```

2. Add the new key as a deploy key with R/W access to the repo on Github.
   https://developer.github.com/v3/guides/managing-deploy-keys/

3. Add a SSH host alias for Github that uses the new key pair. Create or open
   `~/.ssh/config` and add the following:

   ```
   Host github-badger-sett
     HostName github.com
     User git
     IdentityFile /home/USER/.ssh/id_rsa_badger_sett
   ```

4. Configure git to connect to the remote over SSH. Edit `.git/config`:

   ```
   [remote "origin"]
     url = ssh://git@github-badger-sett:/efforg/badger-sett
   ```

   This will have `git` connect to the remote using the new SSH keys by default.

5. Create a cron job to call `runscan.sh` once a day. Set the environment
   variable `RUN_BY_CRON=1` to turn off TTY forwarding to `docker run` (which
   would break the script in cron), and set `GIT_PUSH=1` to have the script
   automatically commit and push `results.json` when the scan finishes. Here's an
   example `crontab` entry:

   ```
   0 0 * * *  RUN_BY_CRON=1 GIT_PUSH=1 /home/USER/badger-sett/runscan.sh
   ```

6. If everything has been set up correctly, the script should push a new version
   of `results.json` after each crawl. Soon, whenever you `make` a new version of
   Privacy Badger, it will pull the latest version of the crawler's data and
   ship it with the new version of the extension.
