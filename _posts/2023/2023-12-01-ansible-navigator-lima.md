---
title: "Running Ansible Navigator on MacOS without Docker Desktop"
classes: wide
categories:
  - Blog
tags:
  - Ansible
  - Red Hat
  - Automation
  - installation 
  - Ansible Navigator
  - Ansible Execution Environments
  - Ansible Automation Platform
---

"[Ansible Navigator](https://ansible.readthedocs.io/projects/navigator/) is a command-line tool and a text-based user interface (TUI) for creating, reviewing, running and troubleshooting Ansible content, including inventories, playbooks, collections, documentation and container images (execution environments)."

Currently Ansible Navigator for MacOS [requires Docker Desktop](https://ansible.readthedocs.io/projects/navigator/installation/#requirements-macos) and doesn't support Podman due to other technical contraints on how podman handles mounting native folders on MacOS. At the time of writing this blog there is an open issue in the ansible-navigator repo for podman support [issue 1259](https://github.com/ansible/ansible-navigator/issues/1259). Like me, if you're unable to use Docker Desktop you may be looking for an alternative way to run Docker containers on MacOS, and with a general search you'll find approaches such as using Lima or Vagrant. [Lima](https://github.com/lima-vm/lima), which stands for Linux on Mac, is a lightweight solution for running Linux VMs on MacOS, specifically with containerized applications, although it can also be used for non-container applications too. 

## Installing Lima
I used this [blog post](https://blog.carlosnunez.me/post/docker-desktop-alternative-for-mac/) by Carlos Nunez as a guide for installing Lima on my laptop.

I'll include a summary of the commands that I ran, but I'd recommend following Carlos's blog for a detailed tutorial 

```
╰─ brew install lima docker


╰─ limactl --version
  limactl version 0.19.0

╰─ mkdir ~/lima_machines

╰─ curl -sSLo ~/lima_machines/docker.yaml \
  https://raw.githubusercontent.com/carlosonunez/bash-dotfiles/main/lima_machine.yaml
```

Edit the dockerfile as stated by the instructions in the file. 
Specifically: 

```
  # NOTE: Replace %TEMPDIR% with your actual temporary directory as provided by
  # the $TMPDIR environment variable.
  - location: "%TEMPDIR%"
    writable: true
```

to which mine resolved to 

```
╰─ echo $TMPDIR
  /var/folders/9p/5rymmxbs59d1svryg_7knth80000gn/T/
```

We're going to come back to the section in the file later as, we'll need to update the mounts to include a new directory when we start using Ansible Navigator.  

Next start the Docker VM:
```
╰─ limactl start ~/lima_machines/docker.yaml --tty=false
```

Perform a final export as mentioned in Carlos's blog and then we're good to go. 
```
╰─ export DOCKER_HOST="unix://$HOME/.lima/docker.sock"

╰─ docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```
Note, we'll need to add this export to ~/.zprofile, which was not mentioned Carlos's blog. 

At this point we are now successfully running Docker on MacOS without Docker Desktop. 


## Installing Ansible Navigator
Next, we're going to now install `ansible-navigator` and you can find the documented [instructions here.](https://ansible.readthedocs.io/projects/navigator/installation/#install-ansible-navigator-macos)

I'll include a screenshot here just for posterity to have a record of the current instructions at the time of writing this blog. 

<figure>
<a href="/assets/images/ansible-navigator/installation-instructions.png"><img src="/assets/images/ansible-navigator/installation-instructions.png"></a>
</figure>

Ensure that you export the appropriate `ansible-navigator` installation path as reported in the output of the installation.

```
╰─ echo 'export PATH=/Users/bpanyar/Library/Python/3.12/bin:$PATH' >> ~/.zprofile

╰─ source ~/.zprofile
```

After that you should be able to launch ansible-navigator:

```
╰─ ansible-navigator 

  ---------------------------------------------------------------------
  Execution environment image and pull policy overview
  ---------------------------------------------------------------------
  Execution environment image name:     ghcr.io/ansible/creator-ee:v0.20.0
  Execution environment image tag:      v0.20.0
  Execution environment pull arguments: None
  Execution environment pull policy:    tag
  Execution environment pull needed:    True
  ---------------------------------------------------------------------
  Updating the execution environment
  ---------------------------------------------------------------------
  Running the command: docker pull ghcr.io/ansible/creator-ee:v0.20.0
  v0.20.0: Pulling from ansible/creator-ee
  1e08c7e6aff8: Pull complete 
  ad97261d044b: Pull complete 
  7b05cec1a1b5: Pull complete 
  eaac7334e7ab: Pull complete 
  df17b198186c: Pull complete 
  6a4437d12ad5: Pull complete 
  90e5f6bd07c8: Pull complete 
  65aa82c7a1f1: Pull complete 
  897151f83099: Pull complete 
  19f93933b64e: Pull complete 
  8038a525cde8: Pull complete 
  11c3d85c8b64: Pull complete 
  d29defd3b078: Pull complete 
  Digest: sha256:b496e72a7b581fc8f437fab84d2e7f59c8a4e4ae8191d4c2478dd151c871cf1c
  Status: Downloaded newer image for ghcr.io/ansible/creator-ee:v0.20.0
  ghcr.io/ansible/creator-ee:v0.20.0
```

```
 0│Welcome
 1│————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————
 2│
 3│Some things you can try from here:
 4│- :collections                                    Explore available collections
 5│- :config                                         Explore the current ansible configuration
 6│- :doc <plugin>                                   Review documentation for a module or plugin
 7│- :help                                           Show the main help page
 8│- :images                                         Explore execution environment images
 9│- :inventory -i <inventory>                       Explore an inventory
10│- :log                                            Review the application log
11│- :lint <file or directory>                       Lint Ansible/YAML files (experimental)
12│- :open                                           Open current page in the editor
13│- :replay                                         Explore a previous run using a playbook artifact
14│- :run <playbook> -i <inventory>                  Run a playbook in interactive mode
15│- :settings                                       Review the current ansible-navigator settings
16│- :quit                                           Quit the application
17│
18│happy automating,
19│
20│-winston
```

## Inspecting Execution Environment Image details
Going into the `:images` option you can see there is a default image called `creator-ee` which we'll use to test running an initial playbook.

```
  Image                       Tag                 Execution environment                                   Created                           Size
0│creator-ee                  v0.20.0             True                                                    2 months ago                      843MB
1│qus                         latest              False                                                   7 months ago                      306MB
```

Inspecting the `creator-ee` image with option `0` lists a number of options of details that you can inspect about the image. By selecting option `1` you should see the following error:

```
  Warning
  ─────────────────────────────────────────────────────────────────────
  humph. Something went really wrong while introspecting the image.
  Details have been added to the log file
  [HINT] Please log an issue about this one, it shouldn't have happened
  ─────────────────────────────────────────────────────────────────────
                                                                    Ok 
```

There will be a `ansible-navigator.log` file generated in your working directory which should show something along the lines of:

```
2023-10-27T21:08:07.304631+00:00 ERROR 'ansible_navigator.actions.images._parse' Unable to extract introspection from stdout
2023-10-27T21:08:07.305286+00:00 ERROR 'ansible_navigator.actions.images._parse' Image introspection failed (parsed), the return value was: /usr/bin/python3: can't open file '/Users/bpanyar/.cache/ansible-navigator/image_introspect.py': [Errno 2] No such file or directory
```

Which tells us that the reported `~/.cache` directory is not accessible to the EE container. This is where we are going to go back to the `docker.yaml` file from earlier and update the mount section to incude this directory. We can either update the file we used previously at `~/lima_machines/docker.yaml` or update  `~/.lima/lima.yaml` directly. If you choose to update the previous `docker.yaml` file, you need to copy it to `~/.lima/lima.yaml`. 

My `mounts:` section looks like this with the `~/.cache` directory added to the bottom:

```
mounts:
  - location: "~/src"
    writable: true
  - location: "~/tmp"
    writable: true
  # NOTE: Replace %TEMPDIR% with your actual temporary directory as provided by
  # the $TMPDIR environment variable.
  - location: "/var/folders/9p/5rymmxbs59d1svryg_7knth80000gn/T/"
    writable: true
  - location: "/tmp/lima"
    writable: true
  - location: "~/.config"
    writable: true
  - location: "~/.lima"
    writable: false
  - location: "~/.kube"
    writable: true
  - location: "~/.ssh"
    writable: false
  - location: "~/.gnupg"
    writable: false
  - location: "~/.cache"
    writable: false
  ```

Next, we need to restart the Docker VM with:

```
limactl stop docker && limactl start docker
```

After restarting Docker and re-launching `ansible-navigator`, we can now inspect all the image details of the `creator-ee` image. 


## Running a playbook with the sample EE. 


For this I created a simple ping playbook which just pings localhost. 

inventory.yml
```
systems:
  hosts:
    localhost:
      ansible_host: localhost
      ansible_connection: local
```

ping.yml
```
---
- hosts: all
  gather_facts: false

  tasks:
    - name: run ping
      ansible.builtin.ping:
      register: result

    - debug:
        var: result
```

To run the playbook with the default EE, I'll change to the directory where this playbook is located and run the following command:

```
ansible-navigator run ping.yml -i inventory.yml
```
Note, by default `ansible-navigator` is configured to use the `creator-ee`. When creating your own execution environments you can use the `--eei` argument to specify the name of the execution environment image to use. 

Immediately, you should see that the playbook runs without completing any tasks

```
  Warning
  ────────────────────────────────────────────────────────────────────────────
  The playbook completed without tasks. Redirecting to ':stdout' for review.
  ────────────────────────────────────────────────────────────────────────────
                                                                         Ok 
```

And by pressing enter you should see something like:

```
0│ERROR! the playbook: /Users/bpanyar/Downloads/navigator-test/ping.yml could not be found
```

Again, this is due to the EE container not being able to access the working directory where our playbook is located, so we'll need to go back and update the `mounts:` section in the `~/.lima/lima.yaml` file and restart Docker again.

after doing that and re-running `ansible-navigator run ping.yml -i inventory.yml` you should see it execute successfully and be able to see that it completed 2 tasks and drill down into the play for more details.

```
  Play name        Ok   Changed       Unreachable          Failed      Skipped       Ignored       In progress          Task count                 Progress
0│all               2         0                 0               0            0             0                 0                   2                 Complete
```
Option 0

```
  Result         Host                  Number         Changed          Task               Task action                                   Duration
0│Ok             localhost                  0         False            run ping           ansible.builtin.ping                                0s
1│Ok             localhost                  1         False            debug              debug                                               0s
```
You can see the results of each task by entering its associated option number.

```
Play name: all:0
Task name: run ping
Ok: localhost                                                                                                                                               
 0│---
 1│duration: 0.18527
 2│end: '2023-12-01T21:49:57.495027'
 3│event_loop: null
 4│host: localhost
 5│play: all
 6│play_pattern: all
 7│playbook: /Users/bpanyar/Downloads/navigator-test/ping.yml
 8│remote_addr: localhost
 9│res:
10│  _ansible_no_log: null
11│  ansible_facts:
12│    discovered_interpreter_python: /usr/bin/python3
13│  changed: false
14│  invocation:
15│    module_args:
16│      data: pong
17│  ping: pong
18│resolved_action: ansible.builtin.ping
19│start: '2023-12-01T21:49:57.309757'
20│task: run ping
21│task_action: ansible.builtin.ping
22│task_args: ''
23│task_path: /Users/bpanyar/Downloads/navigator-test/ping.yml:6
```

Also notice that there is a json file generated in the working directory for every run of playbook. This is a playbook artifact that contains details about the `ansible-navigator` settings along with the play output. This can be inspected manually, or replayed in `ansible-navigator` using the `replay` argument. This is especially helpful for sharing your playbook output with other or debugging someone elses play when you can't recreate it in your own environment. 

Note, there seems to be a bug with `ansible-navigator` and you may need to rename your playbook artifact to be able to replay it. The artifact is created with `/` in the file name and the replay option interprets them as child directories thus is not able to find the artifact. 

```
╰─ ansible-navigator replay ping-artifact-2023-12-01T21/49/57.821234+00/00.json  
Warning: Issues were found while applying the settings.
   Hint: Command provided: 'replay ping-artifact-2023-12-01T21/49/57.821234+00/00.json'

  Error: The specified playbook artifact could not be found:
         /Users/bpanyar/Downloads/navigator-test/ping-artifact-2023-12-01T21/49/57.821234+00/00.json
   Hint: Try again with 'replay <valid path to playbook artifact>'

   Note: Configuration failed, using default log file location. (/Users/bpanyar/Downloads/navigator-test/ansible-navigator.log) Log
         level set to debug
   Hint: Review the hints and log file to see what went wrong.
```

## Conclusion
In summary, its definitely possible to use `ansible-navigator` without Docker Desktop, with a small amount of effort and some minor technicalities to be aware of. While its not an officially supported installation path, it seems to do the job for building and executing Ansible Execution Environments for development purposes. In this blog I only covered using the default Execution Environment included with Ansible Navigator, but in a future blog I'll cover creating your own Execution Environments using `ansible-builder`.

### Tips
- Directories will need to be explicitely mounted for `ansible-navigator` and the EEs to accesss, so you may choose to mount a parent directory that holds all of your Ansible playbooks. 
