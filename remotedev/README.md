# Create a remote Zulip dev server

This guide is for mentors who want to help create remote Zulip dev servers
for GCI participants.

This year, [Digital Ocean](https://www.digitalocean.com/) is providing virtual
machines (droplets) to help GCI participants get up and running as easily as
possible. Thank you Digital Ocean!

The `create.py` create uses the Digital Ocean API to quickly create new virtual
machines (droplets) with the Zulip dev server already configured.

## Requirements

The create script requires **python3** and the [python-digitalocean
module][python-digitalocean].

Follow your operating system's instructions for installing python 3.

Next, install [python-digitalocean][python-digitalocean] with `pip`:

```
$ pip install python-digitalocean==1.10.1
```

Note: If you have python 2 and python 3 installed on your system, you might
need to use `pip3` instead.

The `1.10.1` version is explicitly specified because later versions of the
module have caused issues.

## Step 1: Fork & clone zulip/zulip-gci

If you haven't already, fork and clone [zulip/zulip-gci][zulip-zulip-gci].

The create script and configuration file live in the **remotedev** directory.

## Step 2: Join Zulip Digital Ocean team

We have created a team on Digital Ocean for Zulip mentors. Ask Tim or Christie
to be added. You need access to the team so you can create your Digital Ocean
API token.

## Step 3: Create your Digital Ocean API token

Once you've been added to the Zulip team,
[login](https://cloud.digitalocean.com/droplets) to the Digital Ocean control
panel and [create your personal API token][do-create-api-token]. **Make sure
you create your API token under the Zulip team.** (It should look something
like [this][image-zulip-team]).

Copy the API token and store it somewhere safe. You'll need it in the next
step.

## Step 4: Configure create.py

In `remotedev` there is a sample configuration file `conf.ini-template`.

Copy this file to `conf.ini`:

```
$ cp conf.ini-template conf.ini
```

Now edit the file and replace `APITOKEN` with the personal API token you
generated earlier.

```
[digitalocean]
api_token = APITOKEN
```

Now you're ready to use the script.

## Usage

`create.py` takes one argument: a GitHub username.

```
$ python3 create.py <username>
```

In order for the script to work, the GitHub user must have:

- forked the [zulip/zulip][zulip-zulip] repository, and
- created an ssh key pair and added it to their GitHub account.

(Share [this link][how-to-request] with students if they need to do these
steps.)

The script will stop if it can't find the user's fork or ssh keys.

The script will also stop if a droplet has already been created for the user.
If you need to re-create a droplet, login to Digital Ocean with your browser
and delete **both** the **droplet** and its **dns entry**.

Once the droplet is created, you will see something similar to this message:

```
Your remote Zulip dev server has been created!

- Connect to your server by running
  `ssh zulipdev@<username>.zulipdev.org` on the command line
  (Terminal for macOS and Linux, Bash for Git on Windows).
- There is no password; your account is configured to use your ssh keys.
- Once you log in, you should see `(zulip-venv) ~$`.
- To start the dev server, `cd zulip` and then run `./tools/run-dev.py`.
- While the dev server is running, you can see the Zulip server in your browser
  at http://<username>.zulipdev.org:9991.

See [Developing
remotely](http://zulip.readthedocs.io/en/latest/dev-remote.html) for tips on
using the remote dev instance and [Git & GitHub
Guide](http://zulip.readthedocs.io/en/latest/git-guide.html) to learn how to
use Git with Zulip.
```

Copy and paste this message to the user via Zulip chat. Be sure to CC the user
so they are notified.

[do-create-api-token]: https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2#how-to-generate-a-personal-access-token
[image-zulip-team]: http://cdn.subfictional.com/dropshare/Screen-Shot-2016-11-28-10-53-24-X86JYrrOzu.png
[zulip-zulip-gci]: https://github.com/zulip/zulip-gci
[zulip-zulip]: https://github.com/zulip/zulip
[python-digitalocean]: https://github.com/koalalorenzo/python-digitalocean
[how-to-request]: https://github.com/zulip/zulip-gci/blob/master/request-remote-dev.md

## Updating the base image

Rough steps:

1. Get the `ssh` key for `base.zulipdev.org` from Christie or Rishi.
1. Power up the `base.zulipdev.org` droplet from the digitalocean UI. You
   probably have to be logged in in the Zulip organization view, rather than
   via your personal account.
1. `ssh zulipdev@base.zulipdev.org`
1. `git pull upstream master`
1. `tools/provision`
1. `tools/run-dev.py`, and then Ctrl-C (to clear out anything in the Rabbit MQ queue, load messages, etc).
1. `tools/run-dev.py`, and check that `base.zulipdev.org:9991` is up and running.
1. `history -c` to clear any command line history, if you made a typo (to reduce chance of confusing new contributors).
1. `sudo shutdown -h now`
1. Go to the Images tab on DigitalOcean, and "Take a Snapshot".
1. Wait for several minutes.
1. Make sure to add the appropriate regions via More -> "Add to region" in the Snapshots section.
1. Do something like `curl -X GET -H "Content-Type: application/json" -u <API_KEY>: "https://api.digitalocean.com/v2/images?page=5" | grep --color=always base.zulipdev.org` (maybe with a different page number, and replace your API_KEY).
1. Replace `template_id` in `create.py` in this directory with the appropriate `id`, and region with the appropriate region.
1. Test and push to zulip/zulip-gci!
