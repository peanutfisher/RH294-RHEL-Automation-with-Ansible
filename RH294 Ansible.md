# RH294 Red Hat Enterprise Linux Automation with Ansible

---

Start time: 2024/07/19

---

## Lab Env

![image-20240720111819144](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\image-20240720111819144.png)

In this course, the main computer system used for hands-on learning activities is `workstation`. Four other machines are also used by students for these activities: `servera`, `serverb`, `serverc`, and `serverd`. All these five systems are in the `lab.example.com` DNS domain.

**User/passwd**: `student`/ `student`. AND `root`/ `redhat`.



**Table 1. Classroom Machines**

| Machine name                  | IP addresses     | Role                                                         |
| :---------------------------- | :--------------- | :----------------------------------------------------------- |
| `bastion.lab.example.com`     | `172.25.250.254` | Gateway system to connect student private network to classroom server (must always be running) |
| `utility.lab.example.com`     | `172.25.250.8`   | System with utility services required for the classroom      |
| `workstation.lab.example.com` | `172.25.250.9`   | Graphical workstation used for system administration         |
| `servera.lab.example.com`     | `172.25.250.10`  | Host managed with Ansible                                    |
| `serverb.lab.example.com`     | `172.25.250.11`  | Host managed with Ansible                                    |
| `serverc.lab.example.com`     | `172.25.250.12`  | Host managed with Ansible                                    |
| `serverd.lab.example.com`     | `172.25.250.13`  | Host managed with Ansible                                    |

The primary function of `bastion` is that it acts as a router between the network that connects the student machines and the classroom network.

Several systems in the classroom provide supporting services. Two servers, `content.example.com` and `materials.example.com`, are sources for software and lab materials used in hands-on activities. These are provided by the `classroom.example.com` virtual machine. Both `classroom` and `bastion` should always be running for proper use of the lab environment.

---

## Chapter 1. Introducing Ansible


### Ansible Concepts and Architecture

**The Ansible architecture consists of two types of machines: *control nodes* and *managed hosts***. Ansible is installed and run from a control node, and this machine also has copies of your Ansible project files.

Managed hosts are listed in an *inventory*, which also organizes those systems into groups for easier collective management. You can define the inventory statically in a text file, or dynamically using scripts that obtain group and host information from external sources.

Instead of writing complex scripts, Ansible users create high-level *plays* to ensure that a host or group of hosts is in a particular state. **A play performs a series of *tasks* on the hosts, in the order specified by the play. These plays are expressed in YAML format in a text file. A file that contains one or more plays is called a *playbook*.**

Each task runs a *module*, a small piece of code (written in Python, PowerShell, or some other language), with specific arguments. Each module is essentially a tool in your toolkit. Ansible ships with hundreds of useful modules that can perform a wide variety of automation tasks. They can act on system files, install software, or make API calls.

When used in a task, a module generally ensures that some particular aspect of the machine is in a particular state. For example, a task using one module might ensure that a file exists and has particular permissions and content. A task using a different module might ensure that a particular file system is mounted. If the system is not in that state, the task should put it in that state, or do nothing. If a task fails, the default Ansible behavior is to abort the rest of the playbook for the hosts that had a failure and continue with the remaining hosts.

Tasks, plays, and playbooks are designed to be *idempotent*. This means that you can safely run a playbook on the same hosts multiple times. When your systems are in the correct state, the playbook makes no changes when you run it. Numerous modules are available that you can use to run arbitrary commands. However, you must use those modules with care to ensure that they run in an idempotent way.

Ansible also uses *plug-ins*. Plug-ins are code that you can add to Ansible to extend it and adapt it to new uses and platforms.

**The Ansible architecture is agentless. Typically, when an administrator runs an Ansible Playbook, the control node connects to the managed host by using SSH (by default) or WinRM.** This means that you do not need to have an Ansible-specific agent installed on managed hosts, and do not need to permit any additional communication between the control node and managed hosts.



### Ansible Source

- **Community Ansible**: *Ansible Core* / *community Ansible*(包含开源社区Ansible modules and roles)
- **Ansible Core in Red Hat Enterprise Linux**(RPM package, `ansible-core`, included with Red Hat Enterprise Linux 9 in the AppStream repository.)
- **Red Hat Ansible Automation Platform** (subscription, Ansible Core toolset plus additional certified and supported content, tools, components, and cloud services)



### **Red Hat Ansible Automation Platform 2**

- Ansible Core(Red Hat Ansible Automation Platform 2.2 provides Ansible Core 2.13 in the `ansible-core` RPM package and in its `ee-minimal-rhel8` and `ee-supported-rhel8` automation execution environments.)

- Ansible Content Collections

- Automation Content Navigator (`ansible-navigator` running your playbooks in a container )

- Automation Execution Environments(a container image that contains Ansible Core, Ansible Content Collections, and any Python libraries, executables, or other dependencies needed to run your playbook.)

- Automation Controller (Red Hat Ansible Tower, web UI and a REST API that can be used to configure, run, and evaluate your automation jobs)

- Automation Hub (`console.redhat.com` provides access to Red Hat Certified Ansible Content Collections)

![image-20240720170930324](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\image-20240720170930324.png)

### Prepare Control Node

- If use your control node as the execution environment :

  ```
  [user@controlnode ~]$ sudo dnf install ansible-core
  ```

  

- If container: install ansible-navigator on control node

  - Install automation content navigator on your control nodes.

  ```
  [user@controlnode ~]$ sudo dnf install ansible-navigator
  ```

  - Verify that automation content navigator is installed on the system.

  ```
  [user@controlnode ~]$ ansible-navigator --version
  ansible-navigator 2.1.0
  ```

  - Log in to the container registry.

  ```
  [user@controlnode ~]$ podman login registry.redhat.io
  Username: your-registry-username
  Password: your-registry-password
  Login Succeeded!
  ```

  - Download the container image for the execution environment that you plan to use with automation content navigator. ( `ansible-navigator images`might also automatically download the default execution environment when you run the command.)

  ```
  [user@controlnode ~]$ podman pull \
  > registry.redhat.io/ansible-automation-platform-22/ee-supported-rhel8:latest
  ```

  - Display the list of locally available container images to verify that the image was downloaded.

  ```
  [user@controlnode ~]$ ansible-navigator images
    Image                    Tag      Execution environment         Created         Size
  0│ee-supported-rhel8       latest   True                          5 weeks ago     1.32 GB
  ```

- `python>=3.8` installed

- valid Red Hat Ansible Automation Platform subscription



### Managed hosts

- **Linux/Unix hosts**
  - python installed(check [Support Matrix](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix) for python version)
  - If SELinux, the `python3-libselinux` package need to be installed firstly
  - Regular user need `sudo` to get superuser access
- **Windows hosts**
  - Use `ansible.windows` Ansible Content Collection
  - managed hosts require PowerShell 3.0 or later
  - managed hosts need to have Windows PowerShell remoting configured.
  - requires .NET Framework 4.0 or later to be installed
  - More information on the Ansible website at https://docs.ansible.com/ansible/latest/user_guide/windows.html, or in the Red Hat training course *Microsoft Windows Automation with Red Hat Ansible Automation Platform* (DO417).

- Network Devices
  - Ansible automation supported routers and switches as well, This includes support for Cisco IOS, IOS XR, and NX-OS; Juniper Junos; Arista EOS; and VyOS-based networking devices, among others.
  - most network devices cannot run Python, Ansible runs network modules on the control node, not on the managed hosts. Special connection methods are also used to communicate with network devices, typically using either CLI over SSH, XML over SSH, or API over HTTP(S).
  - more information see [*Ansible for Network Automation* ](https://docs.ansible.com/ansible/latest/network/index.html)on the Ansible community website, or the Red Hat training course *Network Automation with Red Hat Ansible Automation Platform* (DO457).

---

## Chapter 2. Implementing Ansible Playbook

### Building inventory

- An *inventory* defines a collection of hosts that Ansible manages. 
  - hosts can be assigned to groups
  - group can contain child groups
  - hosts can be members of multiple groups
  - inventory can set variable to select a range of hosts/groups
  - static host inventory(INI-style or YAML)
  - Dynamic host inventory(Ansible plug-in)



#### Static Inventory

- txt file that support a number of different formats, including INI-style(most common) or YAML
- hostname cannot have space, like `[New York]` would not work, correct should be `[Newyork]`

1. A list of hostnames or IP addresses of managed hosts, each on a single line:

    ```
    web1.example.com
    web2.example.com
    db1.example.com
    db2.example.com
    192.0.2.42
    ```

2. Organize managed hosts into *host groups*.
    In the following example, the host inventory defines two host groups: `webservers` and `db-servers`. 

  ```
  [webservers]
  web1.example.com
  web2.example.com
  192.0.2.42
  
  [db-servers]
  db1.example.com
  db2.example.com
  ```

  

3. Hosts can be in multiple groups.  (role of the host, its physical location, whether it is in production or not, and so on).
    ```
    [webservers]
    web1.example.com
    web2.example.com
    192.0.2.42
    
    [db-servers]
    db1.example.com
    db2.example.com
    
    [east-datacenter]
    web1.example.com
    db1.example.com
    
    [production]
    web1.example.com
    web2.example.com
    db1.example.com
    db2.example.com
    
    [development]
    192.0.2.42
    ```

4. Nested Groups
   - creating a host group name with the `:children` suffix. The following example creates a new group called `north-america`, which includes all hosts from the `usa` and `canada` groups.
   - A group can have both managed hosts and child groups as members. For example, in the previous inventory you could add a `[north-america]` section that has its own list of managed hosts. **That list of hosts would be merged with the additional hosts that the `north-america` group inherits from its child groups.**
   
    ```
    [usa]
    washington1.example.com
    washington2.example.com
   
    [canada]
    ontario01.example.com
    ontario02.example.com
   
    [north-america:children]
    canada
    usa
    
    [north-america]
    washington03.example.com
    ```



##### Simplifying Host Specifications with Ranges

Ranges have the following syntax:

```
[START:END]
```

Ranges match all values from *START* to *END*, inclusively. Consider the following examples:

- `192.168.[4:7].[0:255]` matches all IPv4 addresses in the 192.168.4.0/22 network (192.168.4.0 through 192.168.7.255).
- `server[01:20].example.com` matches all hosts named `server01.example.com` through `server20.example.com`.
- `[a:c].dns.example.com` matches hosts named `a.dns.example.com`, `b.dns.example.com`, and `c.dns.example.com`.
- `2001:db8::[a:f]` matches all IPv6 addresses from 2001:db8::a through 2001:db8::f.

**NTOE: If leading zeros are included in numeric ranges, they are used in the pattern. The second example above does not match `server1.example.com` but does match `server07.example.com`.**



##### Verifying the Inventory

 `ansible-navigator inventory ` command to query an inventory file. 

```
[student@workstation ~]$ ansible-navigator inventory --help
Usage: ansible-navigator inventory [options]

inventory: Explore an inventory

Options (global):
 -h     --help                                   Show this help message and exit
 --version                                       Show the application version and exit
 --rad  --ansible-runner-artifact-dir            The directory path to store artifacts generated by ansible-runner
 --rac  --ansible-runner-rotate-artifacts-count  Keep ansible-runner artifact directories, for last n runs, if set to 0
                                                 artifact directories won't be deleted
 --rt   --ansible-runner-timeout                 The timeout value after which ansible-runner will forcefully stop the
                                                 execution
 --cdcp --collection-doc-cache-path              The path to collection doc cache (default:
                                                 /home/student/.cache/ansible-navigator/collection_doc_cache.db)
 --ce   --container-engine                       Specify the container engine (auto=podman then docker)
                                                 (auto|podman|docker) (default: auto)
 --co   --container-options                      Extra parameters passed to the container engine command
 --dc   --display-color                          Enable the use of color for mode interactive and stdout (true|false)
                                                 (default: true)
 --ecmd --editor-command                         Specify the editor command (default: vi +{line_number} {filename})
 --econ --editor-console                         Specify if the editor is console based (true|false) (default: true)
 --ee   --execution-environment                  Enable or disable the use of an execution environment (true|false)
                                                 (default: true)
 --eei  --execution-environment-image            Specify the name of the execution environment image (default:
                                                 registry.redhat.io/ansible-automation-platform-22/ee-supported-
                                                 rhel8:latest)
 --eev  --execution-environment-volume-mounts    Specify volume to be bind mounted within an execution environment
                                                 (--eev /home/user/test:/home/user/test:Z)
 --la   --log-append                             Specify if log messages should be appended to an existing log file,
                                                 otherwise a new log file will be created per session (true|false)
                                                 (default: true)
 --lf   --log-file                               Specify the full path for the ansible-navigator log file (default:
                                                 /home/student/ansible-navigator.log)
 --ll   --log-level                              Specify the ansible-navigator log level
                                                 (debug|info|warning|error|critical) (default: warning)
 -m     --mode                                   Specify the user-interface mode (stdout|interactive) (default:
                                                 interactive)
 --osc4 --osc4                                   Enable or disable terminal color changing support with OSC 4
                                                 (true|false) (default: true)
 --penv --pass-environment-variable              Specify an existing environment variable to be passed through to and
                                                 set within the execution environment (--penv MY_VAR)
 --pa   --pull-arguments                         Specify any additional parameters that should be added to the pull
                                                 command when pulling an execution environment from a container
                                                 registry. e.g. --pa='--tls-verify=false'
 --pp   --pull-policy                            Specify the image pull policy always:Always pull the image,
                                                 missing:Pull if not locally available, never:Never pull the image,
                                                 tag:if the image tag is 'latest', always pull the image, otherwise
                                                 pull if not locally available (always|missing|never|tag) (default:
                                                 tag)
 --senv --set-environment-variable               Specify an environment variable and a value to be set within the
                                                 execution environment (--senv MY_VAR=42)
 --tz   --time-zone                              Specify the IANA time zone to use or 'local' to use the system time
                                                 zone (default: utc)

Options (inventory subcommand):
 --hi   --help-inventory                         Help options for ansible-inventory command in stdout mode (true|false)
 -i     --inventory                              Specify an inventory file path or comma separated host list
 --ic   --inventory-column                       Specify a host attribute to show in the inventory view

Note: With '--mode stdout', 'ansible-navigator inventory' additionally supports the same parameters as the 'ansible-
inventory' command. For more information about these, try 'ansible-navigator inventory --help-inventory --mode stdout'

```



Check local 'inventory' file:

`[user@controlnode ~]$ ansible-navigator inventory -i inventory`

```
# The examples assume that a file named `inventory` exists in the current directory and that the file uses ranges to simplify the `[usa]` and `[canada]` group definitions

[student@workstation playbook-inventory]$ cat inventory 
[Webserver]
servera.lab.example.com
serverb.lab.example.com
serverc.lab.example.com
serverd.lab.example.com

[Raleigh]
servera.lab.example.com
serverb.lab.example.com

[Mountainview]
serverc.lab.example.com

[London]
serverd.lab.example.com

[Development]
servera.lab.example.com

[Testing]
serverb.lab.example.com

[Production]
serverc.lab.example.com
serverd.lab.example.com

[US:children]
Raleigh
Mountainview
```



Verify if the host in the inventory:
`[user@controlnode ~]$ ansible-navigator inventory -i inventory -m stdout --host servera.lab.example.com`



Lists all hosts in the inventory:

`[user@controlnode ~]$ ansible-navigator inventory -i inventory -m stdout --list`

```
[student@workstation playbook-inventory]$ ansible-navigator inventory -i inventory -m stdout --list
{
    "Development": {
        "hosts": [
            "servera.lab.example.com"
        ]
    },
    "London": {
        "hosts": [
            "serverd.lab.example.com"
        ]
    },
    "Mountainview": {
        "hosts": [
            "serverc.lab.example.com"
        ]
    },
    "Production": {
        "hosts": [
            "serverc.lab.example.com",
            "serverd.lab.example.com"
        ]
    },
    "Raleigh": {
        "hosts": [
            "servera.lab.example.com",
            "serverb.lab.example.com"
        ]
    },
    "Testing": {
        "hosts": [
            "serverb.lab.example.com"
        ]
    },
    "US": {
        "children": [
            "Mountainview",
            "Raleigh"
        ]
    },
    "Webserver": {
        "hosts": [
            "servera.lab.example.com",
            "serverb.lab.example.com",
            "serverc.lab.example.com",
            "serverd.lab.example.com"
        ]
    },
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "Development",
            "London",
            "Production",
            "Testing",
            "US",
            "Webserver",
            "ungrouped"
        ]
    }
}
```

The following command lists all hosts in a group.

`[user@controlnode ~]$ ansible-navigator inventory -i inventory -m stdout --graph US`

```
[student@workstation playbook-inventory]$ ansible-navigator inventory -i inventory -m stdout --graph US
@US:
  |--@Mountainview:
  |  |--serverc.lab.example.com
  |--@Raleigh:
  |  |--servera.lab.example.com
  |  |--serverb.lab.example.com
  

[student@workstation playbook-inventory]$ ansible-navigator inventory -i inventory -m stdout --graph all
@all:
  |--@Development:
  |  |--servera.lab.example.com
  |--@London:
  |  |--serverd.lab.example.com
  |--@Production:
  |  |--serverc.lab.example.com
  |  |--serverd.lab.example.com
  |--@Testing:
  |  |--serverb.lab.example.com
  |--@US:
  |  |--@Mountainview:
  |  |  |--serverc.lab.example.com
  |  |--@Raleigh:
  |  |  |--servera.lab.example.com
  |  |  |--serverb.lab.example.com
  |--@Webserver:
  |  |--servera.lab.example.com
  |  |--serverb.lab.example.com
  |  |--serverc.lab.example.com
  |  |--serverd.lab.example.com
  |--@ungrouped:
```



Run the `ansible-navigator inventory` command to interactively browse inventory hosts and groups:

```
[user@controlnode ~]$ ansible-navigator inventory -i inventory
  Title             Description
0│Browse groups     Explore each inventory group and group members members
1│Browse hosts      Explore the inventory with a list of all hosts


Type `:0` to select "Browse Groups":

Type `:1` to select "Browse Hosts"
```

Press the **ESC** key to exit the Groups menu. 

Ensure that host groups do not use the same names as hosts in the inventory or you will get a warning when runs commands.



##### Overriding the Location of the Inventory

The `/etc/ansible/hosts` file is considered the system's default static inventory file. However, normal practice is not to use that file but to specify a different location for your inventory files as following:

`ansible-navigator --inventory <pathname>`

`ansible-navigator -i <pathname>`



You can also define a different default location for the inventory file in your Ansible configuration file.



#### Dynamic Inventories

By using Ansible plug-ins, Ansible inventory information can also be dynamically generated, using information provided by external databases. 

For example, a dynamic inventory program could contact your Red Hat Satellite server or Amazon EC2 account, and use information stored there to construct an Ansible inventory. It can populate the inventory with up-to-date information provided by the service as new hosts are added, and old hosts are removed.

REFERENCES [How to build your inventory: Ansible Documentation](http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)





### Managing Ansible Configuration Files

#### Ansible configuration file

You can create and edit two files in each of your **Ansible project directories** that configure the behavior of Ansible and the `ansible-navigator` command. 

- `ansible.cfg`, which configures the behavior of several Ansible tools.
- `ansible-navigator.yml`, which changes default options for the `ansible-navigator` command.



#### Ansible setting

use the following two sections:

- `[defaults]`, which sets defaults for Ansible operation
- `[privilege_escalation]`, which configures how Ansible performs privilege escalation on managed hosts

The following is a sample `ansible.cfg` file:

```
[defaults]
inventory = ./inventory 
remote_user = user 
ask_pass = false 

[privilege_escalation]
become = true 
become_method = sudo 
become_user = root 
become_ask_pass = false 
```

| inventroy           | The `inventory` parameter specifies the path to a static inventory file, or to a directory containing multiple static inventory files and dynamic inventory scripts. |
| ------------------- | :----------------------------------------------------------- |
| **remote_user**     | The `remote_user` parameter specifies the username that Ansible uses to connect to the managed hosts. <u>In a container-based automation execution environment run by `ansible-navigator`, this defaults to `root`.</u> |
| **ask_pass**        | The `ask_pass` parameter specifies whether to prompt for an SSH password. **Defaults to `false`**. Set this parameter to `true` for password-based SSH authentication, or to `false` for SSH public key authentication. <u>If you use the `ansible-navigator` command, then setting this parameter to `true` requires disabling **playbook artifacts** and using the standard output mode.</u><br />`ssh-keygen` to create a public key and then `ssh-copy-id user@host.example.com` to copy the public key to the managed host `host.example.com` |
| **become**          | The `become` parameter specifies whether to automatically switch users on the managed host (typically to `root`) after connecting. Defaults to `false`. Although you can enable privilege escalation for all plays by setting this parameter to `true`, you might decide to only enable privilege escalation on the plays, blocks, or tasks that require elevated privileges. |
| **become_method**   | The `become_method` parameter specifies how to switch users. Defaults to `sudo`, although other methods, such as `su`, are available.  <br />config file: `/etc/sudoers` or create file under folder`/etc/sudoers.d`, command: `visudo` <br /><br />## Allows people in group wheel to run all commands %wheel  ALL=(ALL)       ALL <br /> ## Same thing without a password <br /># %wheel        ALL=(ALL)       NOPASSWD: ALL |
| **become_user**     | The `become_user` parameter specifies which user to switch to on the managed host. Defaults to `root`. |
| **become_ask_pass** | The `become_ask_pass` parameter specifies whether to prompt for a password for the `become_method` parameter. Defaults to `false`. If you use the `ansible-navigator` command, then setting this parameter to `true` requires disabling playbook artifacts and using the standard output mode. |

**NOTE:**

- The `ansible-playbook` command can also use a `.ansible.cfg` file in your home directory or the `/etc/ansible/ansible.cfg` file, but these local files on your system are not used by the `ansible-navigator` command.

- If the managed hosts would not have SSH key-based authentication configured yet, you would have to run like following:

  `ansible-navigator run --ask-pass -pae false -m stdout <ping-internetweb.yml>`   the command prompts to authenticate as the remote user.



#### Determining Current config

`ansible-navigator config` command run from a `/home/student/project/` directory that contains an `ansible.cfg` file:

```
   Name                        Default Source                            Current
...output omitted...
44│Default ask pass            True    default                           False
45│Default ask vault pass      True    default                           False
46│Default become              False   /home/student/project/ansible.cfg True
47│Default become ask pass     False   /home/student/project/ansible.cfg True
 ...output omitted...
50│Default become method       False   /home/student/project/ansible.cfg sudo
51│Default become user         False   /home/student/project/ansible.cfg root
 ...output omitted...
```

In the preceding example, each line describes an Ansible configuration parameter.

- The `True` value in the `Default` column for the `Default ask pass` and `Default ask vault pass` parameters means that the parameters are using their default values. The current value for both parameters is `False` and the values are displayed in the `Current` column.
- The `Default become` and `Default become ask pass` parameters have been manually configured to `True` in the `/home/student/project/ansible.cfg` configuration file. The `Default` column is `False` for these two parameters. The `Source` column provides the path to the configuration file which defines these parameters, and the `Current` column shows that the value for these two parameters is `True`.
- The `Default become method` parameter has the current value of `sudo`, and the `Default become user` parameter has the current value of `root`.



#### Ansible-navigator setting

Creating a configuration file (or settings file) for `ansible-navigator` to override the default values of its configuration settings. The settings file can be in JSON (`.json`) or YAML (`.yml` or `.yaml`) format.

Automation content navigator looks for a settings file in the following order and uses the first file that it finds:

- If the `ANSIBLE_NAVIGATOR_CONFIG` environment variable is set, then use the configuration file at the location it specifies.
- An `ansible-navigator.yml` file in your current Ansible project directory.
- A `.ansible-navigator.yml` file in your home directory (notice that the file name contains a "dot" at the start of the name).

Just like the Ansible configuration file, each project can have its own automation content navigator settings file.

The following `ansible-navigator.yml` file configures some common settings:

```yaml
---
ansible-navigator:
  execution-environment: 
    image: utility.lab.example.com/ee-supported-rhel8:latest 
    pull:
      policy: missing 
  playbook-artifact: 
    enable: false 
  mode: stdout 
```

| execution-environment | The `execution-environment` section configures settings for the automation execution environment that the `ansible-navigator` command uses. |
| --------------------- | ------------------------------------------------------------ |
| **image**             | The `image` key defines the container image name to use for the automation execution environment. |
| **policy**            | The `policy` key nested below the `pull` section states to only pull the container image if it does not already exist on the local machine. |
| **playbook-artifact** | The `playbook-artifact` section configures settings for the JSON files that Ansible generates every time you run a playbook. **Each generated JSON file records information about a specific playbook run.** You can use these files to review the results of a playbook run or to troubleshoot playbook issues. |
| **enable**            | The `enable` key nested below the `playbook-artifact` section disables generating playbook artifacts when using the `ansible-navigator run` command. <u>Playbook artifacts must be disabled when you require a prompt for a password when running a playbook. You can temporarily override this setting from the command line with the `--pae` option.</u> |
| **mode**              | The `mode` key defines the output mode for the `ansible-navigator` command. The value for this key should be set to either `interactive` (the default) or `stdout`. You can temporarily override this setting from the command line with the `-m` option. |

**Additional Ref:**
*Developing Advanced Automation with Red Hat Ansible Automation Platform (DO374)*. Refer to https://ansible.readthedocs.io/projects/navigator/settings/ for more documentation on the settings that you can use in this file.



#### Configuration File Comments

Both the `ansible.cfg` file and `ansible-navigator.yml` support the number **sign** (#) at the start of a line as a comment character.

In addition, the `ansible.cfg` file supports the **semicolon** (;) as a comment character. The semicolon character comments out everything to the right of it on the line.





### Writing and Running Ansible Playbooks

#### Format of Ansible Playbooks

- The power of Ansible is that you can use playbooks to run multiple, complex tasks against a set of targeted hosts in an easily repeatable manner.

- A *task* is the application of a module to perform a specific unit of work. A *play* is a sequence of tasks to be applied, in order, to one or more hosts selected from your inventory. A *playbook* is a text file containing a list of one or more plays to run in a specific order.

- The following example contains one play with a single task.

  ```
  ---
  - name: Configure important user consistently
    hosts: servera.lab.example.com
    tasks:
      - name: Newbie exists with UID 4000
        ansible.builtin.user:
          name: newbie
          uid: 4000
          state: present
  ```

  - A playbook is a text file written in YAML format(`.yml file`)

  - Uses indentation with space characters to indicate the structure of its data (Do not use `TAB` characters). YAML does not place strict requirements on how many spaces are used for the indentation, but **two** basic rules apply:

    - Data elements at the same level in the hierarchy (such as items in the same list) must have the **same** indentation.

    - Items that are children of another item must be indented more than their parents.

  - You can also add blank lines for readability.

  - For `vi` text editor to easily edit your playbooks. add the following line to your `$HOME/.vimrc` file, and when `vi` detects that you are editing a YAML file, it performs a 2-space indentation when you press the **Tab** key and automatically indents subsequent lines.

    ```
    autocmd FileType yaml setlocal ai ts=2 sw=2 et
    ```



- Rules of playbooks

  - Usually start with three dashes(---), might end with three dots(...) but often being omitted.

  - The play itself is a collection of key-value pairs. Keys in the same play should have the same indentation. The following example shows a YAML snippet with three keys. The first two keys have simple values. The third has a list of three items as a value. **The Ansible always run plays in order**.

    ```yaml
      name: just an example
      hosts: webservers
      tasks:
    	- name: Web server is enabled
          ansible.builtin.service:
            name: httpd
            enabled: true
    
        - name: NTP server is enabled
          ansible.builtin.service:
            name: chronyd
            enabled: true
    
        - name: Postfix is enabled
          ansible.builtin.service:
            name: postfix
            enabled: true
    ```

    

- Finding Modules for tasks

  - The `ansible-core` package provides a single Ansible Content Collection named `ansible.builtin`. These modules are always available to you. Visit https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ for a list of modules contained in the `ansible.builtin` collection.
  -  `ansible-navigator collections` command show Red Hat Ansible Automation Platform 2.2, `ee-supported-rhel8`, includes a number of other Ansible Content Collections.
  - Other ansible contents:(installed in the `collections` directory of your Ansible project, no formal support by RH)
    - The automation hub offered through the Red Hat Hybrid Cloud Console at `https://console.redhat.com/ansible/automation-hub`
    - A private automation hub managed by your organization
    - The community's Ansible Galaxy website at [https://galaxy.ansible.com](https://galaxy.ansible.com/)

  

#### Running Playbooks

-  `ansible-navigator run` 

- The following example shows the contents of a simple playbook, and then the result of running it.

  ```yaml
  [user@controlnode playdemo]$ cat webserver.yml
  ---
  - name: Play to set up web server
    hosts: servera.lab.example.com
    tasks:
    - name: Latest httpd version installed
      ansible.builtin.dnf:
        name: httpd
        state: latest
  ...output omitted...
  
  [user@controlnode playdemo]$ ansible-navigator run \
  > -m stdout webserver.yml
  
  PLAY [Play to set up web server] ************************************************
  
  TASK [Gathering Facts] *********************************************************
  ok: [servera.lab.example.com]
  
  TASK [Latest httpd version installed] ******************************************
  changed: [servera.lab.example.com]
  
  PLAY RECAP *********************************************************************
  servera.lab.example.com    : ok=2    changed=1    unreachable=0    failed=0   skipped=0    rescued=0    ignored=0
  ```

  The value of the `name` key for each play and task is displayed when the playbook is run. (The `Gathering Facts` task is a special task that the `ansible.builtin.setup` module usually runs automatically at the start of a play.

  You should also see that the `Latest httpd version installed` task is `changed` for `servera.lab.example.com`. This means that the task changed something on that host to ensure that its specification was met. In this case, it means that the `httpd` package was not previously installed or was not the latest version.

  In general, tasks in Ansible Playbooks are idempotent, and it is safe to run a playbook multiple times. If the targeted managed hosts are already in the correct state, no changes should be made. For example, assume that the playbook from the previous example is run again:

  ```
  [user@controlnode playdemo]$ ansible-navigator run \
  > -m stdout webserver.yml
  
  PLAY [Play to set up web server] ************************************************
  
  TASK [Gathering Facts] *********************************************************
  ok: [servera.lab.example.com]
  
  TASK [Latest httpd version installed] ******************************************
  ok: [servera.lab.example.com]
  
  PLAY RECAP *********************************************************************
  servera.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0   skipped=0    rescued=0    ignored=0
  ```

  This time, all tasks passed with status `ok` and no changes were reported.



##### Increasing Output Verbosity

The default output does not provide detailed task execution information. The `-v` option provides additional information, with up to four levels.



**Table 2.2. Configuring the Output Verbosity of Playbook Execution**

| Option  | Description                                                  |
| :------ | :----------------------------------------------------------- |
| `-v`    | Displays task results.                                       |
| `-vv`   | Displays task results and task configuration.                |
| `-vvv`  | Displays extra information about connections to managed hosts. |
| `-vvvv` | Adds extra verbosity options to the connection plug-ins, including users being used on the managed hosts to execute scripts, and what scripts have been executed. |



##### Syntax Verification

 `ansible-navigator run --syntax-check` command to validate the syntax of a playbook. The following example shows the successful syntax validation of a playbook.

```
[user@controlnode playdemo]$ ansible-navigator run \
> -m stdout webserver.yml --syntax-check
playbook: /home/user/playdemo/webserver.yml
```

When syntax validation fails, a syntax error is reported. The output also includes the approximate location of the syntax issue in the playbook. The following example shows the failed syntax validation of a playbook where the space separator is missing after the `name` attribute for the play.

```
[user@controlnode playdemo]$ ansible-navigator run \
> -m stdout webserver.yml --syntax-check

ERROR! Syntax Error while loading YAML.
  mapping values are not allowed in this context

The error appears to have been in ...output omitted... line 3, column 8, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

- name:Play to set up web server
  hosts: servera.lab.example.com
       ^ here
```



##### Executing a Dry Run

`ansible-navigator run --check`  option to run a playbook in *check mode*, which performs a "dry run" of the playbook.  it does not make any actual changes to managed hosts. (Sometimes the Dry run report a failure but actually Run would finished OK, eg. the Dru Run can not start a service so the later service check step failed but the Real Run would not have the problem.)

In this case, the dry run reports that the task would make a change on the managed host for the latest httpd version to be installed.

```
[user@controlnode playdemo]$ ansible-navigator run \
> -m stdout webserver.yml --check

PLAY [Play to set up web server] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Latest httpd version installed] ******************************************
changed: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

##### REFERENCES

[Intro to playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

[Working with playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)

[Validating tasks: check mode and diff mode — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html)



#### Implementing Multiple Plays

Writing a playbook that contains multiple plays is very straightforward. Each play in the playbook is written as a top-level list item in the playbook. Each play is a list item containing the usual play keywords.

##### Remote Users in Plays

- Ansible determines which user account to use when connecting to a managed host based on the following list, selecting the first username it finds in this order:

  - The `ansible_user` variable set for the host or group, if set.

  - The `remote_user` from the current play, if set.

    ```
    remote_user: remoteuser
    ```

  - The `remote_user` from the `ansible.cfg` configuration file, if set.

If no value has been set for any of the preceding settings, and you are running playbooks by using `ansible-navigator` with an execution environment, Ansible uses `root`. (If you are using `ansible-playbook`, Ansible uses the name of the user that ran the command.)

##### Privilege Escalation in Plays

- Use the `become` Boolean keyword to enable(yes/true) or disable(no/false) privilege escalation for an individual play or task. 

- use the `become_method` keyword in the play to specify the privilege escalation method to use for that play

- use the `become_user` keyword in the play to define the user account to use for privilege escalation in that specific play.

- The following example demonstrates some of these keywords in a play:

  ```
  - name: /etc/hosts is up-to-date
    hosts: datacenter-west
    remote_user: automation
    become: true
    become_method: sudo
    become_user: root
  
    tasks:
      - name: server.example.com in /etc/hosts
        ansible.builtin.lineinfile:
          path: /etc/hosts
          line: '192.0.2.42 server.example.com server'
          state: present
  ```

##### **Selecting Modules**

**Table 2.3. Ansible Modules**

| Category  | Modules                                                      |
| :-------- | :----------------------------------------------------------- |
| Files     | `ansible.builtin.copy`: Copy a local file to the managed host.`ansible.builtin.file`: Set permissions and other properties of files.`ansible.builtin.lineinfile`: Ensure a particular line is or is not in a file.`ansible.posix.synchronize`: Synchronize content using `rsync`. |
| Software  | `ansible.builtin.package`: Manage packages using the automatically detected package manager native to the operating system.`ansible.builtin.dnf`: Manage packages using the DNF package manager.`ansible.builtin.apt`: Manage packages using the APT package manager.`ansible.builtin.pip`: Manage Python packages from PyPI. |
| System    | `ansible.posix.firewalld`: Manage arbitrary ports and services using `firewalld`.`ansible.builtin.reboot`: Reboot a machine.`ansible.builtin.service`: Manage services.`ansible.builtin.user`: Add, remove, and manage user accounts. |
| Net Tools | `ansible.builtin.get_url`: Download files over HTTP, HTTPS, or FTP.`ansible.builtin.uri`: Interact with web services. |

##### Module Documentation

- run the `ansible-navigator doc -l` command to displays a list of module **short names** and a synopsis of their functions.
- run`ansible-navigator doc *`module_name`*` command to display detailed documentation for a module. (interactive mode and can use `-m stdout` to show pages)
-  `ansible-navigator collections` to use interactive mode to browse module
-  `ansible-navigator -s doc *`module_name`*`  to show summary of a module

```
[user@controlnode ~]$ ansible-navigator doc -l
add_host                                                Add a host (and alt...
amazon.aws.aws_az_facts                                 Gather information ...
amazon.aws.aws_az_info                                  Gather information ...
amazon.aws.aws_caller_info                              Get information abo...
amazon.aws.aws_s3                                       manage objects in S...
...output omitted...
vyos.vyos.vyos_user                                     Manage the collecti...
vyos.vyos.vyos_vlan                                     Manage VLANs on VyO...
wait_for                                                Waits for a conditi...
wait_for_connection                                     Waits until remote ...
yum                                                     Manages packages wi...
yum_repository                                          Add or remove YUM r...
```



#### Example 

```
---
- name: Enable intranet services
  hosts: servera.lab.example.com
  become: true
  tasks:
    - name: Latest version of httpd and firewalld installed
      ansible.builtin.dnf:
        name:
          - httpd
          - firewalld
        state: latest

    - name: Test html page is installed
      ansible.builtin.copy:
        content: "Welcome to the example.com intranet!\n"
        dest: /var/www/html/index.html

    - name: Firewall enabled and running
      ansible.builtin.service:
        name: firewalld
        enabled: true
        state: started

    - name: Firewall permits access to httpd service
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: Web server enabled and running
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

- name: Test intranet web server
  hosts: workstation.lab.example.com
  become: false
  tasks:
    - name: Connect to intranet web server
      ansible.builtin.uri:
        url: http://servera.lab.example.com
        return_content: true
        status_code: 200
```

1. The directory has already been populated with an `ansible.cfg` configuration file and an inventory file named `inventory`. The managed host, `servera.lab.example.com`, is already defined in this inventory file.

2. Create a playbook named `/home/student/playbook-multi/intranet.yml` and add the lines needed to start the first play. It should target the managed host `servera.lab.example.com` and enable privilege escalation.

3. As the first task in the first play, define a task that ensures that the `httpd` and `firewalld` packages are up-to-date.

4. Add a task to the first play's list that ensures that the correct content is in the `/var/www/html/index.html` file.

5. Define two more tasks in the play to ensure that the `firewalld` service is running and starts on boot, and allows connections to the `httpd` service.

6. Add a final task to the first play that ensures that the `httpd` service is running and starts at boot.

7. In the `/home/student/playbook-multi/intranet.yml` file, define a second play that targets `workstation.lab.example.com` and tests the intranet web server. By testing from the `workstation.lab.example.com` machine, you can verify that the `servera.lab.example.com` machine allows external web requests through the firewall. This play does not require privilege escalation.

8. Add a single task to the second play, and use the `ansible.builtin.uri` module to request content from `http://servera.lab.example.com`. The task should verify a return HTTP status code of `200`. Configure the task to place the returned content in the task results variable.

   

   Run the playbook using the `ansible-navigator run` command.

   ```
   [student@workstation playbook-multi]$ ansible-navigator run \
   > -m stdout intranet.yml
   
   PLAY [Enable intranet services] ************************************************
   
   TASK [Gathering Facts] *********************************************************
   ok: [servera.lab.example.com]
   
   TASK [Latest version of httpd and firewalld installed] *************************
   changed: [servera.lab.example.com]
   
   TASK [Test html page is installed] *********************************************
   changed: [servera.lab.example.com]
   
   TASK [Firewall enabled and running] ********************************************
   changed: [servera.lab.example.com]
   
   TASK [Firewall permits access to httpd service] ********************************
   changed: [servera.lab.example.com]
   
   TASK [Web server enabled and running] ******************************************
   changed: [servera.lab.example.com]
   
   PLAY [Test intranet web server] ************************************************
   
   TASK [Gathering Facts] *********************************************************
   ok: [workstation.lab.example.com]
   
   TASK [Connect to intranet web server] ******************************************
   ok: [workstation.lab.example.com]
   
   PLAY RECAP *********************************************************************
   servera.lab.example.com    : ok=6    changed=5    unreachable=0    failed=0  ...
   workstation.lab.example.com : ok=2    changed=0    unreachable=0    failed=0 ...
   ```

   Use the `curl` command to verify that an HTTP GET request to `http://servera.lab.example.com` provides the correct content.

   ```
   [student@workstation playbook-multi]$ curl http://servera.lab.example.com
   Welcome to the example.com intranet!
   ```



#### Running Arbitrary Commands on Managed Hosts

The `ansible.builtin.command` module is the simplest of these commands. Its `cmd` argument specifies the command that you want to run.

The following example task runs `/opt/bin/makedb.sh` on managed hosts.

```
- name: Run the /opt/bin/makedb.sh command
  ansible.builtin.command:
    cmd: /opt/bin/makedb.sh
```

Unlike most modules, `ansible.builtin.command` is not idempotent. Every time the task is specified in a play, it runs and it reports that it changed something on the managed host, even if nothing needed to be changed.

You can try to make the task safer by configuring it only to run based on the existence of a file. The `creates` option causes the task to run only if a file is missing; the assumption is that if the task runs, it creates that file. The `removes` option causes the task to run only if a file is present; the assumption is that if the task runs, it removes that file.

For example, the following task only runs if `/opt/db/database.db` is not present:

```
- name: Initialize the database
  ansible.builtin.command:
    cmd: /opt/bin/makedb.sh
    creates: /opt/db/database.db
```

The `ansible.builtin.command` module cannot access shell environment variables or perform shell operations such as input/output redirection or pipelines. When you need to perform shell processing, you can use the `ansible.builtin.shell` module. Like the `ansible.builtin.command` module, you pass the commands to be executed as arguments to the module.

Both `ansible.builtin.command` and `ansible.builtin.shell` modules require a working Python installation on the managed host. A third module, `ansible.builtin.raw`, can run commands directly using the remote shell, bypassing the module subsystem. This is useful when you are managing systems that cannot have Python installed (for example, a network router). It can also be used to install Python on a managed host.



**IMPORTANT NOTE:**

When possible, try to avoid the `ansible.builtin.command`, `ansible.builtin.shell`, and `ansible.builtin.raw` modules in playbooks, even though they might seem simple to use. Because these run arbitrary commands on the managed hosts, it is very easy to write non-idempotent playbooks with these modules.

If you must use them, it is probably best to use the `ansible.builtin.command` module first, resorting to `ansible.builtin.shell` or `ansible.builtin.raw` only if you need their special features.

As another example, the following task using the `ansible.builtin.shell` module is not idempotent. Every time the play is run, it rewrites `/etc/resolv.conf` even if it already consists of the line `nameserver 192.0.2.1`. A task that is not idempotent displays the `changed` status every time the task is run.

```
- name: Non-idempotent approach with shell module
  ansible.builtin.shell:
    cmd: echo "nameserver 192.0.2.1" > /etc/resolv.conf
```

You can create idempotent tasks in several ways using the `ansible.builtin.shell` module, and sometimes making those changes and using `ansible.builtin.shell` is the best approach. But in this case, a better solution would be to use `ansible-navigator doc` to discover the `ansible.builtin.copy` module and use that to get the desired effect.

The following example does not rewrite the `/etc/resolv.conf` file if it already consists of the correct content:

```
- name: Idempotent approach with copy module
  ansible.builtin.copy:
    dest: /etc/resolv.conf
    content: "nameserver 192.0.2.1\n"
```

The `ansible.builtin.copy` module tests to see if the state has already been met, and if so, it makes no changes. The `ansible.builtin.shell` module allows a lot of flexibility, but also requires more attention to ensure that it runs with idempotency.

You can run idempotent playbooks repeatedly to ensure systems are in a particular state without disrupting those systems if they already are.

### YAML Syntax

The last part of this section investigates some variations of YAML or Ansible Playbook syntax that you might encounter.

#### YAML Comments

Comments can also be used to aid readability. In YAML, everything to the right of the number sign (#) is a comment. If there is content to the left of the comment, precede the hash with a space.

```
# This is a YAML comment
some data # This is also a YAML comment
```

#### YAML Strings

Strings in YAML do not normally need to be put in quotation marks even if the string contains no spaces. You can enclose strings in either double or single quotation marks.

```
this is a string
'this is another string'
"this is yet another a string"
```

You can write multiline strings in either of two ways. You can use the **vertical bar (|)** character to denote that newline characters within the string are to be preserved. (keep the original format of sentence, not single sentence once folded.)

```
include_newlines: |
        Example Company
        123 Main Street
        Atlanta, GA 30303
```

You can also write multiline strings using the **greater-than (>) character** to indicate that newline characters are to be converted to spaces and that leading white spaces in the lines are to be removed. This method is often used to break long strings at space characters so that they can span multiple lines for better readability. (break down a single long sentence into pieces)

```
fold_newlines: >
        This is an example
        of a long string,
        that will become
        a single sentence once folded.
```

#### YAML Dictionaries

You have seen collections of key-value pairs written as an indented block, as follows:

```
  name: svcrole
  svcservice: httpd
  svcport: 80
```

Dictionaries can also be written in an inline block format enclosed in braces, as follows:

```
  {name: svcrole, svcservice: httpd, svcport: 80}
```

Avoid the inline block format because it is harder to read. However, there is at least one situation in which it is more commonly used. The use of *roles* is discussed later in this course. When a playbook includes a list of roles, it is more common to use this syntax to make it easier to distinguish roles included in a play from the variables being passed to a role.

#### YAML Lists

You have also seen lists written with the normal single-dash syntax:

```
  hosts:
    - servera
    - serverb
    - serverc
```

Lists also have an inline format enclosed in square braces, as follows:

```
hosts: [servera, serverb, serverc]
```

You should avoid this syntax because it is usually harder to read.

#### REFERENCES

[Intro to playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

[Working with playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)

[Module Maintenance & Support — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/modules_support.html)

[Adding modules and plugins locally — Ansible Documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.html)

[YAML Syntax — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

[Ansible.Builtin — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)



### Chapter 2 TEST

The `/home/student/playbook-review` working directory has been created on the `workstation` machine for the Ansible project. The directory has already been populated with an `ansible.cfg` configuration file and an `inventory` file. The managed host, `serverb.lab.example.com`, is already defined in this inventory file.



**Instructions**

1. Change into the `/home/student/playbook-review` directory and create a new playbook called `internet.yml`. Add the necessary entries to start a first play named `Enable internet services` and specify its intended managed host, `serverb.lab.example.com`. Add the necessary entry to enable privilege escalation, and one to start a task list.

   

2. Add the necessary entries to the `/home/student/playbook-review/internet.yml` file to define a task that installs the latest versions of the `firewalld`, `httpd`, `mariadb-server`, `php`, and `php-mysqlnd` packages. Indent the beginning of the entry four spaces.

   

3. Add the necessary entries to the `/home/student/playbook-review/internet.yml` file to define the firewall configuration tasks. They should ensure that the `firewalld` service is enabled and running, and that access is allowed to the `http` service. Indent the beginning of these entries four spaces.

   

4. Add the necessary entries to ensure the `httpd` and `mariadb` services are enabled and running. Indent the beginning of these entries four spaces.

   

5. Add the necessary entry that uses the `ansible.builtin.copy` module to copy the `/home/student/playbook-review/index.php` file to the `/var/www/html/` directory on the managed host. Ensure the file mode is set to 0644. Indent the beginning of these entries four spaces.

   

6. Define another play in the `/home/student/playbook-review/internet.yml` file for a task to be performed on the control node. This play tests access to the web server that should be running on the `serverb.lab.example.com` managed host. This play does not require privilege escalation, and runs on the `workstation.lab.example.com` managed host.

   

7. Add the necessary entry that tests the web service running on `serverb` from the control node using the `ansible.builtin.uri` module. Look for a return status code of `200`. Indent the beginning of the entry four spaces.

   

   internet.yml:

   ```yaml
   ---
   - name: Enable internet services
     hosts: serverb.lab.example.com
     become: true
   
     tasks:
       - name: Install latest packages for APPs
         ansible.builtin.dnf:
           name:
             - firewalld
             - httpd
             - mariadb-server
             - php
             - php-mysqlnd
           state: latest
   
       - name: Firewall configuration tasks(running)
         ansible.builtin.service:
           name: firewalld
           state: started
           enabled: true
   
       - name: Enable access to httpd
         ansible.posix.firewalld:
           service: http
           permanent: true
           state: enabled
           immediate: true
   
       - name: Ensure httpd running
         ansible.builtin.service:
           name: httpd
           state: started
           enabled: true
   
       - name: Ensure mariadb running
         ansible.builtin.service:
           name: mariadb
           state: started
           enabled: true
   
   
       - name: Copy index.html to /var/www/html folder
         ansible.builtin.copy:
           src: /home/student/playbook-review/index.php
           dest: /var/www/html
           mode: 0644
   
   - name: Access the internet from workstation to serverb
     hosts: workstation
     become: false
     
     tasks:
       - name: Connect to serverb web
         ansible.builtin.uri:
           url: http://serverb.lab.example.com
           return_content: true
           status_code: 200
   
   
   ```

   

8. Validate the syntax of the `internet.yml` playbook.

   ```
   [student@workstation playbook-review]$ ansible-navigator run \
   > -m stdout internet.yml --syntax-check
   playbook: /home/student/playbook-review/internet.yml
   ```

   

9. Use the `ansible-navigator run` command to run the playbook. Read through the generated output to ensure that all tasks completed successfully.

   ```
   [student@workstation playbook-review]$ ansible-navigator run -m stdout internet.yml
   
   PLAY [Enable internet services] ************************************************
   
   TASK [Gathering Facts] *********************************************************
   ok: [serverb.lab.example.com]
   
   TASK [Install latest packages for APPs] ****************************************
   ok: [serverb.lab.example.com]
   
   TASK [Firewall configuration tasks(running)] ***********************************
   ok: [serverb.lab.example.com]
   
   TASK [Enable access to httpd] **************************************************
   ok: [serverb.lab.example.com]
   
   TASK [Ensure httpd running] ****************************************************
   changed: [serverb.lab.example.com]
   
   TASK [Ensure mariadb running] **************************************************
   changed: [serverb.lab.example.com]
   
   TASK [Copy index.html to /var/www/html folder] *********************************
   changed: [serverb.lab.example.com]
   
   PLAY [Access the internet from workstation to serverb] *************************
   
   TASK [Gathering Facts] *********************************************************
   ok: [workstation]
   
   TASK [Connect to serverb web] **************************************************
   ok: [workstation]
   
   PLAY RECAP *********************************************************************
   serverb.lab.example.com    : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   workstation                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   ```

   



---

## Chapter 3. Managing Variables and Facts

### Managing Variables

Variables provide a convenient way to manage dynamic values for a given environment in your Ansible project. Examples of values that variables might contain include:

- Users to create
- Packages to install
- Services to restart
- Files to remove
- Archives to retrieve from the internet

#### Naming Variables

Variable names must start with a letter, and they can only contain letters, numbers, and underscores.

The following table illustrates the difference between invalid and valid variable names.

**Table 3.1. Examples of Invalid and Valid Ansible Variable Names**

| Invalid variable names | Valid variable names                  |
| :--------------------- | :------------------------------------ |
| `web server`           | `web_server`                          |
| `remote.file`          | `remote_file`                         |
| `1st file`             | `file_1``file1`                       |
| `remoteserver$1`       | `remote_server_1` or `remote_server1` |

#### Defining Variables

The following simplified list shows ways to define a variable, ordered from the **lowest** precedence to the **highest**:

- Group variables defined in the inventory (**lowest**)

- Group variables defined in files in a `group_vars` subdirectory in the same directory as the inventory or the playbook

- Host variables defined in the inventory

- Host variables defined in files in a `host_vars` subdirectory in the same directory as the inventory or the playbook

- Host facts, discovered at runtime

- Play variables in the playbook (`vars` and `vars_files`)

  - One common method is to place a variable in a `vars` block at the beginning of a play:

    ```yaml
    - hosts: all
      vars:
        user: joe
        home: /home/joe
    or
    
    - hosts: all
      vars_files:
        - vars/users.yml
    ```

  - Using Variables in Playbooks

    Variables are referenced by placing the variable name in double braces (`{{ }}`). Ansible substitutes the variable with its value when the task is executed.(quotes are mandatory if the variable is the  first element to start)

    ```
    vars:
      user: joe
    
    tasks:
      # This line will read: Creates the user joe
      - name: Creates the user {{ user }}
        user:
          # This line will create the user named Joe
          name: "{{ user }}"
    ```

- Task variables

- Extra variables defined on the command line (**highest**)

  - by using the `--extra-vars` or `-e` option and specifying those variables

  - ```
    [user@demo ~]$ ansible-navigator run main.yml -e "package=apache"
    ```



For the Directories to Populate Host and Group Variables

The following examples illustrate this approach in more detail. Consider a scenario where you need to manage two data centers, and the data center hosts are defined in the `~/project/inventory` inventory file:

```
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2
```

- If you need to define a general value for all servers in both data centers, set a group variable for the `datacenters` host group:

  ```
  [admin@station project]$ cat ~/project/group_vars/datacenters
  package: httpd
  ```

- If you need to define difference values for each data center, set a group variable for each data center host group:

  ```
  [admin@station project]$ cat ~/project/group_vars/datacenter1
  package: httpd
  [admin@station project]$ cat ~/project/group_vars/datacenter2
  package: apache
  ```

- If you need to define different values for each managed host in every data center, then define the variables in separate host variable files:

  ```
  [admin@station project]$ cat ~/project/host_vars/demo1.example.com
  package: httpd
  [admin@station project]$ cat ~/project/host_vars/demo2.example.com
  package: apache
  [admin@station project]$ cat ~/project/host_vars/demo3.example.com
  package: mariadb-server
  [admin@station project]$ cat ~/project/host_vars/demo4.example.com
  package: mysql-server
  ```

The directory structure for the example project, `project`, if it contained all the example files above, would appear as follows:

```
project
├── ansible.cfg
├── group_vars
│   ├── datacenters
│   ├── datacenters1
│   └── datacenters2
├── host_vars
│   ├── demo1.example.com
│   ├── demo2.example.com
│   ├── demo3.example.com
│   └── demo4.example.com
├── inventory
└── playbook.yml
```



#### Using Dictionaries as Variables

For example, consider the following snippet:

```
user1_first_name: Bob
user1_last_name: Jones
user1_home_dir: /users/bjones
user2_first_name: Anne
user2_last_name: Cook
user2_home_dir: /users/acook
```

This could be rewritten as a dictionary called `users`:

```
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```

You can then use the following variables to access user data:

```
# Returns 'Bob'
users.bjones.first_name

# Returns '/users/acook'
users.acook.home_dir
```

Because the variable is defined as a Python dictionary, an alternative syntax is available.

```
# Returns 'Bob'
users['bjones']['first_name']

# Returns '/users/acook'
users['acook']['home_dir']
```

#### Capturing Command Output with Registered Variables

You can use the `register` statement to capture the output of a command or other information about the execution of a module. 

The following play demonstrates how to capture the output of a command for debugging purposes:

```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      ansible.builtin.dnf:
        name: httpd
        state: installed
      register: install_result

    - debug:
        var: install_result
```

When you run the play, the `debug` module dumps the value of the `install_result` registered variable to the terminal.

```
[user@demo ~]$ ansible-navigator run playbook.yml -m stdout
PLAY [Installs a package and prints the result] ****************************

TASK [setup] ***************************************************************
ok: [demo.example.com]

TASK [Install the package] *************************************************
ok: [demo.example.com]

TASK [debug] ***************************************************************
ok: [demo.example.com] => {
    "install_result": {
        "changed": false,
        "msg": "",
        "rc": 0,
        "results": [
            "httpd-2.4.51-7.el9_0.x86_64 providing httpd is already installed"
        ]
    }
}

PLAY RECAP *****************************************************************
demo.example.com    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### REFERENCES

[How to build your inventory — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

[Using Variables — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)

[Variable precedence: Where should I put a variable?](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)

[YAML Syntax — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)



#### Data-variable example

```
---
- name: Deploy and start Apache HTTPD service
  hosts: webserver
  vars:
    web_pkg: httpd
    firewall_pkg: firewalld
    web_service: httpd
    firewall_service: firewalld
    rule: http

  tasks:
    - name: Required packages are installed and up to date
      ansible.builtin.dnf:
        name:
          - "{{ web_pkg  }}"
          - "{{ firewall_pkg }}"
        state: latest

    - name: The {{ firewall_service }} service is started and enabled
      ansible.builtin.service:
        name: "{{ firewall_service }}"
        enabled: true
        state: started

    - name: The {{ web_service }} service is started and enabled
      ansible.builtin.service:
        name: "{{ web_service }}"
        enabled: true
        state: started

    - name: Web content is in place
      ansible.builtin.copy:
        content: "Example web content"
        dest: /var/www/html/index.html

    - name: The firewall port for {{ rule }} is open
      ansible.posix.firewalld:
        service: "{{ rule }}"
        permanent: true
        immediate: true
        state: enabled

- name: Verify the Apache service
  hosts: workstation
  become: false
  tasks:
    - name: Ensure the webserver is reachable
      ansible.builtin.uri:
        url: http://servera.lab.example.com
        status_code: 200
```





### Managing Secret

- use the command-line tool named `ansible-vault` to create, edit, encrypt, decrypt, and view files.
- Ansible Vault does not implement its own cryptographic functions but rather uses an external Python toolkit. Files are protected with symmetric encryption using AES256 with a password as the secret key. Note that the way this is done has not been formally audited by a third party.

#### Creating an Encrypted File

Use `ansible-vault create *`filename`*` ---This command prompts for the new Vault password and then opens a file using the default editor, `vi`. You can export the `EDITOR` environment variable to specify a different default editor. To set the default editor to `nano`, run the `export EDITOR=nano`.

```
[student@demo ~]$ ansible-vault create secret.yml
New Vault password: redhat
Confirm New Vault password: redhat
```

Also can use a Vault password file to store the Vault password with param `--vault-password-file=FILE_NAME`. 

```
[student@demo ~]$ ansible-vault create --vault-password-file=vault-pass secret.yml
```

#### Viewing an Encrypted File

Use the `ansible-vault view *`filename`*` command to view an Ansible Vault-encrypted file without opening it for editing.

```
[student@demo ~]$ ansible-vault view secret1.yml
Vault password: secret
my_secret: "yJJvPqhsiusmmPPZdnjndkdnYNDjdj782meUZcw"
```

#### Editing an Existing Encrypted File

Use `ansible-vault edit *`filename`*` command. This command decrypts the file to a temporary file and allows you to edit it. When saved, it copies the content and removes the temporary file.

```
[student@demo ~]$ ansible-vault edit secret.yml
Vault password: redhat
```

**Note**

The `edit` subcommand always rewrites the file, so you should only use it when making changes. This can have implications when the file is kept under version control. You should always use the `view` subcommand to view the file's contents without making changes.

#### Encrypting an Existing File

Use the `ansible-vault encrypt *`filename`*` command. This command can take the names of multiple files to be encrypted as arguments.

```
[student@demo ~]$ ansible-vault encrypt secret1.yml secret2.yml
New Vault password: redhat
Confirm New Vault password: redhat
Encryption successful
```

Use the `--output=OUTPUT_FILE` option to save the encrypted file with a new name. You can only use one input file with the `--output` option.

#### Decrypting an Existing File

Use the `ansible-vault decrypt *`filename`*` command. When decrypting a single file, you can use the `--output` option to save the decrypted file under a different name.

```
[student@demo ~]$ ansible-vault decrypt secret1.yml --output=secret1-decrypted.yml
Vault password: redhat
Decryption successful
```

#### Changing the Password of an Encrypted File

Use the `ansible-vault rekey *`filename`*` command to change the password of an encrypted file. This command can rekey multiple data files at the same time. It prompts for the original password and then the new password.

```
[student@demo ~]$ ansible-vault rekey secret.yml
Vault password: redhat
New Vault password: RedHat
Confirm New Vault password: RedHat
Rekey successful
```

When using a Vault password file, use the `--new-vault-password-file` option:

```
[student@demo ~]$ ansible-vault rekey \
> --new-vault-password-file=NEW_VAULT_PASSWORD_FILE secret.yml
```



#### Playbooks and Ansible Vault

To run a playbook that accesses files encrypted with Ansible Vault, you need to provide the encryption password to the `ansible-navigator` command. If you do not provide the password, the playbook returns an error:

```
[student@demo ~]$ ansible-navigator run -m stdout test-secret.yml
ERROR! Attempting to decrypt but no vault secrets found
```

You can provide the Vault password using one of the following **options**:

- Prompt interactively
- Specify the Vault password file
- Use the `ANSIBLE_VAULT_PASSWORD_FILE` environment variable

To provide the Vault password interactively, use `--playbook-artifact-enable false` (or `--pae false`) and `--vault-id @prompt` as illustrated in the following example:

```
[student@demo ~]$ ansible-navigator run -m stdout --pae false site.yml \
> --vault-id @prompt
Vault password (default): redhat
```

Or disable the playbook artifact from the following minimal `ansible-navigator.yml` file:

```
ansible-navigator:
  playbook-artifact:
    enable: false
```

Instead of providing the Vault encryption password interactively, you can specify a file that stores the encryption password in plain text by using the `--vault-password-file` option.

**The password must be a string stored as a single line in the file.** Because that file contains the sensitive plain text password, it is vital that it be protected through file permissions and other security measures.

```
[student@demo ~]$ ansible-navigator run -m stdout site.yml \
> --vault-password-file=vault-pw-file
```

You can also use the `ANSIBLE_VAULT_PASSWORD_FILE` environment variable to specify the default location of the password file.

#### Use multiple pass for ansible navigator

To use multiple passwords, pass multiple `--vault-id` or `--vault-password-file` options to the `ansible-navigator` command.

```
[student@demo ~]$ ansible-navigator run -m stdout --pae false site.yml \
> --vault-id one@prompt --vault-id two@prompt
Vault password (one):
Vault password (two):
...output omitted...
```

The Vault IDs `one` and `two` preceding `@prompt` can be anything, and you can even omit them entirely. If you use the `--vault-id *`id`*` option when you encrypt a file with the `ansible-vault` command, then the password for the matching ID is the first password tried when running the `ansible-navigator` command. If it does not match, then `ansible-navigator` tries the other passwords that you provided. The Vault ID `@prompt` with no ID is actually shorthand for `default@prompt`, which means to prompt for the password for Vault ID `default`.

If you are using multiple Vault passwords with your playbook, make sure that each encrypted file is assigned a Vault ID, and that you enter the matching password with that Vault ID when running the playbook. This ensures that the correct password is selected first when decrypting the vault-encrypted file, which is faster than forcing Ansible to try each of the Vault passwords that you provided until it finds the right one.



#### Recommended Practices for Variable File Management

Remember that the preferred way to manage group variables and host variables is to create directories at the playbook level. The `group_vars` directory normally contains variable files with names that match the host groups to which they apply. The `host_vars` directory normally contains variable files with names that match the hostnames of managed hosts to which they apply.

You can use subdirectories within the `group_vars` or `host_vars` directories for each host group or managed host. Those directories can contain multiple variable files, and all of those files are used by the host group or managed host.

In the following example project directory for the `playbook.yml` playbook, members of the `webservers` host group use variables in the `group_vars/webservers/vars` file. The `demo.example.com` host uses the variables in both the `host_vars/demo.example.com/vars` and `host_vars/demo.example.com/vault` files.:

```
.
├── ansible.cfg
├── group_vars
│   └── webservers
│       └── vars
├── host_vars
│   └── demo.example.com
│       ├── vars
│       └── vault
├── inventory
└── playbook.yml
```

If you do create subdirectories for each host group or managed host, most variables for `demo.example.com` can be placed in the `vars` file, but sensitive variables can be kept secret by placing them in the `vault` file. You can use `ansible-vault` to encrypt the `vault` file and leave the `vars` file as plain text.



Playbook variables (as opposed to inventory variables) can also be protected with Ansible Vault. You can place sensitive playbook variables in a separate file that is encrypted with Ansible Vault, then include that encrypted variables file in a playbook by using a `vars_files` directive. This can be useful, because playbook variables take precedence over inventory variables.

**Example:**

**secret.yml** was protected by ansible-vault(pass: redhat)

```yaml
[student@workstation data-secret]$ cat secret.yml 
$ANSIBLE_VAULT;1.1;AES256
36333765303261353032373636386233653935656662613965386230646163396231303364623464
3935303532343139353032333235643831323237633364660a313661303031316263376335306136
66663436313830303766653034313261383962613764356233343163386163353239363633393031
3539626334303834370a666535646135303337323937393438366434333432653639303832663435
36363362626365383534646336363363323135306536623264346461336132323936626538383561
34373266626163623934373736366362326233353630613961313137393235616631646562343961
62633239653635376132663464656265346135636564303535306139363933623831623039346364
37323835333631623130316465366139646135376665303264336165303732363333316363343532
35373230646261303932303837313065353365636364653733386139306166366263666630373830
65303537366636613862346663373762633233633461656230643138633339356666343037626234
323334353065303465663630663738653765

[student@workstation data-secret]$ ansible-vault view secret.yml 
Vault password:
username: ansibleuser1
pwhash: abcdefgh2345690009988767622341123453
```

Here is a playbook`create_users.yml` which should contain one play (`Create user accounts for all our servers` in the following example), which uses the variables defined in the `secret.yml` encrypted file.

Configure the play to use the `devservers` host group. Run this play as the `devops` user on the remote managed host. Configure the play to create the `ansibleuser1` user defined by the `username` variable. Set the user's password using the password hash stored in the `pwhash` variable.

```
---
- name: Create user accounts for all our servers
  hosts: devservers
  become: true
  remote_user: devops
  vars_files:
    - secret.yml
  tasks:
    - name: Creating user from secret.yml
      ansible.builtin.user:
        name: "{{ username }}"
        password: "{{ pwhash }}"
```

Resolve any syntax errors in your playbook before you continue.

```
[student@workstation data-secret]$ ansible-navigator run -m stdout \
> --pae false create_users.yml --syntax-check --vault-id @prompt
Vault password (default): redhat

playbook: /home/student/data-secret/create_users.yml
```

Create a password file named `vault-pass` that contains the password (redhat) and Change the permissions of the file to `0600`.

```
[student@workstation data-secret]$ echo 'redhat' > vault-pass
[student@workstation data-secret]$ chmod 0600 vault-pass
```

Run the Ansible Playbook to create the `ansibleuser1` user on a remote system, using the Vault password in the `vault-pass` file to decrypt the hashed password for that user. That password is stored as a variable in the `secret.yml` Ansible Vault encrypted file.

```
[student@workstation data-secret]$ ansible-navigator run \
> -m stdout create_users.yml --vault-password-file=vault-pass

PLAY [Create user accounts for all our servers] ********************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Creating users from secret.yml] ******************************************
changed: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Verify that the playbook ran correctly. The `ansibleuser1` user should exist and have the correct password on the `servera.lab.example.com` machine.

To make sure that SSH only tries to authenticate by password and not by using an SSH key, use the `-o PreferredAuthentications=password` option when you log in.

```
[student@workstation data-secret]$ ssh -o PreferredAuthentications=password \
> ansibleuser1@servera.lab.example.com
ansibleuser1@servera.lab.example.com's password: redhat
...output omitted...
[ansibleuser1@servera ~]$ exit
logout
Connection to servera.lab.example.com closed.
```

#### References

[Encrypting content with Ansible Vault — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

[Keep vaulted variables safely visible — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#keep-vaulted-variables-safely-visible)





### Managing Facts

#### Describing Ansible Facts

Ansible *facts* are variables that are automatically discovered by Ansible on a managed host. 

Facts contain host-specific information that can be used just like regular variables in plays, conditionals, loops, or any other statement that depends on a value collected from a managed host.

Some facts gathered for a managed host might include:

- The host name
- The kernel version
- Network interface names
- Network interface IP addresses
- Operating system version
- Number of CPUs
- Available or free memory
- Size and free space of storage devices

You can even create *custom facts*, which are stored on the managed host and are unique to that system.

Facts are a convenient way to retrieve the state of a managed host and to determine what action to take based on that state. For example:

- Your play might restart a server by using a conditional task based on the value of a fact that was gathered, such as the status of a particular service.
- The play might customize a MySQL configuration file depending on the available memory that is reported by a fact.
- The IPv4 address used in a configuration file might be set based on the value of a fact.

**Normally, every play runs the `ansible.builtin.setup` module automatically to gather facts, before it performs its first task.**

This is reported as the `Gathering Facts` task in Ansible 2.3 and later, or simply as `setup` in earlier versions of Ansible. By default, you do not need to have a task to run `ansible.builtin.setup` in your play. It is normally run automatically for you.

One way to see what facts are gathered for your managed hosts is to run a short playbook that gathers facts and uses the `ansible.builtin.debug` module to print the value of the `ansible_facts` variable.

```
- name: Fact dump
  hosts: all
  tasks:
    - name: Print all facts
      ansible.builtin.debug:
        var: ansible_facts
```

When you run the playbook, the facts are displayed in the job output(JSON format):

```
[user@demo ~]$ ansible-navigator run -m stdout facts.yml

PLAY [Fact dump] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [demo1.example.com]

TASK [Print all facts] *********************************************************
ok: [demo1.example.com] => {
    "ansible_facts": {
        "all_ipv4_addresses": [
            "10.30.0.178",
            "172.25.250.10"
        ],
        "all_ipv6_addresses": [
            "fe80::8389:96fd:e53e:979",
            "fe80::cb51:6814:6342:7bbc"
        ],
        "ansible_local": {}
            }
        },
        "apparmor": {
            "status": "disabled"
        },
        "architecture": "x86_64",
        "bios_date": "04/01/2014",
        "bios_vendor": "SeaBIOS",
        "bios_version": "1.13.0-2.module+el8.2.1+7284+aa32a2c4",
        "board_asset_tag": "NA",
        "board_name": "NA",
        "board_serial": "NA",
        "board_vendor": "NA",
        "board_version": "NA",
        "chassis_asset_tag": "NA",
        "chassis_serial": "NA",
        "chassis_vendor": "Red Hat",
        "chassis_version": "RHEL 7.6.0 PC (i440FX + PIIX, 1996)",
        "cmdline": {
            "BOOT_IMAGE": "(hd0,gpt3)/vmlinuz-5.14.0-70.13.1.el9_0.x86_64",
            "console": "ttyS0,115200n8",
            "crashkernel": "1G-4G:192M,4G-64G:256M,64G-:512M",
            "net.ifnames": "0",
            "no_timer_check": true,
            "root": "UUID=fb535add-9799-4a27-b8bc-e8259f39a767"
        },
...output omitted...
```



The following table shows some facts that might be gathered from a managed node and which might be useful in a playbook:

**Table 3.3. Examples of Ansible Facts**

| Fact                                        | Variable                                                     |
| :------------------------------------------ | :----------------------------------------------------------- |
| Short hostname                              | `ansible_facts['hostname']`                                  |
| Fully qualified domain name                 | `ansible_facts['fqdn']`                                      |
| Main IPv4 address (based on routing)        | `ansible_facts['default_ipv4']['address']`                   |
| List of the names of all network interfaces | `ansible_facts['interfaces']`                                |
| Size of the `/dev/vda1` disk partition      | `ansible_facts['devices']['vda']['partitions']['vda1']['size']` |
| List of DNS servers                         | `ansible_facts['dns']['nameservers']`                        |
| Version of the currently running kernel     | `ansible_facts['kernel']`                                    |

**Note**:

Remember that when a variable's value is a dictionary, one of two syntaxes can be used to retrieve the value. To take two examples from the preceding table:

- `ansible_facts['default_ipv4']['address']` can also be written `ansible_facts.default_ipv4.address`

When a fact is used in a playbook, Ansible dynamically substitutes the variable name for the fact with the corresponding value:

```
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    ansible.builtin.debug:
      msg: >
        The default IPv4 address of {{ ansible_facts.fqdn }}
        is {{ ansible_facts.default_ipv4.address }}
```

The following output shows how Ansible was able to query the managed node and dynamically use the system information to update the variable. You can also use facts to create dynamic groups of hosts that match particular criteria.

```
[user@demo ~]$ ansible-navigator run -m stdout playbook.yml
PLAY [all] ***********************************************************************

TASK [Gathering Facts] *****************************************************
ok: [demo1.example.com]

TASK [Prints various Ansible facts] ****************************************
ok: [demo1.example.com] => {
    "msg": "The default IPv4 address of demo1.example.com is 172.25.250.10\n"
}

PLAY RECAP *****************************************************************
demo1.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Ansible Facts Injected as Variables

Before Ansible 2.5, facts were always injected as individual variables prefixed with the string `ansible_` instead of being part of the `ansible_facts` variable. For example, the `ansible_facts['distribution']` fact was called `ansible_distribution`.

Many playbooks still use facts injected as variables instead of the new syntax, which uses the `ansible_facts.*` namespace.

One reason why the Ansible community discourages injecting facts as variables is because it risks unexpected collisions between facts and variables. A fact has a very high precedence that overrides playbook and inventory host and group variables, so this can lead to unexpected side effects.

The following table shows some examples of facts with both the `ansible_*` and `ansible_facts.*` names.

**Table 3.4. Comparison of Selected Ansible Fact Names**

| ansible_facts.* name                                         | ansible_* name                                         |
| :----------------------------------------------------------- | :----------------------------------------------------- |
| `ansible_facts['hostname']`                                  | `ansible_hostname`                                     |
| `ansible_facts['fqdn']`                                      | `ansible_fqdn`                                         |
| `ansible_facts['default_ipv4']['address']`                   | `ansible_default_ipv4['address']`                      |
| `ansible_facts['interfaces']`                                | `ansible_interfaces`                                   |
| `ansible_facts['devices']['vda']['partitions']['vda1']['size']` | `ansible_devices['vda']['partitions']['vda1']['size']` |
| `ansible_facts['dns']['nameservers']`                        | `ansible_dns['nameservers']`                           |
| `ansible_facts['kernel']`                                    | `ansible_kernel`                                       |

**Important NOTE:**

Currently, Ansible recognizes both the new fact-naming system (using `ansible_facts`) and the earlier, pre-2.5 "facts injected as separate variables" naming system.

You can disable the `ansible_` naming system by setting the `inject_facts_as_vars` parameter in the `[defaults]` section of the Ansible configuration file to `false`. The default setting is currently `true`.

If it is set to `false`, you can only reference Ansible facts using the new `ansible_facts.*` naming system. In that case, attempts to reference facts through the `ansible_*` namespace results in an error.

#### Turning off Fact Gathering

Sometimes, you do not want to gather facts for your play. This might be for several reasons:

- You might not be using any facts and want to **speed up** the play, or reduce load caused by the play on the managed hosts.
- The managed hosts perhaps cannot run the `ansible.builtin.setup` module for some reason, or you need to install some prerequisite software before gathering facts.

To disable fact gathering for a play, set the `gather_facts` keyword to `false`:

```
---
- name: This play does not automatically gather any facts
  hosts: large_datacenter
  gather_facts: false
```

Even if `gather_facts: false` is set for a play, you can manually gather facts at any time by running a task that uses the `ansible.builtin.setup` module:

```
  tasks:
    - name: Manually gather facts
      ansible.builtin.setup:
```

#### Gathering a Subset of Facts

All facts are gathered by default. You can configure the `ansible.builtin.setup` module to only gather a subset of facts, instead of all facts. For example, to only gather hardware facts, set `gather_subset` to `hardware`:

```
- name: Collect only hardware facts
  ansible.builtin.setup:
    gather_subset:
      - hardware
```

If you want to gather all facts except a certain subset, **add an exclamation point (!) in front of the subset name**. Add quotes around the string because in YAML the exclamation point cannot be used at the start of an unquoted string.

```
- name: Collect all facts except for hardware facts
  ansible.builtin.setup:
    gather_subset:
      - "!hardware"
```

Visit https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html#parameter-gather_subset to view possible values for the `gather_subset` parameter.

#### Creating Custom Facts

You can use *custom facts* to define certain values for managed hosts. Plays can use custom facts to populate configuration files or conditionally run tasks.

Custom facts are **stored locally** on **each managed host**. These facts are integrated into the list of standard facts gathered by the `ansible.builtin.setup` module when it runs on the managed host.

You can statically define custom facts in an INI or JSON file, or you can generate them dynamically when you run a play. Dynamic custom facts are gathered via executable scripts, which generate JSON output.

By default, the `ansible.builtin.setup` module loads custom facts from files and scripts in the `etc/ansible/facts.d` directory of each managed host. The name of each file or script must end in `.fact` for it to be used. Dynamic custom fact scripts must output JSON-formatted facts and must be executable.

The following example static custom facts file is written in INI format. An INI-formatted custom facts file contains a top level defined by a section, followed by the key-value pairs of the facts to define:

```
[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = joe
user2 = jane
```

You can provide the same facts in JSON format. The following JSON facts are equivalent to the facts specified by the INI format in the preceding example. The JSON data could be stored in a static text file or printed to standard output by an executable script:

```
{
  "packages": {
    "web_package": "httpd",
    "db_package": "mariadb-server"
  },
  "users": {
    "user1": "joe",
    "user2": "jane"
  }
}
```

**Note**:

Custom fact files cannot be in YAML format like a playbook. JSON format is the closest equivalent.

The `ansible.builtin.setup` module stores custom facts in the `ansible_facts['ansible_local']` variable. Facts are organized based on the name of the file that defined them. For example, assume that the `/etc/ansible/facts.d/custom.fact` file on the managed host produces the preceding custom facts. In that case, the value of `ansible_facts['ansible_local']['custom']['users']['user1']` is `joe`.

You can inspect the structure of your custom facts by gathering facts and using the `ansible.builtin.debug` module to display the contents of the `ansible_local` variable with a play similar to the following example:

```
- name: Custom fact testing
  hosts: demo1.example.com
  gather_facts: true

  tasks:
    - name: Display all facts in ansible_local
      ansible.builtin.debug:
        var: ansible_local
```

When you run the play, you might see output similar to the following example:

```
...output omitted...
TASK [Display all facts in ansible_local] *********************************
ok: [demo1.example.com] => {
    "ansible_local": {
        "custom": {
            "packages": {
                "db_package": "mariadb-server",
                "web_package": "httpd"
            },
            "users": {
                "user1": "joe",
                "user2": "jane"
            }
        }
    }
}
...output omitted...
```

You can use custom facts the same way as default facts in playbooks:

```
[user@demo ~]$ cat playbook.yml
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    ansible.builtin.debug:
      msg: >
           The package to install on {{ ansible_facts['fqdn'] }}
           is {{ ansible_facts['ansible_local']['custom']['packages']['web_package'] }}

[user@demo ~]$ ansible-navigator run -m stdout playbook.yml
PLAY [all] ***********************************************************************

TASK [Gathering Facts] *****************************************************
ok: [demo1.example.com]

TASK [Prints various Ansible facts] ****************************************
ok: [demo1.example.com] => {
    "msg": "The package to install on demo1.example.com is httpd"
}

PLAY RECAP *****************************************************************
demo1.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Creating Variables from Other Variables

Sometimes you might want to create a new variable that uses the value of a different variable. One reason to create a new variable is to minimize typing.

The previous example created custom facts on managed hosts. In that example, you can use the `ansible_facts['ansible_local']['custom']` variable to reference those custom facts. That variable has the `packages` and `users` keys. If your play contains several references to those custom facts, then you might benefit from creating a new variable.

You can use the `ansible.builtin.set_fact` module to create a new variable associated to the current host. For example, you might define the `custom_host` variable and use the `ansible_facts['ansible_local']['custom']` variable as its value.

```
- name: Set custom_host
  ansible.builtin.set_fact:
    custom_host: "{{ ansible_facts['ansible_local']['custom'] }}"
```

By defining this new variable, your play can use the shorter `custom_host['packages']` and `custom_host['users']` variables rather than the longer `ansible_facts['ansible_local']['custom']['packages']` and `ansible_facts['ansible_local']['custom']['users']` variables.

You might also use the `ansible.builtin.set_fact` module to minimize typing for regular system facts or for registered variables. For example:

```
- name: Set vda_parts
  ansible.builtin.set_fact:
    vda_parts: "{{ ansible_facts['devices']['vda']['partitions'] }}"
```

After adding this task, your play can use the `vda_parts['vda1']['size']` variable rather than the longer `ansible_facts['devices']['vda']['partitions']['vda1']['size']` variable.

#### Using Magic Variables

Ansible sets some special variables automatically.

These *magic variables* can also be useful to get information specific to a particular managed host.

Magic variable names are reserved, so you should not define variables with these names.

Four of the most useful magic variables are:

- `hostvars`

  Contains the variables for managed hosts, and can be used to get the values for another managed host's variables. It does not include the managed host's facts if they have not yet been gathered for that host.

- `group_names`

  Lists all groups that the current managed host is in.

- `groups`

  Lists all groups and hosts in the inventory.

- `inventory_hostname`

  Contains the hostname for the current managed host as configured in the inventory. This might be different from the hostname reported by facts for various reasons.

One way to get insight into their values is to use the `ansible.builtin.debug` module to display the contents of these variables.

For example, the following task causes every host that runs the play to print out a list of all network interfaces on the `demo2.example.com` host. This task works as long as facts were gathered for `demo2` earlier in the play or by a preceding play in the playbook. It uses the `hostvars` magic variable to access the `ansible_facts['interfaces']` fact for that host.

```
    - name: Print list of network interfaces for demo2
      ansible.builtin.debug:
        var: hostvars['demo2.example.com']['ansible_facts']['interfaces']
```

You can use the same approach with regular variables, not only facts. Keep in mind that the preceding task is run by every host in the play, so it would be more efficient to use a different module to apply information gathered from one host to the configuration of each of those other managed hosts.

Remember that you can use the `ansible.builtin.setup` module in a task to refresh gathered facts at any time. However, fact gathering does cause your playbook to take longer to run.

Several other magic variables are also available. For more information, see `https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html`.

#### References

[Ansible facts - Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html#ansible-facts)

[ansible.builtin.setup module - Gathers facts about remote hosts - Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)

[Special Variables - Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)





### Chapter 3 TEST

**Instructions**

1. In the working directory, create the `playbook.yml` playbook. In the playbook, start creating a play to install and configure the web server hosts with an Apache HTTP Server that has basic authentication enabled. Configure the `webserver` host group to contain the managed hosts for the play.

   Define the following play variables:

   | Variable         | Values                         |
   | :--------------- | :----------------------------- |
   | `firewall_pkg`   | `firewalld`                    |
   | `firewall_svc`   | `firewalld`                    |
   | `web_pkg`        | `httpd`                        |
   | `web_svc`        | `httpd`                        |
   | `ssl_pkg`        | `mod_ssl`                      |
   | `httpdconf_src`  | `files/httpd.conf`             |
   | `httpdconf_dest` | `/etc/httpd/conf/httpd.conf`   |
   | `htaccess_src`   | `files/.htaccess`              |
   | `secrets_dir`    | `/etc/httpd/secrets`           |
   | `secrets_src`    | `files/htpasswd`               |
   | `secrets_dest`   | `"{{ secrets_dir }}/htpasswd"` |
   | `web_root`       | `/var/www/html`                |

   

2. Add a `tasks` section to the play. Write a task that ensures the latest version of the necessary packages are installed. These packages are defined by the `firewall_pkg`, `web_pkg`, and `ssl_pkg` variables.

   

3. Add a second task to the play that ensures that the file specified by the `httpdconf_src` variable has been copied (with the `ansible.builtin.copy` module) to the location specified by the `httpdconf_dest` variable on the managed host. The file must be owned by the `root` user and the `root` group. Set `0644` as the file permissions.

   

4. Add a third task that uses the `ansible.builtin.file` module to create the directory specified by the `secrets_dir` variable on the managed host. This directory holds the password files used for the basic authentication of web services. The directory must be owned by the `apache` user and the `apache` group. Set `0500` as the directory permissions.

   

5. Add a fourth task that uses the `ansible.builtin.copy` module to add an `htpasswd` file, used for basic authentication of web users. The source should be defined by the `secrets_src` variable. The destination should be defined by the `secrets_dest` variable. The file must be owned by the `apache` user and group. Set `0400` as the file permissions.

   

6. Add a fifth task that uses the `ansible.builtin.copy` module to create a `.htaccess` file in the document root directory of the web server. Copy the file specified by the `htaccess_src` variable to `{{ web_root }}/.htaccess`. The file must be owned by the `apache` user and the `apache` group. Set `0400` as the file permissions.

   

7. Add a sixth task that uses the `ansible.builtin.copy` module to create the web content file, `index.html`, in the directory specified by the `web_root` variable. The file should contain the message `*`HOSTNAME`* (*`IPADDRESS`*) has been customized by Ansible.`, where `HOSTNAME` is the fully qualified host name of the managed host and `IPADDRESS` is its IPv4 IP address. Use the `content` option with the `ansible.builtin.copy` module to specify the content of the file, and Ansible facts to specify the host name and IP address.

   

8. Add a seventh task that uses the `ansible.builtin.service` module to enable and start the firewall service on the managed host.

   

9. Add an eighth task that uses the `ansible.posix.firewalld` module to enable access to the `https` service that is needed for users to access web services on the managed host. This firewall change should be permanent and should take place immediately.

   

10. Add a final task that uses the `ansible.builtin.service` module to enable and start the web service on the managed host for all configuration changes to take effect. The name of the web service is defined by the `web_svc` variable.

    

11. Define a second play in the `playbook.yml` file that uses the `workstation` machine as the managed host to test authentication to the web server. It does not need privilege escalation. Define a variable named `web_user` with the value `guest`.

    

12. Add a directive to the play that adds additional variables from a variable file named `vars/secret.yml`. This file contains a variable named `web_pass` that specifies the password for the web user. You create this file later in the lab.

    Define the start of the task list.

    

13. Add two tasks to the second play.

    The first task uses the `ansible.builtin.uri` module to request content from `https://serverb.lab.example.com` using basic authentication. Use the `web_user` and `web_pass` variables to authenticate to the web server. The task should verify a return HTTP status code of `200`. Register the task result in a variable named `auth_test`.

    Note that the certificate presented by `serverb` is not trusted, so you need to avoid certificate validation.

    The second task uses the `ansible.builtin.debug` module to print the content returned from the web server, which is contained in the `auth_test` variable.

    

14. Create a `vars/secret.yml` file, encrypted with Ansible Vault. Use the password `redhat` to encrypt it. It should set the `web_pass` variable to `redhat`, which is the web user's password.

    1. Create a subdirectory named `vars` in the working directory.

       ```
       [student@workstation data-review]$ mkdir vars
       ```

    2. Create the encrypted variable file, `vars/secret.yml`, using Ansible Vault. Set the password for the encrypted file to `redhat`.

       ```
       [student@workstation data-review]$ ansible-vault create vars/secret.yml
       New Vault password: redhat
       Confirm New Vault password: redhat
       ```

    3. Add the following variable definition to the file:

       ```
       web_pass: redhat
       ```

    4. Save and close the file.

    

15. Run the `playbook.yml` playbook. Verify that content is successfully returned from the web server, and that it matches what was configured in an earlier task.

playbook.yml

```yaml
---
- name: Webserver with variable test
  hosts: webserver
  vars:
    firewall_pkg: firewalld
    firewall_svc: firewalld
    web_pkg: httpd
    web_svc: httpd
    ssl_pkg: mod_ssl
    httpdconf_src: files/httpd.conf
    httpdconf_dest: /etc/httpd/conf/httpd.conf
    htaccess_src: files/.htaccess
    secrets_dir:  /etc/httpd/secrets
    secrets_src:  files/htpasswd
    secrets_dest: "{{ secrets_dir }}/htpasswd"
    web_root: /var/www/html

  tasks:
    - name: Installing latest packages for PKG
      ansible.builtin.dnf:
        name:
          - "{{ firewall_pkg }}"
          - "{{ web_pkg }}"
          - "{{ ssl_pkg }}"
        state: latest

    - name: Copying FILES
      ansible.builtin.copy:
        src: "{{ httpdconf_src }}"
        dest: "{{ httpdconf_dest }}"
        owner: root
        group: root
        mode: 0644

    - name: Creating Secret folder
      ansible.builtin.file:
        path: "{{ secrets_dir }}"
        state: directory
        owner: apache
        group: apache
        mode: 0500

    - name: Coping secret files
      ansible.builtin.copy:
        src: "{{ secrets_src }}"
        dest: "{{ secrets_dest }}"
        owner: apache
        group: apache
        mode: 0400

    - name: Coping htaccess files
      ansible.builtin.copy:
        src: "{{ htaccess_src }}"
        dest: "{{ web_root }}/.htaccess"
        owner: apache
        group: apache
        mode: 0400

    - name: Coping webcontent files
      ansible.builtin.copy:
        dest: "{{ web_root }}/index.html"
        content: >
          {{ ansible_facts['fqdn'] }}({{ansible_facts['default_ipv4']['address']}}) has been customized by Ansible.

    - name: Starting firewall service
      ansible.builtin.service:
        name: "{{ firewall_svc }}"
        state: started
        enabled: true

    - name: Enable httpd access through firewall
      ansible.posix.firewalld:
        service: https
        permanent: true
        immediate: true
        state: enabled

    - name: Starting web service
      ansible.builtin.service:
        name: "{{ web_svc }}"
        state: started
        enabled: true

- name: Testing webservice
  hosts: workstation
  become: false
  vars:
    web_user: guest

  vars_files:
    - vars/secret.yml

  tasks:
    - name: Accessing the web service
      ansible.builtin.uri:
        url: https://serverb.lab.example.com
        force_basic_auth: true
        user: "{{ web_user }}"
        password: "{{ web_pass }}"
        validate_certs: false
        return_content: true
        status_code: 200

      register: auth_test

    - name: debug info
      ansible.builtin.debug:
        var: auth_test['content']
```

result:

```
[student@workstation data-review]$ ansible-navigator run -m stdout playbook.yml --pae false --vault-id @prompt
Vault password (default): 

PLAY [Webserver with variable test] ******************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [serverb.lab.example.com]

TASK [Installing latest packages for PKG] ************************************************************
ok: [serverb.lab.example.com]

TASK [Copying FILES] *********************************************************************************
ok: [serverb.lab.example.com]

TASK [Creating Secret folder] ************************************************************************
ok: [serverb.lab.example.com]

TASK [Coping secret files] ***************************************************************************
ok: [serverb.lab.example.com]

TASK [Coping htaccess files] *************************************************************************
ok: [serverb.lab.example.com]

TASK [Coping webcontent files] ***********************************************************************
ok: [serverb.lab.example.com]

TASK [Starting firewall service] *********************************************************************
ok: [serverb.lab.example.com]

TASK [Enable httpd access through firewall] **********************************************************
ok: [serverb.lab.example.com]

TASK [Starting web service] **************************************************************************
ok: [serverb.lab.example.com]

PLAY [Testing webservice] ****************************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [workstation]

TASK [Accessing the web service] *********************************************************************
ok: [workstation]

TASK [debug info] ************************************************************************************
ok: [workstation] => {
    "auth_test['content']": "\"serverb.lab.example.com\" (172.25.250.11) has been customized by Ansible.\n        \n"
}

PLAY RECAP *******************************************************************************************
serverb.lab.example.com    : ok=10   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
workstation                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

---

## Chapter 4. Implementing Task Control

### Writing Loops and Conditional Tasks

#### Task Iteration with Loops

Using loops makes it possible to avoid writing multiple tasks that use the same module.

To iterate a task over a set of items, you can use the `loop` keyword. You can configure loops to repeat a task using each item in a list, the contents of each of the files in a list, a generated sequence of numbers, or using more complicated structures.

##### Simple Loops

A simple loop iterates a task over a list of items. The `loop` keyword is added to the task, and takes as a value the list of items over which the task should be iterated. The loop variable `item` holds the value used during each iteration.

Consider the following snippet that uses the `ansible.builtin.service` module twice to ensure that two network services are running:

```yaml
- name: Postfix is running
  ansible.builtin.service:
    name: postfix
    state: started

- name: Dovecot is running
  ansible.builtin.service:
    name: dovecot
    state: started
```

These two tasks can be rewritten to use a simple loop so that only one task is needed to ensure that both services are running:

```yaml
- name: Postfix and Dovecot are running
  ansible.builtin.service:
    name: "{{ item }}"
    state: started
  loop:
    - postfix
    - dovecot
```

The loop can use a list provided by a variable.

In the following example, the `mail_services` variable contains the list of services that need to be running.

```yaml
vars:
  mail_services:
    - postfix
    - dovecot

tasks:
  - name: Postfix and Dovecot are running
    ansible.builtin.service:
      name: "{{ item }}"
      state: started
    loop: "{{ mail_services }}"
```

##### Loops over a List of Dictionaries

The loop list does not need to be a list of simple values.

In the following example, each item in the list is actually a dictionary. Each dictionary in the example has two keys, `name` and `groups`, and the value of each key in the current `item` loop variable can be retrieved with the `item['name']` and `item['groups']` variables, respectively.

```yaml
- name: Users exist and are in the correct groups
  user:
    name: "{{ item['name'] }}"
    state: present
    groups: "{{ item['groups'] }}"
  loop:
    - name: jane
      groups: wheel
    - name: joe
      groups: root
```

The outcome of the preceding task is that the user `jane` is present and a member of the group `wheel`, and that the user `joe` is present and a member of the group `root`.

##### Earlier-style Loop Keywords

Before Ansible 2.5, most playbooks used a different syntax for loops. Multiple loop keywords were provided, which used the `with_` prefix, followed by the name of an Ansible look-up plug-in (an advanced feature not covered in detail in this course). This syntax for looping is very common in existing playbooks, **but will probably be deprecated at some point in the future.**

Some examples are listed in the following table:



**Table 4.1. Earlier-style Ansible Loops**

| Loop keyword    | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `with_items`    | Behaves the same as the `loop` keyword for simple lists, such as a list of strings or a list of dictionaries. Unlike `loop`, if lists of lists are provided to `with_items`, they are flattened into a single-level list. The `item` loop variable holds the list item used during each iteration. |
| `with_file`     | Requires a list of control node file names. The `item` loop variable holds the content of a corresponding file from the file list during each iteration. |
| `with_sequence` | Requires parameters to generate a list of values based on a numeric sequence. The `item` loop variable holds the value of one of the generated items in the generated sequence during each iteration. |

The following playbook shows an example of the `with_items` keyword:

```yaml
  vars:
    data:
      - user0
      - user1
      - user2
  tasks:
    - name: "with_items"
      ansible.builtin.debug:
        msg: "{{ item }}"
      with_items: "{{ data }}"
```

**Important note:**

Since Ansible 2.5, the recommended way to write loops is to use the `loop` keyword.

However, you should still understand the earlier syntax, especially `with_items`, because it is widely used in existing playbooks. You are likely to encounter playbooks and roles that continue to use `with_*` keywords for looping.

 The Ansible documentation contains a good reference on how to convert the earlier loops to the new syntax, as well as examples of how to loop over items that are not simple lists. See the ["Migrating from with_X to loop"](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#migrating-from-with-x-to-loop) section of the *Ansible User Guide*.



##### Using Register Variables with Loops

The `register` keyword can also capture the output of a task that loops. The following snippet shows the structure of the `register` variable from a task that loops:

```yaml
[student@workstation loopdemo]$ cat loop_register.yml
---
- name: Loop Register Test
  gather_facts: false
  hosts: localhost
  tasks:
    - name: Looping Echo Task
      ansible.builtin.shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results

    - name: Show echo_results variable
      ansible.builtin.debug:
        var: echo_results
```

Running the preceding playbook yields the following output:

```
[student@workstation loopdemo]$ ansible-navigator run -m stdout loop_register.yml

PLAY [Loop Register Test] ******************************************************

TASK [Looping Echo Task] *******************************************************
changed: [localhost] => (item=one)
changed: [localhost] => (item=two)

TASK [Show echo_results variable] **********************************************
ok: [localhost] => {
    "echo_results": {
        "changed": true,
        "msg": "All items completed",
        "results": [
            {
                "ansible_loop_var": "item",
                "changed": true,
                "cmd": "echo This is my item: one",
                "delta": "0:00:00.004519",
                "end": "2022-06-29 17:32:54.065165",
                "failed": false,
                ...output omitted...
                "item": "one",
                "msg": "",
                "rc": 0,
                "start": "2022-06-29 17:32:54.060646",
                "stderr": "",
                "stderr_lines": [],
                "stdout": "This is my item: one",
                "stdout_lines": [
                    "This is my item: one"
                ]
            },
            {
                "ansible_loop_var": "item",
                "changed": true,
                "cmd": "echo This is my item: two",
                "delta": "0:00:00.004175",
                "end": "2022-06-29 17:32:54.296940",
                "failed": false,
                ...output omitted...
                "item": "two",
                "msg": "",
                "rc": 0,
                "start": "2022-06-29 17:32:54.292765",
                "stderr": "",
                "stderr_lines": [],
                "stdout": "This is my item: two",
                "stdout_lines": [
                    "This is my item: two"
                ]
            }
        ],
        "skipped": false
    }
}
...output omitted...
```

In the preceding example, the `results` key contains a list. In the next example, the playbook is modified so that the second task iterates over this list:

```yaml
# new_loop_register.yml
---
- name: Loop Register Test
  gather_facts: false
  hosts: localhost
  tasks:
    - name: Looping Echo Task
      ansible.builtin.shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results

    - name: Show stdout from the previous task.
      ansible.builtin.debug:
        msg: "STDOUT from previous task: {{ item['stdout'] }}"
      loop: "{{ echo_results['results'] }}"
```

After running the preceding playbook, you see the following output:

```
PLAY [Loop Register Test] ******************************************************

TASK [Looping Echo Task] *******************************************************
changed: [localhost] => (item=one)
changed: [localhost] => (item=two)

TASK [Show stdout from the previous task.] *************************************
ok: [localhost] => (item={'changed': True, 'stdout': 'This is my item: one', 'stderr': '', 'rc': 0, 'cmd': 'echo This is my item: one', 'start': '2022-06-29 17:41:15.558529', 'end': '2022-06-29 17:41:15.563615', 'delta': '0:00:00.005086', 'msg': '', 'invocation': {'module_args': {'_raw_params': 'echo This is my item: one', '_uses_shell': True, 'warn': False, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stdout_lines': ['This is my item: one'], 'stderr_lines': [], 'failed': False, 'item': 'one', 'ansible_loop_var': 'item'}) => {
    "msg": "STDOUT from previous task: This is my item: one"
}
ok: [localhost] => (item={'changed': True, 'stdout': 'This is my item: two', 'stderr': '', 'rc': 0, 'cmd': 'echo This is my item: two', 'start': '2022-06-29 17:41:15.810566', 'end': '2022-06-29 17:41:15.814932', 'delta': '0:00:00.004366', 'msg': '', 'invocation': {'module_args': {'_raw_params': 'echo This is my item: two', '_uses_shell': True, 'warn': False, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stdout_lines': ['This is my item: two'], 'stderr_lines': [], 'failed': False, 'item': 'two', 'ansible_loop_var': 'item'}) => {
    "msg": "STDOUT from previous task: This is my item: two"
}
...output omitted...
```

#### Running Tasks Conditionally

Ansible can use *conditionals* to run tasks or plays when certain conditions are met. 

The following scenarios illustrate the use of conditionals in Ansible.

- Define a hard limit in a variable (for example, `min_memory`) and compare it against the available memory on a managed host.
- Capture the output of a command and evaluate it to determine whether a task completed before taking further action. For example, if a program fails, then a batch is skipped.
- Use Ansible facts to determine the managed host network configuration and decide which template file to send (for example, network bonding or trunking).
- Evaluate the number of CPUs to determine how to properly tune a web server.
- Compare a registered variable with a predefined variable to determine if a service changed. For example, test the MD5 checksum of a service configuration file to see if the service is changed.

##### Conditional Task Syntax

The `when` statement is used to run a task conditionally. It takes as a value the condition to test. If the condition is met, the task runs. If the condition is not met, the task is skipped.

One of the simplest conditions that can be tested is whether a Boolean variable is true or false. The `when` statement in the following example causes the task to run only if `run_my_task` is true.

```yaml
---
- name: Simple Boolean Task Demo
  hosts: all
  vars:
    run_my_task: true

  tasks:
    - name: httpd package is installed
      ansible.builtin.dnf:
        name: httpd
      when: run_my_task
```

**Note:**

Boolean variables can have the value `true` or `false`.

In Ansible content, you can express those values in other ways(NTOE: the newest YAML only allow to use `true/false` for boolean ):

-  `True`, `yes`, or `1`  

- `False`, `no`, or `0` 

Starting with Ansible Core 2.12, **strings** are always treated by `when` conditionals as `true` Booleans if they contain any content.

Therefore, if the `run_my_task` variable in the preceding example were written as shown in the following example then it would be treated as a string with content and have the Boolean value `true`, and the task would run. This is probably not the behavior that you want.

```yaml
  run_my_task: "false"
```

If it had been written as shown in the next example, however, it would be treated as the Boolean value `false` and the task would *not* run:

```yaml
  run_my_task: false
```

To ensure that this is the case, you could rewrite the previous `when` condition to convert an accidental string value to a Boolean and to pass Boolean values unchanged:

```yaml
      when: run_my_task | bool
```

The next example is a bit more sophisticated, and tests whether the `my_service` variable has a value. If it does, the value of `my_service` is used as the name of the package to install. If the `my_service` variable is not defined, then the task is skipped without an error.

```yaml
---
- name: Test Variable is Defined Demo
  hosts: all
  vars:
    my_service: httpd

  tasks:
    - name: "{{ my_service }} package is installed"
      ansible.builtin.dnf:
        name: "{{ my_service }}"
      when: my_service is defined
```

The following table shows some operations that you can use when working with conditionals:



**Table 4.2. Example Conditionals**

| Operation                                                    | Example                                              |
| :----------------------------------------------------------- | :--------------------------------------------------- |
| Equal (value is a string)                                    | `ansible_facts['machine'] == "x86_64"`               |
| Equal (value is numeric)                                     | `max_memory == 512`                                  |
| Less than                                                    | `min_memory < 128`                                   |
| Greater than                                                 | `min_memory > 256`                                   |
| Less than or equal to                                        | `min_memory <= 256`                                  |
| Greater than or equal to                                     | `min_memory >= 512`                                  |
| Not equal to                                                 | `min_memory != 512`                                  |
| Variable exists                                              | `min_memory is defined`                              |
| Variable does not exist                                      | `min_memory is not defined`                          |
| Boolean variable is `true`. The values of `1`, `True`, or `yes` evaluate to `true`. | `memory_available`                                   |
| Boolean variable is `false`. The values of `0`, `False`, or `no` evaluate to `false`. | `not memory_available`                               |
| First variable's value is present as a value in second variable's list | `ansible_facts['distribution'] in supported_distros` |

The last entry in the preceding table might be confusing at first. The following example illustrates how it works.

In the example, the `ansible_facts['distribution']` variable is a fact determined during the `Gathering Facts` task, and identifies the managed host's operating system distribution. The `supported_distros` variable was created by the playbook author, and contains a list of operating system distributions that the playbook supports. If the value of `ansible_facts['distribution']` is in the `supported_distros` list, the conditional passes and the task runs.

```yaml
---
- name: Demonstrate the "in" keyword
  hosts: all
  gather_facts: true
  vars:
    supported_distros:
      - RedHat
      - Fedora
  tasks:
    - name: Install httpd using dnf, where supported
      ansible.builtin.dnf:
        name: http
        state: present
      when: ansible_facts['distribution'] in supported_distros
```

Importantyaml

Observe the indentation of the `when` statement. Because the `when` statement is not a module variable, it must be placed outside the module by being indented at the top level of the task.



##### Testing Multiple Conditions

One `when` statement can be used to evaluate multiple conditionals. To do so, conditionals can be combined with either the `and` or `or` keywords, and grouped with parentheses.

The following snippets show some examples of how to express multiple conditions.

- If a conditional statement should be met when either condition is true, then use the `or` statement. For example, the following condition is met if the machine is running either Red Hat Enterprise Linux or Fedora:

  ```yaml
  when: ansible_facts['distribution'] == "RedHat" or ansible_facts['distribution'] == "Fedora"
  ```

- With the `and` operation, both conditions have to be true for the entire conditional statement to be met. For example, the following condition is met if the remote host is a Red Hat Enterprise Linux 9.0 host, and the installed kernel is the specified version:

  ```yaml
  when: ansible_facts['distribution_version'] == "9.0" and ansible_facts['kernel'] == "5.14.0-70.13.1.el9_0.x86_64"
  ```

  The `when` keyword also supports using a list to describe a list of conditions. When a list is provided to the `when` keyword, all the conditionals are combined using the `and` operation. The example below demonstrates another way to combine multiple conditional statements using the `and` operator:

  ```yaml
  when:
    - ansible_facts['distribution_version'] == "9.0"
    - ansible_facts['kernel'] == "5.14.0-70.13.1.el9_0.x86_64"
  ```

  This format improves readability, a key goal of well-written Ansible Playbooks.

- You can express more complex conditional statements by grouping conditions with parentheses. This ensures that they are correctly interpreted.

  For example, the following conditional statement is met if the machine is running either Red Hat Enterprise Linux 9 or Fedora 34. This example uses the greater-than character (>) so that the long conditional can be split over multiple lines in the playbook, to make it easier to read.

  ```yaml
  when: >
      ( ansible_facts['distribution'] == "RedHat" and
        ansible_facts['distribution_major_version'] == "9" )
      or
      ( ansible_facts['distribution'] == "Fedora" and
      ansible_facts['distribution_major_version'] == "34" )
  ```

#### Combining Loops and Conditional Tasks

You can combine loops and conditionals.

In the following example, the `ansible.builtin.dnf` module installs the `mariadb-server` package if there is a file system mounted on `/` with more than 300 MiB free. The `ansible_facts['mounts']` fact is a list of dictionaries, each one representing facts about one mounted file system. The loop iterates over each dictionary in the list, and the conditional statement is not met unless a dictionary is found that represents a mounted file system where both conditions are true.

```yaml
- name: install mariadb-server if enough space on root
  ansible.builtin.dnf:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_facts['mounts'] }}"
  when: item['mount'] == "/" and item['size_available'] > 300000000
```

Important

When you use `when` with `loop` for a task, the `when` statement is checked for each item.

The following example also combines conditionals and `register` variables. This playbook restarts the `httpd` service only if the `postfix` service is running:

```yaml
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      ansible.builtin.command: /usr/bin/systemctl is-active postfix 
      register: result

    - name: Restart Apache HTTPD based on Postfix status
      ansible.builtin.service:
        name: httpd
        state: restarted
      when: result.rc == 0
```

#### References

[Loops — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)

[Tests — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)

[Conditionals — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)

[What Makes A Valid Variable Name — Variables — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#what-makes-a-valid-variable-name)

For more information on the change to Boolean handling in conditionals in community Ansible 5 (and Ansible Core 2.12) and later, see https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_5.html#deprecated

#### Example

```yaml
# playbook.yml 
# - The first task installs the MariaDB required packages only when hosts are Redhat, and the second task ensures that the MariaDB service is running.

---
- name: Test example for control flow
  hosts: database_prod
  vars:
    mariadb_packages:
      - mariadb-server
      - python3-PyMySQL
  
  tasks:
    - name: Installing MariaDB
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ mariadb_packages }}"
      when: ansible_facts['distribution'] == "RedHat"

    - name: Starting MariaDB
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: true
```





### Implementing Handlers

#### Ansible Handlers

*Handlers* are tasks that respond to a notification triggered by other tasks. Tasks only notify their handlers when the task changes something on a managed host. Each handler is triggered by its name after the play's block of tasks.

If no task notifies the handler by name then the handler does not run. If one or more tasks notify the handler, **the handler runs once after all other tasks in the play have completed**. Because handlers are tasks, administrators can use the same modules in handlers that they would use for any other task.

Normally, handlers are used to **reboot** hosts and restart services.



**Important**:

Always use **unique names** for your handlers. You might have unexpected results if more than one handler uses the same name.

Handlers can be considered as *inactive* tasks that only get triggered when explicitly invoked using a `notify` statement. The following snippet shows how the Apache server is only restarted by the `restart apache` handler when a configuration file is updated and notifies it:

```yaml
tasks:
  - name: copy demo.example.conf configuration template
    ansible.builtin.template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart apache

handlers:
  - name: restart apache
    ansible.builtin.service:
      name: httpd
      state: restarted
```

In the previous example, the `restart apache` handler is triggered when notified by the `template` task that a change happened. A task might call more than one handler in its `notify` section. Ansible treats the `notify` statement as an array and iterates over the handler names:

```yaml
tasks:
  - name: copy demo.example.conf configuration template
    ansible.builtin.template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart mysql
      - restart apache

handlers:
  - name: restart mysql
    ansible.builtin.service:
      name: mariadb
      state: restarted

  - name: restart apache
    ansible.builtin.service:
      name: httpd
      state: restarted
```

#### Describing the Benefits of Using Handlers

As discussed in the Ansible documentation, there are some important things to remember about using handlers:

- Handlers always run in the order specified by the `handlers` section of the play. They do not run in the order in which they are listed by `notify` statements in a task, or in the order in which tasks notify them.
- Handlers normally run after all other tasks in the play complete. A handler called by a task in the `tasks` part of the playbook does not run until *all* tasks under `tasks` have been processed. (Some minor exceptions to this exist.)
- Handler names exist in a per-play namespace. If two handlers are incorrectly given the same name, only one of them runs.
- Even if more than one task notifies a handler, the handler runs one time. If no tasks notify it, the handler does not run.
- If a task that includes a `notify` statement does not report a `changed` result (for example, a package is already installed and the task reports `ok`), the handler is not notified. Ansible notifies handlers only if the task reports the `changed` status.

**Important:** 

Handlers are meant to perform an extra action when a task makes a change to a managed host. They should not be used as a replacement for normal tasks.

#### References

[Handlers: running operations on change — Ansible Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)

#### Example

```yml
# Handlers example: the configure_webapp.yml playbook file. This playbook installs and configures a web application server. When the web application server configuration changes, the playbook triggers a restart of the appropriate service.


---
- name: Web application server is deployed
  hosts: webapp
  vars:
    packages:
      - nginx
      - php-fpm
      - firewalld
    web_service: nginx
    app_service: php-fpm
    firewall_service: firewalld
    firewall_service_rules:
      - http
    web_config_src: files/nginx.conf.standard
    web_config_dst: /etc/nginx/nginx.conf
    app_config_src: files/php-fpm.conf.standard
    app_config_dst: /etc/php-fpm.conf

  tasks:
    - name: Installing Web application server
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"

    - name: Starting servcices...
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop: 
        - "{{ web_service }}"
        - "{{ app_service }}"
        - "{{ firewall_service }}"

    - name: Going through firewall
      ansible.posix.firewalld:
        service: "{{ item }}"
        permanent: true
        immediate: true
        state: enabled
      loop: "{{ firewall_service_rules }}"

    - name: Downlaoding config files and restart web services
      ansible.builtin.copy:
        src: "{{ web_config_src }}"
        dest: "{{ web_config_dst }}"
        mode: "0644"
      notify:
        - restart web service


    - name: Downlaoding config files and restart web services
      ansible.builtin.copy:
        src: "{{ app_config_src }}"
        dest: "{{ app_config_dst }}"
        mode: "0644"
      notify:
        - restart app service

  handlers:
    - name: restart web service
      ansible.builtin.service:
        name: "{{ web_service }}" 
        state: restarted

    - name: restart app service
      ansible.builtin.service:
        name: "{{ app_service }}" 
        state: restarted

```

Results:

```
# First RUN:
[student@workstation control-handlers]$ ansible-navigator run -m stdout configure_webapp.yml

PLAY [Web application server is deployed] **************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Installing Web application server] ***************************************
ok: [servera.lab.example.com] => (item=nginx)
ok: [servera.lab.example.com] => (item=php-fpm)
ok: [servera.lab.example.com] => (item=firewalld)

TASK [Starting servcices...] ***************************************************
ok: [servera.lab.example.com] => (item=nginx)
ok: [servera.lab.example.com] => (item=php-fpm)
ok: [servera.lab.example.com] => (item=firewalld)

TASK [Going through firewall] **************************************************
ok: [servera.lab.example.com] => (item=http)

TASK [Downlaoding config files and restart web services] ***********************
changed: [servera.lab.example.com]

TASK [Downlaoding config files and restart web services] ***********************
changed: [servera.lab.example.com]

RUNNING HANDLER [restart web service] ******************************************
changed: [servera.lab.example.com]

RUNNING HANDLER [restart app service] ******************************************
changed: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


#Second RUN: (No handler running as no files changed during 2nd time run)
[student@workstation control-handlers]$ ansible-navigator run -m stdout configure_webapp.yml

PLAY [Web application server is deployed] **************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Installing Web application server] ***************************************
ok: [servera.lab.example.com] => (item=nginx)
ok: [servera.lab.example.com] => (item=php-fpm)
ok: [servera.lab.example.com] => (item=firewalld)

TASK [Starting servcices...] ***************************************************
ok: [servera.lab.example.com] => (item=nginx)
ok: [servera.lab.example.com] => (item=php-fpm)
ok: [servera.lab.example.com] => (item=firewalld)

TASK [Going through firewall] **************************************************
ok: [servera.lab.example.com] => (item=http)

TASK [Downlaoding config files and restart web services] ***********************
ok: [servera.lab.example.com]

TASK [Downlaoding config files and restart web services] ***********************
ok: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
[student@workstation control-handlers]$ 
```





### Handling Task Failure

Ansible evaluates the return code of each task to determine whether the task succeeded or failed. Normally, when a task fails Ansible immediately skips all subsequent tasks.

However, sometimes you might want to have play execution continue even if a task fails. For example, you might expect that a particular task could fail, and you might want to recover by conditionally running some other task. A number of Ansible features can be used to manage task errors.

#### Ignoring Task Failure

By default, if a task fails, the play is aborted. However, this behavior can be overridden by ignoring failed tasks. You can use the `ignore_errors` keyword in a task to accomplish this.

The following snippet shows how to use `ignore_errors` in a task to continue playbook execution on the host even if the task fails. For example, if the `notapkg` package does not exist then the `ansible.builtin.dnf` module fails, but having `ignore_errors` set to `true` allows execution to continue.

```yaml
- name: Latest version of notapkg is installed
  ansible.builtin.dnf:
    name: notapkg
    state: latest
  ignore_errors: true
```

#### Forcing Execution of Handlers After Task Failure

Normally when a task fails and the play aborts on that host, any handlers that had been notified by earlier tasks in the play do not run. If you set the `force_handlers: true` keyword on the play, then notified handlers are called even if the play aborted because a later task failed.

**Important:**

If you have `ignore_errors: true` set on a task or for the task's play, if that task fails the failure is ignored. In that case, the play keeps running and handlers still run, even if you have `force_handlers: false` set, unless some other error causes the play to fail.

The following snippet shows how to use the `force_handlers` keyword in a play to force execution of the notified handler even if a subsequent task fails:

```yaml
---
- hosts: all
  force_handlers: true
  tasks:
    - name: a task which always notifies its handler
      ansible.builtin.command: /bin/true
      notify: restart the database

    - name: a task which fails because the package doesn't exist
      ansible.builtin.dnf:
        name: notapkg
        state: latest

  handlers:
    - name: restart the database
      ansible.builtin.service:
        name: mariadb
        state: restarted
```

**Important**:

Remember that handlers are notified when a task reports a `changed` result but are not notified when it reports an `ok` or `failed` result.

If you set `force_handlers: true` on the play, then any handlers that have been notified are run even if a later task failure causes the play to fail. Otherwise, handlers are not run at all when a play fails.

Setting `force_handlers: true` on a play does not cause handlers to be notified for tasks that report `ok` or `failed`; it only causes the handlers to run that have already been notified before the point at which the play failed.

#### Specifying Task Failure Conditions

You can use the `failed_when` keyword on a task to specify which conditions indicate that the task has failed. This is often used with command modules that might successfully execute a command, but where the command's output indicates a failure.

For example, you may check for failure by searching for a word or phrase in the output of a command. The following example shows one way that you can use the `failed_when` keyword in a task:

```yaml
tasks:
  - name: Run user creation script
    ansible.builtin.shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```

The `ansible.builtin.fail` module can also be used to force a task failure. You could instead write that example as two tasks:

```yaml
tasks:
  - name: Run user creation script
    ansible.builtin.shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: true

  - name: Report script failure
    ansible.builtin.fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"
```

You can use the `ansible.builtin.fail` module to provide a clear failure message for the task. This approach also enables delayed failure, which means that you can run intermediate tasks to complete or roll back other changes.



or it can also based on the return code

```yaml
- name: Fail task when both files are identical
  ansible.builtin.raw: diff foo/file1 bar/file2
  register: diff_cmd
  failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2
```

You can also combine multiple conditions for failure. This task will fail if both conditions are true:

```yaml
- name: Check if a file exists in temp and fail task if it does
  ansible.builtin.command: ls /tmp/this_should_not_be_here
  register: result
  failed_when:
    - result.rc == 0
    - '"No such" not in result.stdout'
```

If you want the task to fail when only one condition is satisfied, change the `failed_when` definition to

```
failed_when: result.rc == 0 or "No such" not in result.stdout
```

If you have too many conditions to fit neatly into one line, you can split it into a multi-line YAML value with `>`.

```yaml
- name: example of many failed_when conditions with OR
  ansible.builtin.shell: "./myBinary"
  register: ret
  failed_when: >
    ("No such file or directory" in ret.stdout) or
    (ret.stderr != '') or
    (ret.rc == 10)
```



#### Specifying When a Task Reports "Changed" Results

When a task makes a change to a managed host, it reports the `changed` state and notifies handlers. When a task does not need to make a change, it reports `ok` and does not notify handlers.

Use the `changed_when` keyword to control how a task reports that it has changed something on the managed host. For example, the `ansible.builtin.command` module in the next example validates the `httpd` configuration on a managed host.

This task validates the configuration syntax, but nothing is actually changed on the managed host. Subsequent tasks can use the value of the `httpd_config_status` variable.

It normally would always report `changed` when it runs. To suppress that change report, `changed_when: false` is set so that it only reports `ok` or `failed`.

```yaml
  - name: Validate httpd configuration
    ansible.builtin.command: httpd -t
    changed_when: false
    register: httpd_config_status
```

The following example uses the `ansible.builtin.shell` module and only reports `changed` if the string "Success" is found in the output of the registered variable. If it does report `changed`, then it notifies the handler.

```yaml
tasks:
  - ansible.builtin.shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

handlers:
  - name: restart_database
     ansible.builtin.service:
       name: mariadb
       state: restarted
```

#### Ansible Blocks and Error Handling

In playbooks, *blocks* are clauses that logically group tasks, and can be used to control how tasks are executed. For example, a task block can have a `when` keyword to apply a conditional to multiple tasks:

```yaml
- name: block example
  hosts: all
  tasks:
    - name: installing and configuring DNF versionlock plugin
      block:
      - name: package needed by dnf
        ansible.builtin.dnf:
          name: python3-dnf-plugin-versionlock
          state: present
      - name: lock version of tzdata
        ansible.builtin.lineinfile:
          dest: /etc/yum/pluginconf.d/versionlock.list
          line: tzdata-2016j-1
          state: present
      when: ansible_distribution == "RedHat"
```

Blocks also allow for error handling in combination with the `rescue` and `always` statements. If any task in a `block` fails, then `rescue` tasks are executed to recover.

After the tasks in the `block` clause run, as well as the tasks in the `rescue` clause if there was a failure, then tasks in the `always` clause run.

To summarize:

- `block`: Defines the main tasks to run.
- `rescue`: Defines the tasks to run if the tasks defined in the `block` clause fail.
- `always`: Defines the tasks that always run independently of the success or failure of tasks defined in the `block` and `rescue` clauses.

The following example shows how to implement a block in a playbook.

```yaml
  tasks:
    - name: Upgrade DB
      block:
        - name: upgrade the database
          ansible.builtin.shell:
            cmd: /usr/local/lib/upgrade-database
      rescue:
        - name: revert the database upgrade
          ansible.builtin.shell:
            cmd: /usr/local/lib/revert-database
      always:
        - name: always restart the database
          ansible.builtin.service:
            name: mariadb
            state: restarted
```

The `when` condition on a `block` clause also applies to its `rescue` and `always` clauses if present.

#### References

[Error Handling in Playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)

[Error Handling — Blocks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html#blocks-error-handling)



#### Example

```yaml
---
- name: How to handle errors example
  hosts: databases
  vars:
    web_package: httpd
    db_package: mariadb-server
    db_service: mariadb

  tasks:
    - name: Running date cmd
      ansible.builtin.command: /usr/bin/date
      register: command_result
      changed_when: false       # the above 'date' command task will not reported 'changed' if we set this to false

    - name: Display the cmd result
      ansible.builtin.debug:
        var: command_result.stdout

    - name: Doing block things
      block: 
        - name: Installing web package
          ansible.builtin.dnf:
            name: "{{ web_package }}"
            state: present
          failed_when: web_package == "httpd"   # Force to report failure if the 'web_package' name is httpd

      rescue:  # this will run only there is failure in above 'block'  
        - name: Installing db package
          ansible.builtin.dnf:
            name: "{{ db_package }}"
            state: present
              
      always:
        - name: Anyway Starting DB server
          ansible.builtin.service:
            name: "{{ db_service }}"
            state: started
            enabled: true
```

**Result**: Look carefully at the output. The `failed_when` keyword changes the status that the task reports *after* the task runs; it does not change the behavior of the task itself.

However, the reported failure might change the behavior of the rest of the play. Because that task was in a block and reported that it failed, the `Install mariadb-server package` task in the block's `rescue` section was run.

```
[student@workstation control-errors]$ ansible-navigator run -m stdout playbook.yml

PLAY [How to handle errors example] ********************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Running date cmd] ********************************************************
>>ok: [servera.lab.example.com]<<

TASK [Display the cmd result] **************************************************
ok: [servera.lab.example.com] => {
    "command_result.stdout": "Wed Aug 28 10:42:15 AM EDT 2024"
}

TASK [Installing web package] **************************************************
fatal: [servera.lab.example.com]: FAILED! => {"changed": false, >>"failed_when_result": true<<, "msg": "Nothing to do", "rc": 0, "results": []}

TASK [Installing db package] ***************************************************
ok: [servera.lab.example.com]

TASK [Anyway Starting DB server] ***********************************************
ok: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=5    changed=0    unreachable=0    >>failed=0<<    skipped=0    >>rescued=1<<    ignored=0   
```



### Chapter4 TEST

Requests:

1. Under the `#Fail fast message` comment, add a task that uses the `ansible.builtin.fail` module. Provide an appropriate name for the task.

   This task should only be executed when the remote system does not meet the following minimum requirements:

   - Has at least the amount of RAM specified by the `min_ram_mb` variable. The `min_ram_mb` variable is defined in the `vars.yml` file and has a value of `256`.
   - Is running Red Hat Enterprise Linux.

2. Under the `#Install all packages` comment, add a task named `Ensure required packages are present` to install the latest version of any missing packages. Required packages are specified by the `packages` variable, which is defined in the `vars.yml` file.

3. Under the `#Enable and start services` comment, add a task to start services. All services specified by the `services` variable, which is defined in the `vars.yml` file, should be started and enabled. Provide an appropriate name for the task.

4. Under the `#Block of config tasks` comment, add a task block to the play. This block contains two tasks:

   - A task to ensure that the directory specified by the `ssl_cert_dir` variable exists on the remote host. This directory stores the web server's certificates.

   - A task to copy all files specified by the `web_config_files` variable to the remote host. Examine the structure of the `web_config_files` variable in the `vars.yml` file. Configure the task to copy each file to the correct destination on the remote host.

     This task should trigger the `Restart web service` handler if any of these files are changed on the remote server.

   Additionally, a debug task is executed if either of the two tasks above fail. In this case, the task prints the following message: `One or more of the configuration changes failed, but the web service is still active.`

   Provide an appropriate name for all tasks.

5. The play configures the remote host to listen for standard HTTPS requests. Under the `#Configure the firewall` comment, add a task to configure `firewalld`.

   Ensure that the task configures the remote host to accept standard HTTP and HTTPS connections. The configuration changes must be effective immediately and persist after a reboot. Provide an appropriate name for the task.

6. Define the `Restart web service` handler.

   When triggered, this task should restart the web service defined by the `web_service` variable, defined in the `vars.yml` file.

```yaml
# vars.yml
min_ram_mb: 256

web_service: httpd
web_package: httpd
ssl_package: mod_ssl

fw_service: firewalld
fw_package: firewalld


services:
 - "{{ web_service }}"
 - "{{ fw_service }}"

packages:
 - "{{ web_package }}"
 - "{{ ssl_package }}"
 - "{{ fw_package }}"

ssl_cert_dir: /etc/httpd/conf.d/ssl

web_config_files:
  - src: server.key
    dest: "{{ ssl_cert_dir }}"
  - src: server.crt
    dest: "{{ ssl_cert_dir }}"
  - src: ssl.conf
    dest: /etc/httpd/conf.d
  - src: index.html
    dest: /var/www/html
```



```yaml
# playbook.yml
---
- name: Playbook Control Lab
  hosts: webservers
  vars_files: vars.yml
  tasks:
    #Fail fast message
    - name: Show failed system requirements message
      ansible.builtin.fail:
        msg: "The {{ inventory_hostname }} did not meet minimum reqs."
      when: >
        ansible_facts['memtotal_mb'] < min_ram_mb or
        ansible_facts['distribution'] != "RedHat"

    #Install all packages
    - name: Ensure required packages are present
      ansible.builtin.dnf:
        name: "{{ packages }}"
        state: latest

    #Enable and start services
    - name: Ensure services are started and enabled
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop: "{{ services }}"

    #Block of config tasks
    - name: Setting up the SSL cert directory and config files
      block:
        - name: Create SSL cert directory
          ansible.builtin.file:
            path: "{{ ssl_cert_dir }}"
            state: directory

        - name: Copy config files
          ansible.builtin.copy:
            src: "{{ item['src'] }}"
            dest: "{{ item['dest'] }}"
          loop: "{{ web_config_files }}"
          notify: Restart web service

      rescue:
        - name: Configuration error message
          ansible.builtin.debug:
            msg: >
              One or more of the configuration
              changes failed, but the web service
              is still active.

    #Configure the firewall
    - name: Ensure web server ports are open
      ansible.posix.firewalld:
        service: "{{ item }}"
        immediate: true
        permanent: true
        state: enabled
      loop:
        - http
        - https

  #Add handlers
  handlers:
    - name: Restart web service
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: restarted
```

output:

```
[student@workstation control-review]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Playbook Control Lab] ****************************************************

TASK [Gathering Facts] *********************************************************
ok: [serverb.lab.example.com]

TASK [Show failed system requirements message] *********************************
skipping: [serverb.lab.example.com]

TASK [Ensure required packages are present] ************************************
changed: [serverb.lab.example.com]

TASK [Ensure services are started and enabled] *********************************
changed: [serverb.lab.example.com] => (item=httpd)
ok: [serverb.lab.example.com] => (item=firewalld)

TASK [Create SSL cert directory] ***********************************************
changed: [serverb.lab.example.com]

TASK [Copy config files] *******************************************************
changed: [serverb.lab.example.com] => (item={'src': 'server.key', 'dest': '/etc/httpd/conf.d/ssl'})
changed: [serverb.lab.example.com] => (item={'src': 'server.crt', 'dest': '/etc/httpd/conf.d/ssl'})
changed: [serverb.lab.example.com] => (item={'src': 'ssl.conf', 'dest': '/etc/httpd/conf.d'})
changed: [serverb.lab.example.com] => (item={'src': 'index.html', 'dest': '/var/www/html'})

TASK [Ensure web server ports are open] ****************************************
changed: [serverb.lab.example.com] => (item=http)
changed: [serverb.lab.example.com] => (item=https)

RUNNING HANDLER [Restart web service] ******************************************
changed: [serverb.lab.example.com]

PLAY RECAP *********************************************************************
serverb.lab.example.com    : ok=7    changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

Verify Web service

```
[student@workstation control-review]$ curl -k -vvv https://serverb.lab.example.com
*   Trying 172.25.250.11:443...
* Connected to serverb.lab.example.com (172.25.250.11) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
*  CAfile: /etc/pki/tls/certs/ca-bundle.crt
* TLSv1.0 (OUT), TLS header, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS header, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS header, Finished (20):
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.2 (OUT), TLS header, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS header, Unknown (23):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=US; L=Default City; O=Red Hat; OU=Training; CN=serverb.lab.example.com
*  start date: Nov 13 15:52:18 2018 GMT
*  expire date: Aug  9 15:52:18 2021 GMT
*  issuer: C=US; L=Default City; O=Red Hat; OU=Training; CN=serverb.lab.example.com
*  SSL certificate verify result: self-signed certificate (18), continuing anyway.
* TLSv1.2 (OUT), TLS header, Unknown (23):
> GET / HTTP/1.1
> Host: serverb.lab.example.com
> User-Agent: curl/7.76.1
> Accept: */*
> 
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.2 (IN), TLS header, Unknown (23):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* TLSv1.2 (IN), TLS header, Unknown (23):
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Fri, 30 Aug 2024 15:21:47 GMT
< Server: Apache/2.4.51 (Red Hat Enterprise Linux) OpenSSL/3.0.1
< Last-Modified: Fri, 30 Aug 2024 14:35:03 GMT
< ETag: "24-620e77f28279e"
< Accept-Ranges: bytes
< Content-Length: 36
< Content-Type: text/html; charset=UTF-8
< 
Configured for both HTTP and HTTPS.
* Connection #0 to host serverb.lab.example.com left intact
```



---

## Chapter 5. Deploying Files to Managed Hosts

### Modifying and Copying Files to Hosts

#### **Describing File Modules**

Most of the commonly used modules related to Linux file management are provided with `ansible-core` in the `ansible.builtin` collection. They perform tasks such as creating, copying, editing, and modifying permissions and other attributes of files. The following table provides a list of frequently used file management modules:



**Table 5.1. Commonly Used File Modules in `ansible.builtin`**

| Module name   | Module description                                           |
| :------------ | :----------------------------------------------------------- |
| `blockinfile` | Insert, update, or remove a block of multiline text surrounded by customizable marker lines. |
| `copy`        | Copy a file from the local or remote machine to a location on a managed host. Similar to the `file` module, the `copy` module can also set file attributes, including SELinux context. |
| `fetch`       | This module works like the `copy` module, but in reverse. This module is used for fetching files from remote machines to the control node and storing them in a file tree, organized by host name. |
| `file`        | Set attributes such as permissions, ownership, SELinux contexts, and time stamps of regular files, symlinks, hard links, and directories. This module can also create or remove regular files, symlinks, hard links, and directories. A number of other file-related modules support the same options to set attributes as the `file` module, including the `copy` module. |
| `lineinfile`  | Ensure that a particular line is in a file, or replace an existing line using a back-reference regular expression. This module is primarily useful when you want to change a single line in a file. |
| `stat`        | Retrieve status information for a file, similar to the Linux `stat` command. |

In addition, the `ansible.posix` collection, which is included in the default automation execution environment, provides some additional modules that are useful for file management:



**Table 5.2. Commonly Used File Modules in `ansible.posix`**

| Module name   | Module description                                           |
| :------------ | :----------------------------------------------------------- |
| `patch`       | Apply patches to files by using GNU `patch`.                 |
| `synchronize` | A wrapper around the `rsync` command to simplify common tasks. The `synchronize` module is not intended to provide access to the full power of the `rsync` command, but does make the most common invocations easier to implement. You might still need to call the `rsync` command directly via the `command` module depending on your use case. |

#### Automation Examples with Files Modules

The following examples show ways that you can use these modules to automate common file management tasks.

##### Ensuring a File Exists on Managed Hosts

Use the `ansible.builtin.file` module to touch a file on managed hosts. This works like the `touch` command, creating an empty file if it does not exist, and updating its modification time if it does exist. In this example, in addition to touching the file, Ansible ensures that the owning user, group, and permissions of the file are set to specific values.

```yaml
- name: Touch a file and set permissions
  ansible.builtin.file:
    path: /path/to/file
    owner: user1
    group: group1
    mode: 0640
    state: touch
```

Example outcome:

```
[user@host ~]$ ls -l file
-rw-r-----.  user1 group1 0 Nov 25 08:00 file
```

##### Modifying File Attributes

You can use the `ansible.builtin.file` module to ensure that a new or existing file has the correct permissions or SELinux type as well.

For example, the following file has retained the default SELinux context relative to a user's home directory, which is not the desired context.

```
[user@host ~]$ ls -Z samba_file
-rw-r--r--.  owner group unconfined_u:object_r:user_home_t:s0 samba_file
```

The following task ensures that the SELinux context type attribute of the `samba_file` file is the desired `samba_share_t` type. This behavior is similar to the Linux `chcon` command.

```yaml
- name: SELinux type is set to samba_share_t
  ansible.builtin.file:
    path: /path/to/samba_file
    setype: samba_share_t
```

Example outcome:

```
[user@host ~]$ ls -Z samba_file
-rw-r--r--.  owner group unconfined_u:object_r:samba_share_t:s0 samba_file
```

File attribute parameters are available in multiple file management modules. Use the `ansible-navigator doc` command for additional information, providing the `ansible.builtin.file` or `ansible.builtin.copy` module as an argument.

**Note**

To set SELinux file contexts persistently in the policy, some options include:

- If you know how to use Ansible roles, you can use the supported `redhat.rhel_system_roles.selinux` role. That is covered in Chapter 7 of the *Red Hat Enterprise Linux Automation with Ansible* (RH294) training course.
- You can use the module `community.general.sefcontext` in the community-supported `community.general` Ansible Content Collection.

##### Copying and Editing Files on Managed Hosts

In this example, the `ansible.builtin.copy` module is used to copy a file located in the Ansible working directory on the control node to selected managed hosts.

By default, this module assumes that `force: true` is set. That forces the module to overwrite the remote file if it exists but has different contents to the file being copied. If `force: false` is set, then it only copies the file to the managed host if it does not already exist.

```yaml
- name: Copy a file to managed hosts
  ansible.builtin.copy:
    src: file
    dest: /path/to/file
```

To retrieve files from managed hosts use the `ansible.builtin.fetch` module. This could be used to retrieve a file such as an SSH public key from a reference system before distributing it to other managed hosts.

```yaml
- name: Retrieve SSH key from reference host
  ansible.builtin.fetch:
    src: "/home/{{ user }}/.ssh/id_rsa.pub
    dest: "files/keys/{{ user }}.pub"
```

To ensure a specific single line of text exists in an existing file, use the `lineinfile` module:

```yaml
- name: Add a line of text to a file
  ansible.builtin.lineinfile:
    path: /path/to/file
    line: 'Add this line to the file'
    state: present
```

To add a block of text to an existing file, use the `ansible.builtin.blockinfile` module:

```yaml
- name: Add additional lines to a file
  ansible.builtin.blockinfile:
    path: /path/to/file
    block: |
      First line in the additional block of text
      Second line in the additional block of text
    state: present
```

**Note**

When using the `ansible.builtin.blockinfile` module, commented block markers are inserted at the beginning and end of the block to ensure idempotency.

```
# BEGIN ANSIBLE MANAGED BLOCK
First line in the additional block of text
Second line in the additional block of text
# END ANSIBLE MANAGED BLOCK
```

You can use the `marker` parameter to the module to help ensure that the right comment character or text is being used for the file in question.



##### Removing a File from Managed Hosts

A basic example to remove a file from managed hosts is to use the `ansible.builtin.file` module with the `state: absent` parameter. The `state` parameter is optional to many modules. You should always make your intentions clear whether you want `state: present` or `state: absent` for several reasons. Some modules support other options as well. It is possible that the default could change at some point, but perhaps most importantly, it makes it easier to understand the state the system should be in based on your task.

```yaml
- name: Make sure a file does not exist on managed hosts
  ansible.builtin.file:
    dest: /path/to/file
    state: absent
```

##### Retrieving the Status of a File on Managed Hosts

The `ansible.builtin.stat` module retrieves facts for a file, similar to the Linux `stat` command. Parameters provide the functionality to retrieve file attributes, determine the checksum of a file, and more.

The `ansible.builtin.stat` module returns a dictionary of values containing the file status data, which allows you to refer to individual pieces of information using separate variables.

The following example registers the results of a `ansible.builtin.stat` module task and then prints the MD5 checksum of the file that it checked. (The more modern SHA256 algorithm is also available; MD5 is being used here for legibility.)

```yaml
- name: Verify the checksum of a file
  ansible.builtin.stat:
    path: /path/to/file
    checksum_algorithm: md5
  register: result

- ansible.builtin.debug
    msg: "The checksum of the file is {{ result.stat.checksum }}"
```

The outcome should be similar to the following:

```
TASK [Get md5 checksum of a file] *****************************************
ok: [hostname]

TASK [debug] **************************************************************
ok: [hostname] => {
    "msg": "The checksum of the file is 5f76590425303022e933c43a7f2092a3"
}
```

Information about the values returned by the `ansible.builtin.stat` module are documented in `ansible-navigator doc ansible.builtin.stat`, or you can register a variable and display its contents to see what is available:

```yaml
- name: Examine all stat output of /etc/passwd
  hosts: workstation

  tasks:
    - name: stat /etc/passwd
      ansible.builtin.stat:
        path: /etc/passwd
      register: results

    - name: Display stat results
      debug:
        var: results
```

##### Synchronizing Files Between the Control Node and Managed Hosts

The `ansible.posix.synchronize` module is a wrapper around the `rsync` tool, which simplifies common file management tasks in your playbooks. The `rsync` tool must be installed on both the local and remote host. By default, when using the `ansible.posix.synchronize` module, the "local host" is the host that the `ansible.posix.synchronize` task originates on (usually the control node), and the "destination host" is the host that `ansible.posix.synchronize` connects to.

The following example synchronizes a file located in the Ansible working directory to the managed hosts:

```yaml
- name: synchronize local file to remote files
  ansible.posix.synchronize:
    src: file
    dest: /path/to/file
```

You can use the `ansible.posix.synchronize` module and its many parameters in many different ways, including synchronizing directories. Run the `ansible-navigator doc ansible.posix.synchronize` command for additional parameters and playbook examples.

##### References

`chmod`(1), `chown`(1), `rsync`(1), `stat`(1) and `touch`(1) man pages

`ansible-navigator doc` command

[Ansible documentation — Index of all Modules - ansible.builtin](https://docs.ansible.com/ansible/latest/collections/index_module.html#ansible-builtin)



##### Example

```yaml
---
- name: File management test
  hosts: servers
  tasks:
    - name: Retrieve secure logs
      ansible.builtin.fetch:  # get files from remote host
        src: /var/log/secure
        dest: secure-backups

    - name: Create files dir and set SE linux context
      ansible.builtin.file:
        path: /home/devops/files
        state: directory   # create the directory 'files'
        owner: devops
        group: devops
        mode: 0755
     # setype: samba_share_t   # change the SE context
        setype: _default

    - name: adding message to text
      ansible.builtin.lineinfile:
        path: /home/devops/files/users.txt
        line: "This line was addes by the lineinfile module"
        state: present
        create: true   # create the file if not exists
        owner: devops
        group: devops
        mode: 0664 

    - name: Copying files from here to remote
      ansible.builtin.copy:
        src: system
        dest: /home/devops/files/
        mode: 0664
        owner: devops
        group: devops

    - name: Adding message with blockinfile
      ansible.builtin.blockinfile:
        path: /home/devops/files/users.txt
        block: |
          this block of text consists of two lines.
          They have been added by the blockinfile module.

```

output:

```
[student@workstation secure-backups]$ ssh devops@servera 'ls -Z'
unconfined_u:object_r:samba_share_t:s0 files


[student@workstation ~]$ [student@workstation file-manage]$ tree -F secure-backups/
secure-backups/
├── servera.lab.example.com/
│   └── var/
│       └── log/
│           └── secure
└── serverb.lab.example.com/
    └── var/
        └── log/
            └── secure


[student@workstation file-manage]$ ssh devops@servera 'cat files/users.txt'
This line was addes by the lineinfile module
# BEGIN ANSIBLE MANAGED BLOCK
this block of text consists of two lines.
They have been added by the blockinfile module.
# END ANSIBLE MANAGED BLOCK
```





### Deploying Custom Files with Jinja2 Templates

#### Templating Files

The `ansible.builtin` Ansible Content Collection provides a number of modules that can be used to modify existing files. These include `lineinfile` and `blockinfile`, among others. However, they are not always easy to use effectively and correctly.

A much more powerful way to manage files is to *template* them. With this method, you can write a template configuration file that is automatically customized for the managed host when the file is deployed, using Ansible variables and facts. This can be easier to control and is less error-prone.



#### Introduction to Jinja2

Ansible uses the Jinja2 templating system for template files. Ansible also uses Jinja2 syntax to reference variables in playbooks, so you already know a little about how to use it.

##### Using Delimiters

Variables and logic expressions are placed between tags, or *delimiters*. When a Jinja2 template is evaluated, the expression `{{ *`EXPR`* }}` is replaced with the results of that expression or variable. Jinja2 templates can also use `{% *`EXPR`* %}` for special control structures or logic that loops over Jinja2 code or perform tests. You can use the `{# *`COMMENT`* #}` syntax to enclose comments that should not appear in the final file.

In the following example of a Jinja2 template file, the first line includes a comment that is not included in the final file. The variable references in the second line are replaced with the values of the system facts being referenced.

```jinja2
{# /etc/hosts line #}
{{ ansible_facts['default_ipv4']['address'] }}    {{ ansible_facts['hostname'] }}
```



#### Building a Jinja2 Template

A Jinja2 template is composed of multiple elements: data, variables, and expressions. Those variables and expressions are replaced with their values when the Jinja2 template is rendered. The variables used in the template can be specified in the `vars` section of the playbook. It is possible to use the managed hosts' facts as variables in a template.

Template files are most commonly kept in the `templates` directory of the project for your playbook, and typically are assigned a `.j2` file extension to make it clear that they are Jinja2 template files.

**Note**:

A file containing a Jinja2 template does not need to have any specific file extension (for example, `.j2`). However, providing such a file extension might make it easier for you to remember that it is a template file.

The following example shows how to create a template for `/etc/ssh/sshd_config` with variables and facts retrieved by Ansible from managed hosts. When the template is deployed by a play, any facts are replaced by their values for the managed host being configured.

```jinja2
# {{ ansible_managed }}
# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE BECAUSE THEY WILL BE LOST

Port {{ ssh_port }}
ListenAddress {{ ansible_facts['default_ipv4']['address'] }}

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

SyslogFacility AUTHPRIV

PermitRootLogin {{ root_allowed }}
AllowGroups {{ groups_allowed }}

AuthorizedKeysFile /etc/.rht_authorized_keys .ssh/authorized_keys

PasswordAuthentication {{ passwords_allowed }}

ChallengeResponseAuthentication no

GSSAPIAuthentication yes
GSSAPICleanupCredentials no

UsePAM yes

X11Forwarding yes
UsePrivilegeSeparation sandbox

AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS

Subsystem sftp	/usr/libexec/openssh/sftp-server
```



#### Deploying Jinja2 Templates

Jinja2 templates are a powerful tool that you can use to customize configuration files to be deployed on managed hosts. When the Jinja2 template for a configuration file has been created, it can be deployed to managed hosts by using the `ansible.builtin.template` module, which supports the transfer of a local file on the control node to the managed hosts.

To use the `ansible.builtin.template` module, use the following syntax. The value associated with the `src` key specifies the source Jinja2 template, and the value associated with the `dest` key specifies the file to be created on the destination hosts.

```yaml
tasks:
  - name: template render
    ansible.builtin.template:
      src: /tmp/j2-template.j2
      dest: /tmp/dest-config-file.txt
```

Note:

The `ansible.builtin.template` module also allows you to specify the owner (the user that owns the file), group, permissions, and SELinux context of the deployed file, just like the `ansible.builtin.file` module. It can also take a `validate` option to run an arbitrary command (such as `visudo` `-c`) to check the syntax of a file for correctness before templating it into place.

For more details, see `ansible-navigator doc ansible.builtin.template`.



#### Managing Templated Files

To avoid having other system administrators modify files that are managed by Ansible, it is a good practice to include a comment at the top of the template to indicate that the file should not be manually edited.

One way to do this is to use the "Ansible managed" string set by the `ansible_managed` directive. This is not a normal variable but can be used as one in a template. You can set the value for `ansible_managed` in an `ansible.cfg` file:

```
ansible_managed = Ansible managed
```

To include the `ansible_managed` string inside a Jinja2 template, use the following syntax:

```jinja2
{{ ansible_managed }}
```



#### Control Structures

You can use Jinja2 control structures in template files to reduce repetitive typing, to enter entries for each host in a play dynamically, or conditionally insert text into a file.

##### Using Loops

Jinja2 uses the `for` statement to provide looping functionality. In the following example, the `users` variable has a list of values. The `user` variable is replaced with all the values in the `users` variable, one value per line.

```jinja2
{% for user in users %}
      {{ user }}
{% endfor %}
```

The following example template uses a `for` statement and a conditional to run through all the values in the `users` variable, replacing `myuser` with each value, unless the value is `root`.

```jinja2
{# for statement #}
{% for myuser in users if not myuser == "root" %}
User number {{ loop.index }} - {{ myuser }}
{% endfor %}
```

The `loop.index` variable expands to the index number that the loop is currently on. It has a value of 1 the first time the loop executes, and it increments by 1 through each iteration.

As another example, this template also uses a `for` statement. It assumes a `myhosts` variable that contains a list of hosts to be managed has been defined by the inventory being used. If you put the following `for` statement in a Jinja2 template, all hosts in the `myhosts` group from the inventory would be listed in the resulting file.

```jinja2
{% for myhost in groups['myhosts'] %}
{{ myhost }}
{% endfor %}
```

For a more practical example, you can use this example to generate an `/etc/hosts` file from host facts dynamically. Assume that you have the following playbook:

```yaml
- name: /etc/hosts is up to date
  hosts: all
  gather_facts: true
  tasks:
    - name: Deploy /etc/hosts
      ansible.builtin.template:
        src: templates/hosts.j2
        dest: /etc/hosts
```

The following three-line `templates/hosts.j2` template constructs the file from all hosts in the group `all`. (The middle line is extremely long in the template due to the length of the variable names.) It iterates over each host in the group to get three facts for the `/etc/hosts` file.

```jinja2
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```

##### Using Conditionals

Jinja2 uses the `if` statement to provide conditional control. This allows you to put a line in a deployed file if certain conditions are met.

In the following example, the value of the `result` variable is placed in the deployed file only if the value of the `finished` variable is `True`.

```jinja2
{% if finished %}
{{ result }}
{% endif %}
```



#### Variable Filters

Jinja2 provides filters which change the output format for template expressions, essentially converting the data in a variable to some other format in the file that results from the template.

For example, filters are available for languages such as YAML and JSON. The `to_json` filter formats the expression output using JSON, and the `to_yaml` filter formats the expression output using YAML.

```jinja2
{{ output | to_json }}
{{ output | to_yaml }}
```

Additional filters are available, such as the `to_nice_json` and `to_nice_yaml` filters, which format the expression output in either JSON or YAML human-readable format.

```jinja2
{{ output | to_nice_json }}
{{ output | to_nice_yaml }}
```

Both the `from_json` and `from_yaml` filters expect strings in either JSON or YAML format, respectively.

```jinja2
{{ output | from_json }}
{{ output | from_yaml }}
```

For more information you can also review ["Using filters to manipulate data"](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html) in the *Ansible User Guide*.



#### References

[ansible.builtin.template module - Template a file out to a target host — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)

[Using filters to manipulate data — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)



#### Example

```jinja2
# Creating a motd.j2 template file
This is the system {{ ansible_facts['fqdn'] }}

This is a {{ ansible_facts['distribution'] }} version {{ ansible_facts['distribution_version'] }}

{% if ansible_facts['fqdn'] in groups['workstations'] %}
As a workstation user, you need to submit a ticket to receive help with any issues.
{% elif ansible_facts['fqdn'] in groups['webservers'] %}
Please report issues to: {{ system_owner }}.
{% endif  %}
```

```yaml
---
- name: Managing Files temaplate test
  hosts: all
  vars: 
    - system_owner: peanutfish@abc.com
  tasks:
    - name: moving motd files
      ansible.builtin.template:
        src: motd.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644
```

output:

```
[student@workstation file-template]$ ssh devops@servera 'cat /etc/motd'
This is the system servera.lab.example.com

This is a RedHat version 9.0

Please report issues to: peanutfish@abc.com.



[student@workstation file-template]$ ssh devops@workstation 'cat /etc/motd'
This is the system workstation.lab.example.com

This is a RedHat version 9.0

As a workstation user, you need to submit a ticket to receive help with any issues.
```



### Chapter 5 TEST

In the `/home/student/file-review` directory, create a playbook file called `motd.yml` that contains a new play that runs on all hosts in the inventory. It must log in using the `devops` user on the remote host, and use `become` to enable privilege escalation for the whole play.

The play must have a task that uses the `ansible.builtin.template` module to deploy the `motd.j2` Jinja2 template file to the file `/etc/motd` on the managed hosts. The resulting file must have the `root` user as its owning user and group, and its permissions must be `0644`.

Add an additional task that uses the `ansible.builtin.stat` module to verify that `/etc/motd` exists on the managed hosts and registers its results in a variable. That task must be followed by a task that uses `ansible.builtin.debug` to display the information in that registered variable.

Add a task that uses the `ansible.builtin.copy` module to place `files/issue` into the `/etc/` directory on the managed host, use the same ownership and permissions as `/etc/motd`.

**Finally, add a task that uses the `ansible.builtin.file` module to ensure that `/etc/issue.net` is a symbolic link to `/etc/issue` on the managed host.**

```jinja2
# Create a template file here, folder ./templates/motd.j2

Welcome to the END of THE DAY!

This system {{ansible_facts['fqdn']}} provided {{ ansible_facts['memtotal_mb'] }} mb memory and {{ ansible_facts['processor_count'] }} CPU core.
```

```yaml
---
- name: Starting file review session 
  hosts: all
  remote_user: devops
  become: true
  become_user: root

  tasks:
    - name: Check system total memory and number of processors
      ansible.builtin.debug:
        msg: >
          The amount of system momory is {{ ansible_facts['memtotal_mb'] }}mb and
          the number of processors is {{ ansible_facts['processor_count'] }}.
        # var: ansible_facts
    - name: Copying motd file
      ansible.builtin.template:
        src: templates/motd.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644

    - name: Check if motd exists
      ansible.builtin.stat:
        path: /etc/motd
      register: result

    - name: show debug info
      ansible.builtin.debug:
        var: result.stat

    - name: Copying files
      ansible.builtin.copy:
        src: files/issue
        dest: /etc/
        owner: root
        group: root
        mode: 0644

    - name: Verify the symbolic link
      ansible.builtin.file:
        src: /etc/issue
        dest: /etc/issue.net
        state: link
        owner: root
        group: root
        force: true    # Force the creation of the symlinks in two cases: the source file does not exist (but will appear later); the destination exists and is a file (so, we need to unlink the path file and create a symlink to the src file in place of it).
```





---

## Chapter 6. Managing Complex Plays and Playbooks

### Selecting Hosts with Host Patterns

In a play, the `hosts` directive specifies the managed hosts to run the play against.



The following example inventory is used throughout this section to illustrate host patterns.

```
[student@controlnode ~]$ cat myinventory
web.example.com
data.example.com

[lab]
labhost1.example.com
labhost2.example.com

[test]
test1.example.com
test2.example.com

[datacenter1]
labhost1.example.com
test1.example.com

[datacenter2]
labhost2.example.com
test2.example.com

[datacenter:children]
datacenter1
datacenter2

[new]
192.168.2.1
192.168.2.2
```

To demonstrate how host patterns are resolved, the following examples run `playbook.yml` Ansible Playbook, which contains a play that is edited to have different host patterns to target different subsets of managed hosts from the preceding example inventory.

#### Managed Hosts using IP addr

The most basic host pattern is the name of a single managed host listed in the inventory. 

When the playbook runs, the first `Gathering Facts` task should run on all managed hosts that match the host pattern. **A failure during this task causes the managed host to be removed from the play.**

You can only use an IP address in a host pattern if it is explicitly listed in the inventory. If the IP address is not listed in the inventory, then you cannot use it to specify the host even if the IP address resolves to that host name in DNS.

```
[student@controlnode ~]$ cat playbook.yml
---
...output omitted...
  hosts: 192.168.2.1
...output omitted...

[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [192.168.2.1]
...output omitted...
```

**Note**:

You can point an alias at a particular IP address in your inventory by setting the `ansible_host` host variable. For example, you could have a host in your inventory named `host.example` that you could use for host patterns and inventory groups, and direct connections using that name to the IP address `192.168.2.1` by creating a `host_vars/host.example` file containing the following host variable:

```yaml
ansible_host: 192.168.2.1
```



#### Specifying Hosts Using a Group

You can use the names of inventory host groups as host patterns. 

- group_name
- all
- ungrouped

```
[student@controlnode ~]$ cat playbook.yml
---
...output omitted...
  hosts: lab
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
...output omitted...
```

Remember that there is a special group named `all` that matches all managed hosts in the inventory.

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: all
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
ok: [labhost1.example.com]
ok: [192.168.2.1]
ok: [test1.example.com]
ok: [192.168.2.2]
```

There is also a special group named `ungrouped`, which includes all managed hosts in the inventory that are not members of any other group:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: ungrouped
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [web.example.com]
ok: [data.example.com]
```



#### Matching Multiple Hosts with Wildcards

Another method of accomplishing the same thing as the `all` host pattern is to use the asterisk (*) wildcard character, which matches any string. If the host pattern is just a quoted asterisk, then all hosts in the inventory match.

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: '*'
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
ok: [labhost1.example.com]
ok: [192.168.2.1]
ok: [test1.example.com]
ok: [192.168.2.2]
```

**Important**:

Some characters that are used in host patterns also have meaning for the shell. If you are using any special wildcards or list characters in an Ansible Playbook, then you must put your host pattern in single quotes to ensure it is parsed correctly.

```yaml
  hosts: '!test1.example.com,development'
```

The asterisk character can also be used to match any managed hosts or groups that contain a particular substring.

For example, the following wildcard host pattern matches all inventory names that end in `.example.com`:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: '*.example.com'
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
```

The following example uses a wildcard host pattern to match the names of hosts or host groups that start with `192.168.2.`:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: '192.168.2.*'
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [192.168.2.1]
ok: [192.168.2.2]
```

The next example uses a wildcard host pattern to match the names of hosts or host groups that begin with `data`.

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: 'data*'
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [data.example.com]
```

**Important**:

The wildcard host patterns match all inventory names, hosts, and host groups. They do not distinguish between names that are DNS names, IP addresses, or groups, which can lead to some unexpected matches.



#### Lists

Multiple entries in an inventory can be referenced using logical lists. A comma-separated list of host patterns matches all hosts that match any of those host patterns.

If you provide **a comma-separated list of managed hosts**, then all those managed hosts are targeted:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: labhost1.example.com,test2.example.com,192.168.2.2
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test2.example.com]
ok: [192.168.2.2]
```

If you provide **a comma-separated list of groups**, then all hosts in any of those groups are targeted:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: lab,datacenter1
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
ok: [test1.example.com]
```

You can also **mix managed hosts, host groups, and wildcards**, as shown below:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: lab,data*,192.168.2.2
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
ok: [test1.example.com]
ok: [test2.example.com]
ok: [data.example.com]
ok: [192.168.2.2]
```

**Note**:

The colon character (:) can be used instead of a comma. However, the comma is the preferred separator, especially when working with IPv6 addresses as managed host names. 

#### Lists with special char `&/!`

If an item in a list starts with an ampersand character (&), similarly to a logical AND, then hosts must match that item in order to match the host pattern. 

For example, based on our example inventory, the following host pattern matches machines in the `lab` group only if they are also in the `datacenter1` group:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: lab,&datacenter1
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
```

You could also specify that machines in the `datacenter1` group match only if they are in the `lab` group with the host patterns `&lab,datacenter1` or `datacenter1,&lab`.



You can exclude hosts that match a pattern from a list by using the exclamation point or "bang" character (!) in front of the host pattern. This operates like a logical NOT.

This example matches all hosts defined in the `datacenter` group, except `test2.example.com` based on the example inventory:

```
[student@controlnode ~]$ cat playbook.yml
...output omitted...
  hosts: datacenter,!test2.example.com
...output omitted...
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
```

The pattern `'!test2.example.com,datacenter'` could have been used in the preceding example to achieve the same result.

The pattern `hosts: all,!datacenter1`shows the use of a host pattern that matches all hosts in the test inventory, except the managed hosts in the `datacenter1` group.



#### References

[Patterns: targeting hosts and groups — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)

[How to build your inventory — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)



#### Example

inventory files:

```
[student@workstation projects-host]$ cat inventory1
srv1.example.com
srv2.example.com
s1.lab.example.com
s2.lab.example.com

[web]
jupiter.lab.example.com
saturn.example.com

[db]
db1.example.com
db2.example.com
db3.example.com

[lb]
lb1.lab.example.com
lb2.lab.example.com

[boston]
db1.example.com
jupiter.lab.example.com
lb2.lab.example.com

[london]
db2.example.com
db3.example.com
file1.lab.example.com
lb1.lab.example.com

[dev]
web1.lab.example.com
db3.example.com

[stage]
file2.example.com
db2.example.com

[prod]
lb2.lab.example.com
db1.example.com
jupiter.lab.example.com

[function:children]
web
db
lb
city

[city:children]
boston
london
environments

[environments:children]
dev
stage
prod
new

[new]
172.25.252.23
172.25.252.44
172.25.252.32
[student@workstation projects-host]$ cat inventory2
workstation.lab.example.com

[london]
servera.lab.example.com

[berlin]
serverb.lab.example.com

[tokyo]
serverc.lab.example.com

[atlanta]
serverd.lab.example.com

[europe:children]
london
berlin
```



playbook.yml

```yaml
---
- name: Resolve host patterns
  # hosts: db1.example.com
  # hosts: 172.25.252.32
  # hosts: all
  # hosts: '*example.com'
  # hosts: "*example.com,!*.lab.example.com"
  # hosts: lb1.lab.example.com,s1.lab.example.com,db1.example.com
  # hosts: '172.25.*'
  # hosts: 's*'
  # hosts: 'prod,172*,*lab*'
  # hosts: db,&london
  # hosts: london
  # hosts: europe
  # hosts: ungrouped
  hosts: chicken  # invalid group name


  gather_facts: false
  tasks:
  - name: Display managed hosts matching the host pattern
    ansible.builtin.debug:
      msg: "{{ inventory_hostname }}"
```



### Including and Importing Files

**Purpose**: Managing Large Playbooks

**Including or Importing Files**:

Ansible supports two operations for bringing content into a playbook. You can *include* content, or you can *import* content.

When you **include** content, it is a ***dynamic*** operation. Ansible processes included content **during the run of the playbook**, as content is reached.

When you **import** content, it is a ***static*** operation. Ansible preprocesses imported content **when the playbook is initially parsed**, before the run starts.



#### Importing Playbooks

Use the `ansible.builtin.import_playbook` module to import external files containing lists of plays into a playbook. In other words, you can have a main playbook that imports one or more additional playbooks.

Because the content being imported is a complete playbook, the `ansible.builtin.import_playbook` module can only be used **at the top level** **of a playbook** and cannot be used inside a play. If you import multiple playbooks, then they are imported and run in order.

The following is a simple example of a main playbook that imports two additional playbooks:

```yaml
- name: Prepare the web server
  ansible.builtin.import_playbook: web.yml

- name: Prepare the database server
  ansible.builtin.import_playbook: db.yml
```

You can also interleave plays in your main playbook with imported playbooks.

```yaml
---
- name: Play 1
  hosts: localhost
  tasks:
    - name: Display a message
      ansible.builtin.debug:
        msg: Play 1

- name: Import Playbook
  ansible.builtin.import_playbook: play2.yml
```

In the preceding example, the `Play 1` play runs first, followed by the plays imported from the `play2.yml` playbook.



#### Importing and Including Tasks

You can import or include a list of tasks from a task file into a play. A task file is a file that contains a flat list of tasks:

```yaml
[user@host ~]$ cat webserver_tasks.yml
---
- name: Install the httpd package
  ansible.builtin.dnf:
    name: httpd
    state: latest

- name: Start the httpd service
  ansible.builtin.service:
    name: httpd
    state: started
```

##### Importing Task Files

You can statically import a task file into a play inside a playbook by using the `ansible.builtin.import_tasks` module. When you import a task file, the tasks in that file are directly inserted when the playbook is parsed. The location of the task in the playbook that uses the `ansible.builtin.import_tasks` module controls where the tasks are inserted and the order in which multiple imports are run.

```yaml
---
- name: Install web server
  hosts: webservers
  tasks:
    - name: Import webserver tasks
      ansible.builtin.import_tasks: webserver_tasks.yml
```

When you import a task file, the tasks in that file are directly inserted when the playbook is parsed. Because the `ansible.builtin.import_tasks` module statically imports the tasks when the playbook is parsed, the following items must be considered:

- When using the `ansible.builtin.import_tasks` module, conditional statements set on the import, such as `when`, are applied to each of the tasks that are imported.
- You cannot use loops with the `ansible.builtin.import_tasks` module.
- If you use a variable to specify the name of the file to import, then you cannot use a host or group inventory variable.

##### Including Task Files

You can also dynamically include a task file into a play inside a playbook by using the `ansible.builtin.include_tasks` module.

```yaml
---
- name: Install web server
  hosts: webservers
  tasks:
    - name: Include webserver tasks
      ansible.builtin.include_tasks: webserver_tasks.yml
```

The `ansible.builtin.include_tasks` module does not process content in the playbook until the play is running and that part of the play is reached. The order in which playbook content is processed impacts how the `ansible.builtin.include_tasks` module works.

- When using the `ansible.builtin.include_tasks` module, conditional statements such as `when` set on the include determine whether the tasks are included in the play at all.
- If you run `ansible-navigator run --list-tasks` to list the tasks in the playbook, then tasks in the included task files are not displayed. The tasks that include the task files are displayed. By comparison, the `ansible.builtin.import_tasks` module would not list tasks that import task files, but instead would list the individual tasks from the imported task files.
- You cannot use `ansible-navigator run --start-at-task` to start playbook execution from a task that is in an included task file.
- You cannot use a `notify` statement to trigger a handler name that is in an included task file. You can trigger a handler in the main playbook that includes an entire task file, in which case all tasks in the included file run.



##### Importing and Including with Conditionals

Conditional statements behave differently depending on whether you are importing or including tasks.

- When you add a conditional to a task that uses an `ansible.builtin.import_*` module, Ansible applies the condition to all tasks within the imported file. In other words, each task in the imported content performs that conditional check before it runs.
- When you use a conditional on a task that uses an `ansible.builtin.include_*` module, the condition is applied only to the include task itself and not to any other tasks within the included file. in other words, the conditional determines whether the include happens or not. If the include happens, then all the tasks that are included run normally.

Refer to the [Ansible User Guide](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html#applying-when-to-roles-imports-and-includes) for a more detailed discussion of the differences in behavior between the `ansible.builtin.import_tasks` module and the `ansible.builtin.include_tasks` module when conditionals are used.



##### Use Cases for Task Files

Consider the following examples where it might be useful to manage sets of tasks as external files separate from the playbook:

- If new servers require complete configuration, then administrators could create various sets of tasks for creating users, installing packages, configuring services, configuring privileges, setting up access to a shared file system, hardening the servers, installing security updates, and installing a monitoring agent. Each of these sets of tasks could be managed through a separate self-contained task file.
- If servers are managed collectively by the developers, the system administrators, and the database administrators, then every organization can write its own task file which can then be reviewed and integrated by the system manager.
- If a server requires a particular configuration, then it can be integrated as a set of tasks that are executed based on a conditional. In other words, including the tasks only if specific criteria are met.
- If a group of servers needs to run a particular task or set of tasks, then the tasks might only be run on a server if it is part of a specific host group.

##### Managing Task Files

You can create a dedicated directory for task files, and save all task files in that directory. Then your playbook can include or import task files from that directory. This allows construction of a complex playbook and makes it easy to manage its structure and components.



#### Defining Variables for External Plays and Tasks

The incorporation of plays or tasks from external files into playbooks using the Ansible import and include features enhances the ability to reuse tasks and playbooks across an Ansible environment. To maximize the possibility of reuse, these task and play files should be as generic as possible. Variables can be used to parameterize play and task elements to expand the application of tasks and plays.

If you parameterize the package and service elements as shown in the following example, then the task file can also be used for the installation and administration of other software and their services, rather than being useful for a web service only.

```yaml
---
- name: Install the {{ package }} package
  ansible.builtin.dnf:
    name: "{{ package }}"
    state: latest

- name: Start the {{ service }} service
  ansible.builtin.service:
    name: "{{ service }}"
    enabled: true
    state: started
```

Subsequently, when incorporating the task file into a playbook, define the variables to use for the task execution as follows:

```yaml
...output omitted...
  tasks:
    - name: Import task file and set variables
      ansible.builtin.import_tasks: task.yml
      vars:
        package: httpd
        service: httpd
```

Ansible makes the passed variables available to the tasks imported from the external file.

You can use the same technique to make play files more reusable. When incorporating a play file into a playbook, pass the variables to use for the play execution as follows:

```yaml
...output omitted...
- name: Import play file and set the variable
  ansible.builtin.import_playbook: play.yml
  vars:
    package: mariadb
```

**Important NOTE**:

Earlier versions of Ansible used the `ansible.builtin.include` module to include both playbooks and task files, depending on context. This functionality is being deprecated for a number of reasons.

Before Ansible 2.0, the `ansible.builtin.include` module operated like a static import. In Ansible 2.0 it was changed to operate dynamically, but this created some limitations. In Ansible 2.1 it became possible for the `ansible.builtin.include` module to be dynamic or static depending on task settings, which was confusing and error-prone. There were also issues with ensuring that the `ansible.builtin.include` module worked correctly in all contexts.

Thus, `ansible.builtin.include` was replaced in Ansible 2.4 with new directives such as `ansible.builtin.include_tasks`, `import_tasks`, and `ansible.builtin.import_playbook`. You might find examples of the `ansible.builtin.include` module in earlier playbooks, but you should avoid using it in new ones.

#### References

[Including and Importing — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_includes.html)

[Creating Reusable Playbooks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html)

[Conditionals — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)



### Chapter 6 Example

```
[student@workstation projects-file]$ tree -F
.
├── ansible.cfg
├── ansible-navigator.log
├── inventory
├── playbok.yml
├── plays/
│   └── test.yml
└── tasks/
    ├── environment.yml
    ├── firewall.yml
    └── placeholder.yml
```

Contents as below:

```yaml
[student@workstation projects-file]$ cat plays/test.yml 
---
- name: Test web service
  hosts: server*.lab.example.com
  become: false
  tasks:
    - name: Connect to internet web server
      ansible.builtin.uri:
        url: "{{ url }}"
        status_code: 200


[student@workstation tasks]$ cat environment.yml 
---
  - name: Install the {{ package }} package
    ansible.builtin.dnf:
      name: "{{ package }}"
      state: latest
  - name: Start the {{ service }} service
    ansible.builtin.service:
      name: "{{ service }}"
      enabled: true
      state: started


[student@workstation tasks]$ cat firewall.yml 
---
  - name: Install the firewall
    ansible.builtin.dnf:
      name: "{{ firewall_pkg }}"
      state: latest

  - name: Start the firewall
    ansible.builtin.service:
      state: started
      name: "{{ firewall_svc }}"
      enabled: true

  - name: Open the port for {{ rule }}
    ansible.posix.firewalld:
      service: "{{ item }}"
      immediate: true
      permanent: true
      state: enabled
    loop: "{{ rule }}"


[student@workstation tasks]$ cat placeholder.yml 
---
  - name: Create placeholder file
    ansible.builtin.copy:
      content: "{{ ansible_facts['fqdn'] }} has been customized using Ansible.\n"
      dest: "{{ file }}"
```

Main playbook:

```yaml
---
- name: Configure web server
  hosts: servera.lab.example.com

  tasks:
    - name: Installing web service
      ansible.builtin.include_tasks: tasks/environment.yml
      vars:
        package: httpd
        service: httpd
    
    - name: Configuring firewall settings
      ansible.builtin.import_tasks: tasks/firewall.yml
      vars:
        firewall_pkg: firewalld
        firewall_svc: firewalld
        rule: 
          - http
          - https

    - name: Creating placeholder firewall
      ansible.builtin.import_tasks: tasks/placeholder.yml
      vars:
        file: /var/www/html/index.html

- name: Importing test.yml playbook
  ansible.builtin.import_playbook: plays/test.yml
  vars:
    url: 'http://servera.lab.example.com'
```



---

## Chapter 7. Simplifying Playbooks with Roles and Ansible Content Collections

### Describing Role Structure

Ansible *roles* make it easier to reuse Ansible code generically. You can package all the tasks, variables, files, templates, and other resources needed to provision infrastructure or deploy applications in a standardized directory structure. Copy a role from project to project by copying the directory, then call the role within a play.

Ansible roles have the following benefits:

- Roles group content together, enabling easy sharing of code with others.
- Roles can define the essential elements of a system type, such as a web server, database server, or Git repository.
- Roles make larger projects more manageable.
- Roles can be developed in parallel by different users.

In addition to writing, using, reusing, and sharing your own roles, you can obtain roles from other sources. You can find roles by using distribution packages, such as Ansible Content Collections. Or, you can download roles from the Red Hat automation hub, a private automation hub, and from the community's Ansible Galaxy website.

Red Hat Enterprise Linux includes some roles in the `rhel-system-roles` package. You learn more about `rhel-system-roles` later in this chapter.



#### Examining the Ansible Role Structure

An Ansible role is defined by a standardized structure of subdirectories and files.

The top-level directory defines the name of the role itself. Files are organized into subdirectories that are named according to each file's purpose in the role, such as `tasks` and `handlers`.

The `files` and `templates` subdirectories contain files referenced by tasks in other playbooks and task files.

The following `tree` command displays the directory structure of the `user.example` role.

```
[user@host roles]$ tree user.example
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```



**Table 7.1. Ansible Role Subdirectories**

| Subdirectory | Function                                                     |
| :----------- | :----------------------------------------------------------- |
| `defaults`   | The `main.yml` file in this directory contains the default values of role variables that can be overwritten when the role is used. These variables have low precedence and are intended to be changed and customized in plays. |
| `files`      | This directory contains static files that are referenced by role tasks. |
| `handlers`   | The `main.yml` file in this directory contains the role's handler definitions. |
| `meta`       | The `main.yml` file in this directory contains information about the role, including author, license, platforms, and optional role dependencies. |
| `tasks`      | The `main.yml` file in this directory contains the role's task definitions. |
| `templates`  | This directory contains Jinja2 templates that are referenced by role tasks. |
| `tests`      | This directory can contain an inventory and `test.yml` playbook that can be used to test the role. |
| `vars`       | The `main.yml` file in this directory defines the role's variable values. Often these variables are used for internal purposes within the role. These variables have high precedence and are not intended to be changed when used in a playbook. |

Not every role has all of these directories.



#### Defining Variables and Defaults

***Role variables*** are defined by creating a `vars/main.yml` file with key-value pairs in the role directory hierarchy. These variables are referenced in role task files like any other variable: `{{ `VAR_NAME` }}`. These variables have a **high precedence** and **can not be overridden by inventory variables.** These variables are used by the **internal** functioning of the role.

***Default variables*** enable you to set default values for variables that can be used in a play to configure the role or customize its behavior. These variables are defined by creating a `defaults/main.yml` file with key-value pairs in the role directory hierarchy. Default variables have the lowest precedence of any available variables.

Default variable values **can be overridden** by any other variable, including inventory variables. These variables are intended to provide the person writing a play that uses the role with a way to customize or control exactly what it is going to do. You can use default variables to provide information to the role that it needs to configure or deploy something properly.

Define a specific variable in either `vars/main.yml` or `defaults/main.yml`, but not in both places. Use default variables when you intend that the variable values might be overridden.



#### Using Ansible Roles in a Play

There are several ways to call roles in a play. The two primary methods are:

- You can include or import them like a task in your `tasks` list.
- You can create a `roles` list that runs specific roles before your play's tasks.

The first method is the most flexible, but the second method is also commonly used and was invented before the first method.

##### Including and Importing Roles as Tasks

###### Import Roles

[Importing roles: static reuse](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#id8)

Roles can be added to a play by using an ordinary task. Use the `ansible.builtin.import_role` module to statically import a role, and the `ansible.builtin.include_role` module to dynamically include a role.

The following play demonstrates how you can import a role by using a task with the `ansible.builtin.import_role` module. The example play runs the task `A normal task` first, then imports the `role2` role.

```yaml
- name: Run a role as a task
  hosts: remote.example.com
  tasks:
    - name: A normal task
      ansible.builtin.debug:
        msg: 'first task'
    - name: A task to import role2 here
      ansible.builtin.import_role:
        name: role2
```

With the `ansible.builtin.import_role` module, Ansible treats the role as a static import and parses it during initial playbook processing.

In the preceding example, when the playbook is parsed:

- If `roles/role2/tasks/main.yml` exists, Ansible adds the tasks in that file to the play.
- If `roles/role2/handlers/main.yml` exists, Ansible adds the handlers in that file to the play.
- If `roles/role2/defaults/main.yml` exists, Ansible adds the default variables in that file to the play.
- If `roles/role2/vars/main.yml` exists, Ansible adds the variables in that file to the play (possibly overriding values from role default variables due to precedence).

**Important**

Because `ansible.builtin.import_role` is processed when the playbook is parsed, the role's handlers, default variables, and role variables are all exposed to all the tasks and roles in the play, and can be accessed by tasks and roles that precede it in the play (even though the role has not run yet).

You can also set variables for the role when you call the task, in the same way that you can set task variables:

```yaml
- name: Run a role as a task
  hosts: remote.example.com
  tasks:
    - name: A task to include role2 here
      ansible.builtin.import_role:
        name: role2
      vars:
        var1: val1
        var2: val2
```



###### Include Roles

 [Including roles: dynamic reuse](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#id7)

The `ansible.builtin.include_role` module works in a similar way, but it dynamically includes the role when the playbook is running instead of statically importing it when the playbook is initially parsed.

```yaml
---
- hosts: webservers
  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs before the example role"

    - name: Include the example role
      include_role:
        name: example

    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs after the example role"
```



One key difference between the two modules is how they handle task-level keywords, conditionals, and loops:

- `ansible.builtin.import_role` applies the task's conditionals and loops to each of the tasks being imported.
- `ansible.builtin.include_role` applies the task's conditionals and loops to the statement that determines whether the role is included or not.

In addition, when you include a role, its role variables and default variables are *not* exposed to the rest of the play, unlike `ansible.builtin.import_role`.

```yaml
---
- hosts: webservers
  tasks:
    - name: Include the some_role role
      include_role:
        name: some_role
      when: "ansible_facts['os_family'] == 'RedHat'"
```



##### Using a Roles Section in a Play

Another way you can call roles in a play is to list them in a `roles` section. The `roles` section is very similar to the `tasks` section, except instead of consisting of a list of tasks, it consists of a list of roles.

In the following example play, the `role1` role runs, then the `role2` role runs.

```yaml
---
- name: A play that only has roles
  hosts: remote.example.com
  roles:
    - role: role1
    - role: role2
```

For each role specified, the role's tasks, handlers, variables, and dependencies are imported into the play in the order in which they are listed.

When you use a `roles` section to import roles into a play, the roles run first, before any tasks that you define for that play. Whether the `roles` section is listed before or after the `tasks` section in the play does not matter.

```yaml
---
- name: Roles run before tasks
  hosts: remote.example.com
  tasks:
    - name: A task
      ansible.builtin.debug:
        msg: "This task runs after the role."
  roles:
    - role: role1
```

Because roles run first, it generally makes sense to list the `roles` section before the `tasks` section, if you must have both. The preceding play can be rewritten as follows without changing how it runs:

```yaml
---
- name: Roles run before tasks
  hosts: remote.example.com
  roles:
    - role: role1
  tasks:
    - name: A task.
      ansible.builtin.debug:
        msg: "This task runs after the role."
```

**Important**

A `tasks` section in a play is not required. In fact, it is generally a good practice to avoid both `roles` and `tasks` sections in a play to avoid confusion about the order in which roles and tasks run.

If you must have a `tasks` section and roles, it is better to create tasks that use `ansible.builtin.import_role` and `ansible.builtin.include_role` to run at the correct points in the play's execution.

The following example sets values for two role variables of `role2`, `var1` and `var2`. Any `defaults` and `vars` variables are overridden when `role2` is used.

```yaml
---
- name: A play that runs the second role with variables
  hosts: remote.example.com
  roles:
    - role: role1
    - role: role2
      var1: val1
      var2: val2
```

Another equivalent playbook syntax that you might see in this case is:

```yaml
---
- name: A play that runs the second role with variables
  hosts: remote.example.com
  roles:
    - role: role1
    - { role: role2, var1: val1, var2: val2 }
```

There are situations in which this can be harder to read, even though it is more compact.

**Important**

Ansible looks for duplicate role lines in the `roles` section. If two roles are listed with exactly the same parameters, the role only runs once.

For example, the following `roles` section only runs `role1` one time:

```yaml
roles:
  - { role: role1, service: "httpd" }
  - { role: role2, var1: true }
  - { role: role1, service: "httpd" }
```

To run the same role a second time, it must have different parameters defined:

```yaml
roles:
  - { role: role1, service: "httpd" }
  - { role: role2, var1: true }
  - { role: role1, service: "postfix" }
```



##### Special Tasks Sections

There are two special task sections, `pre_tasks` and `post_tasks`, that are occasionally used with `roles` sections. The `pre_tasks` section is a list of tasks, similar to `tasks`, but these tasks run before any of the roles in the `roles` section. If any task in the `pre-tasks` section notify a handler, then those handler tasks run before the roles or normal tasks.

Plays also support a `post_tasks` keyword. These tasks run after the play's `tasks` and any handlers notified by the play's `tasks`.

The following play shows an example with `pre_tasks`, `roles`, `tasks`, `post_tasks` and `handlers`. It is unusual that a play would contain all of these sections.

```yaml
- name: Play to illustrate order of execution
  hosts: remote.example.com
  pre_tasks:
    - name: This task runs first
      ansible.builtin.debug:
        msg: This task is in pre_tasks
      notify: my handler
      changed_when: true
  roles:
    - role: role1
  tasks:
    - name: This task runs after the roles
      ansible.builtin.debug:
        msg: This task is in tasks
      notify: my handler
      changed_when: true
  post_tasks:
    - name: This task runs last
      ansible.builtin.debug:
        msg: This task is in post_tasks
      notify: my handler
      changed_when: true
  handlers:
    - name: my handler
      ansible.builtin.debug:
        msg: Running my handler
```

In the preceding example, an `ansible.builtin.debug` task runs in each `tasks` section and in the role in the `roles` section. Each of those tasks notifies the `my handler` handler, which means the `my handler` task runs three times:

- After all the `pre_tasks` tasks run
- After all the `roles` tasks and `tasks` tasks run
- After all the `post_tasks` run

**Note**

In general, if you think you need `pre_tasks` and `post_tasks` sections in your play because you are using `roles`, consider importing the roles as tasks and including only a `tasks` section. Alternatively, it might be simpler to have multiple plays in your playbook.

#### References

[Roles — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)



### Creating Roles

#### The Role Creation Process

Creating roles in Ansible does not require any special development tools.

Creating and using a role is a three-step process:

1. Create the role directory structure.
2. Define the role content.
3. Use the role in a playbook.

#### Creating the Role Directory Structure

Ansible looks for roles in a subdirectory called `roles` in the directory containing your Ansible Playbook. Each role has its own directory with a standardized directory structure. This structure allows you to store roles with the playbook and other supporting files.

For example, the following directory structure contains the files that define the `motd` role.

```
[user@host ~]$ tree roles/
roles/
└── motd
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    └── templates
        └── motd.j2
```

The `README.md` provides a basic human-readable description of the role, documentation, examples of how to use it, and any non-Ansible role requirements. The `meta` subdirectory contains a `main.yml` file that specifies information about the author, license, compatibility, and dependencies for the module.

The `files` subdirectory contains fixed content files and the `templates` subdirectory contains templates that the role can deploy.

The other subdirectories can contain `main.yml` files that define default variable values, handlers, tasks, role metadata, or variables, depending on their subdirectory.

If a subdirectory exists but is empty, such as `handlers` in this example, it is ignored. You can **omit** the subdirectory altogether if the role does not use a feature. This example omits the `vars` subdirectory.



**Run `ansible-galaxy role init` to create the directory structure for a new role.** 

Specify the role's name as an argument to the command, which creates a subdirectory for the new role in the current working directory.

```
[user@host playbook-project]$ cd roles
[user@host roles]$ ansible-galaxy role init my_new_role
- Role my_new_role was created successfully
[user@host roles]$ ls my_new_role/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

#### Defining the Role Content

After creating the directory structure, you must write the content of the role. A good place to start is the `*`ROLENAME`*/tasks/main.yml` task file, the main list of tasks that the role runs.

The following `tasks/main.yml` file manages the `/etc/motd` file on managed hosts. It uses the `template` module to deploy the template named `motd.j2` to the managed host. Because the `template` module is configured within a role task, instead of a playbook task, the `motd.j2` template is retrieved from the role's `templates` subdirectory.

```yaml
[user@host ~]$ cat roles/motd/tasks/main.yml
---
# tasks file for motd

- name: deliver motd file
  ansible.builtin.template:
    src: motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
```

The following command displays the contents of the `motd.j2` template of the `motd` role. It references Ansible facts and a `system_owner` variable.

```jinja2
[user@host ~]$ cat roles/motd/templates/motd.j2
This is the system {{ ansible_facts['hostname'] }}.

Today's date is: {{ ansible_facts['date_time']['date'] }}.

Only use this system with permission.
You can ask {{ system_owner }} for access.
```

The role defines a default value for the `system_owner` variable. The `defaults/main.yml` file in the role's directory structure is where this value is set.

The following `defaults/main.yml` file sets the `system_owner` variable to `user@host.example.com`. This email address is written in the `/etc/motd` file of managed hosts when this role is applied.

```yaml
[user@host ~]$ cat roles/motd/defaults/main.yml
---
system_owner: user@host.example.com
```



#### Recommended Practices for Role Content Development

Roles allow you to break down playbooks into multiple files, resulting in reusable code. To maximize the effectiveness of newly developed roles, consider implementing the following recommended practices into your role development:

- Maintain each role in its own version control repository. Ansible works well with Git-based repositories.
- Use variables to configure roles so that you can reuse the role to perform similar tasks in similar circumstances.
- Avoid storing sensitive information in a role, such as passwords or SSH keys. Configure role variables that are used to contain sensitive values when called in a play with default values that are not sensitive. Playbooks that use the role are responsible for defining sensitive variables through Ansible Vault variable files or other methods.
- Use the `ansible-galaxy role init` command to start your role, and then remove any unnecessary files and directories.
- Create and maintain `README.md` and `meta/main.yml` files to document the role's purpose, author, and usage.
- Keep your role focused on a specific purpose or function. Instead of making one role do many things, write more than one role.
- Reuse roles often.

Resist creating new roles for edge configurations. If an existing role accomplishes most of the required configuration, refactor the existing role to integrate the new configuration scenario.

Use integration and regression testing techniques to ensure that the role provides the required new functionality and does not cause problems for existing playbooks.



A longer unofficial list of good practices to follow when you write a role is available from `https://redhat-cop.github.io/automation-good-practices/#_roles_good_practices_for_ansible`.



#### Changing a Role's Behavior with Variables

A well-written role uses default variables to alter the role's behavior to match a related configuration scenario. Roles that use variables are more generic and reusable in a variety of contexts.

The value of any variable defined in a role's `defaults` directory is overwritten if that same variable is defined:

- In an inventory file, either as a host variable or a group variable.
- In a YAML file under the `group_vars` or `host_vars` directories of a playbook project.
- As a variable nested in the `vars` keyword of a play.
- As a variable when including the role in `roles` keyword of a play.

The following example shows how to use the `motd` role with a different value for the `system_owner` role variable. The value specified, `someone@host.example.com`, replaces the variable reference when the role is applied to a managed host.

```yaml
[user@host ~]$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  remote_user: devops
  become: true
  vars:
    system_owner: someone@host.example.com
  roles:
    - role: motd
```

When defined in this way, the `system_owner` variable replaces the value of the default variable of the same name. **Any variable definitions nested within the `vars` keyword do NOT replace the value of the same variable if defined in a role's `vars` directory.**

The following example also shows how to use the `motd` role with a different value for the `system_owner` role variable. The value specified, `someone@host.example.com`, replaces the variable reference regardless of being defined in the role's `vars` or `defaults` directory.

```yaml
[user@host ~]$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  remote_user: devops
  become: true
  roles:
    - role: motd
      system_owner: someone@host.example.com
```

**Important NOTE**

Variable precedence can be confusing when working with role variables in a play.

- Most other variables override a role's default variables: inventory variables, play `vars`, inline *role parameters*, and so on.
- Fewer variables can override variables defined in a role's `vars` directory. **Facts, variables loaded with `include_vars`, registered variables, and role parameters can override these variables**. **Inventory variables and play `vars` cannot.** This behavior is important because it helps keep your play from accidentally changing the internal functioning of the role.
- Variables declared inline as role parameters have **very high precedence**; they can also override variables defined in a role's `vars` directory.

If a role parameter has the same name as a variable set in play `vars`, a role's `vars`, or an inventory or playbook variable, the role parameter overrides the other variable.



#### Defining Role Dependencies

Role dependencies allow a role to include other roles as dependencies.

For example, a role that defines a documentation server might depend upon another role that installs and configures a web server.

Dependencies are defined in the `meta/main.yml` file in the role directory hierarchy.

The following is a sample `meta/main.yml` file.

```yaml
---
dependencies:
  - role: apache
    port: 8080
  - role: postgres
    dbname: serverlist
    admin_user: felix
```



A `meta/main.yml` file might also have a top-level `galaxy_info` key that has a dictionary of other attributes that specify the author, purpose, license, and the versions of Ansible Core and operating systems that the role supports.

**By default, if multiple roles have a dependency on a role, and that role is called by different roles in the play multiple times with the same attributes, then the role only runs *the first time* it appears.** This behavior can be overridden by setting the `allow_duplicates` variable to `yes` in your role's `meta/main.yml` file.

Important:

Limit your role's dependencies on other roles. Dependencies make it harder to maintain your role, especially if it has many complex dependencies.

#### References

[Using Roles — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles)

[Using Variables — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)

[Roles Good Practices for Ansible](https://redhat-cop.github.io/automation-good-practices/#_roles_good_practices_for_ansible)



#### Example

```yaml
[student@workstation role-create]$ mkdir -v roles; cd roles
mkdir: created directory 'roles'

# create role 'myvhost'
[student@workstation roles]$ ansible-galaxy role init myvhost
- Role myvhost was created successfully

# delete unused 3 folders
[student@workstation roles]$ rm -rvf myvhost/{defaults,vars,tests}

[student@workstation role-create]$ tree
.
├── ansible.cfg
├── inventory
├── roles
│   └── myvhost
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       └── templates
├── verify-config.yml
├── verify-content.yml
├── verify-httpd.yml
└── vhost.conf.j2

# file contents
[student@workstation role-create]$ cat ansible.cfg 
[defaults]
inventory=inventory
remote_user=devops

#Try me...
#callback_whitelist=timer

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False


[student@workstation role-create]$ cat inventory 
workstation

[webservers]
servera.lab.example.com


[student@workstation role-create]$ cat verify-config.yml 
---
- name: Verify the httpd config
  hosts: servera.lab.example.com

  tasks:

  - name: Verify the httpd config file is in place
    ansible.builtin.command: cat /etc/httpd/conf.d/vhost.conf
    register: config

  - name: What does the httpd config file contain
    ansible.builtin.debug:
      msg: '{{ config }}.stdout_lines'


[student@workstation role-create]$ cat verify-content.yml 
---
- name: Verify the index.html file
  hosts: servera.lab.example.com

  tasks:

  - name: Verify the index.html file is in place
    ansible.builtin.command: cat /var/www/vhosts/servera/index.html
    register: content

  - name: What does the index.html config file contain
    ansible.builtin.debug:
      msg: '{{ content }}.stdout_lines'


[student@workstation role-create]$ cat verify-httpd.yml 
---
- name: Verify the httpd service
  hosts: servera.lab.example.com

  tasks:

  - name: Verify the httpd service is installed
    ansible.builtin.command: rpm -q httpd
    register: installed

  - name: Is the httpd service installed
    ansible.builtin.debug:
      msg: '{{ installed }}.stdout'

  - name: Verify the httpd service is started
    ansible.builtin.command: systemctl is-active httpd
    register: started

  - name: Is the httpd service started
    ansible.builtin.debug:
      msg: '{{ started }}.stdout'

  - name: Verify the httpd service is enabled
    ansible.builtin.command: systemctl is-enabled httpd
    register: enabled

  - name: Is the httpd service enabled
    ansible.builtin.debug:
      msg: '{{ enabled }}.stdout'


[student@workstation role-create]$ cat vhost.conf.j2 
# {{ ansible_managed }}

<VirtualHost *:80>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
    Options +Indexes +FollowSymlinks +Includes
    Order allow,deny
    Allow from all
    </Directory>
</VirtualHost>
```

Edit the `main.yml` file in the `tasks` subdirectory of the role. The role should perform the following tasks:

- Install the `httpd` package.
- Enable and start the `httpd` service.
- Install the web server configuration file using a template provided by the role.

```yaml
[student@workstation role-create]$ cat roles/myvhost/tasks/main.yml 
---
# tasks file for myvhost

- name: installing httpd
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Starting httpd svc
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

- name: Installing web server config file
  ansible.builtin.template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    owner: root
    group: root
    mode: 0644
  notify:
    - restart httpd
    
    
[student@workstation role-create]$ cat roles/myvhost/handlers/main.yml 
---
# handlers file for myvhost

- name: restart httpd
  ansible.builtin.service:
    name: httpd
    state: restarted

# copying vhost template
[student@workstation role-create]$ mv -v vhost.conf.j2 roles/myvhost/templates/
renamed 'vhost.conf.j2' -> 'roles/myvhost/templates/vhost.conf.j2'

# Creating html content
[student@workstation role-create]$ mkdir -pv files/html
mkdir: created directory 'files'
mkdir: created directory 'files/html'

[student@workstation role-create]$ echo 'simple index' > files/html/index.html

```

Creating the `use-vhost-role.yml` which we are using the role `myvhost` created before:

```yaml
---
- name: Use myvhost role playbook
  hosts: webservers
  pre_tasks:
    - name: pre_tasks message
      ansible.builtin.debug:
        msg: 'Ensure web server configuration.'

  roles:
    - myvhost

  post_tasks:
    - name: HTML content is installed
      ansible.builtin.copy:
        src: files/html/
        dest: "/var/www/vhosts/{{ ansible_hostname }}"

    - name: post_tasks message
      ansible.builtin.debug:
        msg: 'Web server is configured.'
```

check the result with:

```
student@workstation role-create]$ ansible-navigator run -m stdout use-vhost-role.yml --syntax-check
playbook: /home/student/role-create/use-vhost-role.yml
[student@workstation role-create]$ ansible-navigator run -m stdout use-vhost-role.yml
[student@workstation role-create]$ ansible-navigator run -m stdout verify-config.yml 
[student@workstation role-create]$ ansible-navigator run -m stdout verify-content.yml 
[student@workstation role-create]$ ansible-navigator run -m stdout verify-httpd.yml 
[student@workstation role-create]$ curl http://servera.lab.example.com
simple index
```





### Deploying Roles from External Content Sources

#### External Content Sources

If you are using roles in your Ansible Playbooks, then you should get those roles from some centrally managed source. This practice ensures that all your projects have the current version of the role and that each project can benefit from bug fixes discovered by other projects with which they share the role.

-  role that stored in a Git repository managed by your organization. 
- a distributed as a `tar` archive file from a website or through other means.
- the open source community maintains some roles through the Ansible Galaxy website. Note: These roles are not reviewed or supported officially by Red Hat, but might contain code that your organization finds useful.



It is increasingly common for roles to be packaged as Ansible Content Collections and offered by their authors through various methods:

- In Red Hat Certified Ansible Content Collections from the Red Hat hosted automation hub at `https://console.redhat.com`, or through a private automation hub
- As private content packaged from a private automation hub
- By the community from Ansible Galaxy at `https://galaxy.ansible.com`

This section only covers how to get roles that are not packaged into an Ansible Content Collections.



#### Introducing Ansible Galaxy

A public library of Ansible content written by a variety of Ansible administrators and users is available at [Ansible Galaxy](https://galaxy.ansible.com/). This library contains thousands of Ansible roles, and it has a searchable database that helps you identify roles that might help you accomplish an administrative task.

The `ansible-galaxy` command that you use to download and manage roles from Ansible Galaxy can also be used to download and manage roles from your own Git repositories.

#### The Ansible Galaxy Command Line Tool

You can use the `ansible-galaxy` command-line tool with the `role` argument to search for, display information about, install, list, remove, or initialize roles.

```
[user@host project]$ ansible-galaxy role -h
usage: ansible-galaxy role [-h] ROLE_ACTION ...

positional arguments:
  ROLE_ACTION
    init      Initialize new role with the base structure of a role.
    remove    Delete roles from roles_path.
    delete    Removes the role from Galaxy. It does not remove or alter the actual GitHub repository.
    list      Show the name and version of each role installed in the roles_path.
    search    Search the Galaxy database by tags, platforms, author and multiple keywords.
    import    Import a role into a galaxy server
    setup     Manage the integration between Galaxy and the given source.
    info      View more details about a specific role.
    install   Install role(s) from file(s), URL(s) or Ansible Galaxy

optional arguments:
  -h, --help   show this help message and exit
```

##### Installing Roles Using a Requirements File

If you have a playbook that must have specific roles installed, then you can create a `roles/requirements.yml` file in the project directory that specifies which roles are needed. This file acts as a dependency manifest for the playbook project that enables playbooks to be developed and tested separately from any supporting roles.

Then, before you run `ansible-navigator run`, you can use the `ansible-galaxy role` command to install those roles in your project's `roles` directory.

**Note**:

If you use automation controller, it automatically downloads roles specified in your `roles/requirements.yml` file when it runs your playbook.

For example, you could have a role in a public repository on a Git server at `https://git.example.com/someuser/someuser.myrole`. A simple `requirements.yml` to install `someuser.myrole` might contain the following content:

```yaml
- src: https://git.example.com/someuser/someuser.myrole
  scm: git
  version: "1.5.0"
```

The `src` attribute specifies the source of the role, in this case the URL for the repository of the role on your Git server. You can also use SSH key-based authentication by specifying something like `git@git.example.com:someuser/someuser.myrole` as provided by your Git repository.

The `scm` attribute indicates that this role is from a Git repository.

The `version` attribute is optional, and specifies the version of the role to install, in this case `1.5.0`. In the case of a role stored in Git, the version can be the name of a branch, a tag, or a Git commit hash. If you do not specify a version, the command uses the latest commit on the default branch.



**Important**:

Specify the **version** of the role in your `requirements.yml` file, especially for playbooks in production.

If you do not specify a version, you get the latest version of the role.

If the upstream author makes changes to the role that are incompatible with your playbook, it might cause an automation failure or other problems.

To install roles using a role requirements file, run the `ansible-galaxy role install` command from within your project directory. Include the following options:

- `-r roles/requirements.yml` to specify the location of your requirements file
- `-p roles` to install the role into a subdirectory of the `roles` directory

```
[user@host project]$ ansible-galaxy role install -r roles/requirements.yml \
> -p roles
Starting galaxy role install process
- downloading role from https://git.example.com/someuser/someuser.myrole
- extracting myrole to /home/user/project/roles/someuser.myrole
- someuser.myrole (1.5.0) was installed successfully
```

**Important**:

If you do not specify the `-p roles` option, then `ansible-galaxy role install` uses the first directory in the default `roles_path` setting to determine where to install the role. This defaults to the user's `~/.ansible/roles` directory, which is outside the project directory and unavailable to the execution environment if you use `ansible-navigator` to run your playbooks.

One way to avoid the need to specify `-p roles` is to apply the following setting in the `defaults` section of your project's `ansible.cfg` file:

```
roles_path = roles
```

If you have a `tar` archive file that contains a role, you can use `roles/requirements.yml` to install that file from a URL:

```yaml
# from a role tar ball, given a URL;
#   supports 'http', 'https', or 'file' protocols
- src: file:///opt/local/roles/tarrole.tar
  name: tarrole

- src: https://www.example.com/role-archive/someuser.otherrole.tar
  name: someuser.otherrole
```

**Note**:

Red Hat recommends that you use version control with roles, storing them in a version control system such as Git. If a recent change to a role causes problems, using version control allows you to roll back to a previous, stable version of the role.



#### Finding Community-managed Roles in Ansible Galaxy

The open source Ansible community operates a public server, `https://galaxy.ansible.com`, that contains roles and Ansible Content Collections shared by other Ansible users.

##### Browsing Ansible Galaxy for Roles

The Search tab on the left side of the Ansible Galaxy website home page gives you access to information about the roles published on Ansible Galaxy. You can search for an Ansible role by name or by using tags or other role attributes.

Results are presented in descending order of the `Best Match` score, which is a computed score based on role quality, role popularity, and search criteria. ([Content Scoring](https://galaxy.ansible.com/docs/contributing/content_scoring.html) in the Ansible Galaxy documentation has more information on how roles are scored by Ansible Galaxy.)



![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\ansible-galaxy-search.png)

Figure 7.1: Ansible Galaxy search screen

Ansible Galaxy reports the number of times each role has been downloaded from Ansible Galaxy. In addition, Ansible Galaxy also reports the number of watchers, forks, and stars the role's GitHub repository has. You can use this information to help determine how active development is for a role and how popular it is in the community.

The following figure shows the search results that Ansible Galaxy displayed after a keyword search for `redis` was performed.

![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\ansible-galaxy-redis-search.png)

Figure 7.2: Ansible Galaxy search results example

The Filters menu to the right of the search box allows searches by type, contributor type, contributor, cloud platform, deprecated, platform, and tags.

Possible platform values include `EL` for Red Hat Enterprise Linux (and closely related distributions such as CentOS) and `Fedora`, among others.

Tags are arbitrary single-word strings set by the role author that describe and categorize the role. You can use tags to find relevant roles. Possible tag values include `system`, `development`, `web`, `monitoring`, and others. In Ansible Galaxy, a role can have up to 20 tags.

**Important**:

In the Ansible Galaxy search interface, keyword searches match words or phrases in the `README` file, content name, or content description. Tag searches, by contrast, specifically match tag values that the author set for the role.



##### Searching for Roles from the Command Line

The `ansible-galaxy role search` command searches Ansible Galaxy for roles.

If you specify a string as an argument, it is used to search Ansible Galaxy for roles by keyword. You can use the `--author`, `--platforms`, and `--galaxy-tags` options to narrow the search results. You can also use those options as the main search key.

For example, the command `ansible-galaxy role search --author geerlingguy` displays all roles submitted by the user `geerlingguy`. Results are displayed in alphabetical order, not by descending `Best Match` score.

The following example displays the names of roles that include `redis`, and are available for the Enterprise Linux (`EL`) platform.

```
[user@host ~]$ ansible-galaxy role search 'redis' --platforms EL

Found 232 roles matching your search:

 Name                                Description
 ----                                -----------
 ...output omitted...
 aboveops.ct_redis                   Ansible role for creating Redis container
 adfinis-sygroup.redis               Ansible role for Redis
 adriano-di-giovanni.redis           Ansible role for Redis
 AerisCloud.redis                    Installs redis on a server
 alainvanhoof.alpine_redis           Redis for Alpine Linux
 ...output omitted...
```

The `ansible-galaxy role info` command displays more detailed information about a role. Ansible Galaxy gets this information from a number of places, including the role's `meta/main.yml` file and its GitHub repository.

The following command displays information about the `geerlingguy.redis` role, available from Ansible Galaxy.

```
[user@host ~]$ ansible-galaxy role info geerlingguy.redis

Role: geerlingguy.redis
        description: Redis for Linux
...output omitted...
        created: 2023-05-08T20:50:00.204075Z
        download_count: 1306420
        github_branch: master
        github_repo: ansible-role-redis
        github_user: geerlingguy
        id: 10990
        imported: 2022-09-26T12:10:34.707632-04:00
        modified: 2023-10-29T18:44:43.771821Z
        path: ('/home/student/.ansible/roles', '/usr/share/ansible/roles', '/etc/ansible/roles')
        upstream_id: 468
        username: geerlingguy
```

##### Downloading Roles from Ansible Galaxy

The following example shows how to configure a requirements file that uses a variety of remote sources.

```yaml
[user@host project]$ cat roles/requirements.yml
# from Ansible Galaxy, using the latest version
- src: geerlingguy.redis 

# from Ansible Galaxy, overriding the name and using a specific version
- src: geerlingguy.redis
  version: "1.5.0" 
  name: redis_prod

# from any Git based repository, using HTTPS
- src: https://github.com/geerlingguy/ansible-role-nginx.git
  scm: git 
  version: master
  name: nginx

# from a role tar ball, given a URL;
#   supports 'http', 'https', or 'file' protocols
- src: file:///opt/local/roles/myrole.tar 
  name: myrole 
  

Explanation:
# The `version` keyword is used to specify a role's version. The `version` keyword can be any value that corresponds to a branch, tag, or commit hash from the role's software repository.

# If the role is hosted in a source control repository, the `scm` attribute is required.
 
# If the role is hosted on Ansible Galaxy or as a tar archive, the `scm` keyword is omitted.
 
# The `name` keyword is used to override the local name of the role.
```

#### Managing Downloaded Roles

The `ansible-galaxy role` command can also manage local roles, such as those roles found in the `roles` directory of a playbook project. The `ansible-galaxy role list` command lists the local roles.

```
[user@host project]$ ansible-galaxy role list
# /home/user/project/roles
- geerlingguy.redis, 1.7.0
- redis_prod, 1.5.0
- nginx, master
- myrole, (unknown version)
...output omitted...
```

You can remove a role with the `ansible-galaxy role remove` command.

```
[user@host ~]$ ansible-galaxy role remove nginx
- successfully removed nginx
```

#### References

[ansible-galaxy — Ansible Documentation](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html)

[Red Hat Hybrid Cloud Console | Ansible Automation Platform Dashboard](https://console.redhat.com/ansible/ansible-dashboard)



#### Example

Create a role requirements file in your project directory that downloads the `student.bash_env` role. This role configures the default initialization files for the Bash shell used for newly created users, the default prompt for the accounts of these users, and the prompt color to use.

```
[student@workstation ~]$ cd role-galaxy/
[student@workstation role-galaxy]$ ll
total 68
-rw-r--r--. 1 student student   144 Sep  6 10:14 ansible.cfg
-rw-r--r--. 1 student student 61440 Sep  6 10:14 bash_env.tar
-rw-r--r--. 1 student student    67 Sep  6 10:14 inventory
drwxr-xr-x. 2 student student     6 Sep  6 10:15 roles

[student@workstation role-galaxy]$ cat ansible.cfg 
[defaults]
inventory=inventory
remote_user=devops

[privilege_escalation]
become=true
become_method=sudo
become_user=root
become_ask_pass=false

[student@workstation role-galaxy]$ cat inventory 
workstation.lab.example.com

[devservers]
servera.lab.example.com

[student@workstation role-galaxy]$ ll roles/
total 0
[student@workstation role-galaxy]$ cd roles/
[student@workstation roles]$ ll
total 0
```



Create a file called `requirements.yml` in the `roles` subdirectory. The URL of the role's Git repository is: `git@workstation.lab.example.com:student/bash_env`.

To see how the role affects the behavior of production hosts, use the `main` branch of the repository. Set the local name of the role to `student.bash_env`.

the `roles/requirements.yml` file contains the following content:

```yaml
---
# requirements.yml

- src: git@workstation.lab.example.com:student/bash_env
  scm: git
  version: main
  name: student.bash_env
  

[student@workstation role-galaxy]$ tree
.
├── ansible.cfg
├── bash_env.tar
├── inventory
└── roles
    └── requirements.yml

1 directory, 4 files
```

installing the role

```
[student@workstation role-galaxy]$ ansible-galaxy role install -r roles/requirements.yml -p roles/
Starting galaxy role install process
- extracting student.bash_env to /home/student/role-galaxy/roles/student.bash_env
- student.bash_env (main) was installed successfully

[student@workstation role-galaxy]$ tree
.
├── ansible.cfg
├── bash_env.tar
├── inventory
└── roles
    ├── requirements.yml
    └── student.bash_env
        ├── defaults
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        │   ├── _bash_profile.j2
        │   ├── _bashrc.j2
        │   └── _vimrc.j2
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

8 directories, 14 files

# look the roles folder
[student@workstation role-galaxy]$ ansible-galaxy role list
# /usr/share/ansible/roles
- linux-system-roles.certificate, (unknown version)
- linux-system-roles.cockpit, (unknown version)
- linux-system-roles.crypto_policies, (unknown version)
- linux-system-roles.firewall, (unknown version)
- linux-system-roles.ha_cluster, (unknown version)
...omited...
# /etc/ansible/roles
[WARNING]: - the configured path /home/student/.ansible/roles does not exist.

# use -p specific the roles folder, here we see the roles we created
[student@workstation role-galaxy]$ ansible-galaxy role list -p roles/
# /home/student/role-galaxy/roles
- student.bash_env, main
# /usr/share/ansible/roles
- linux-system-roles.certificate, (unknown version)
- linux-system-roles.cockpit, (unknown version)
```

Test1:

Create a playbook named `use-bash_env-role.yml` that uses the `student.bash_env` role. The playbook must have the following contents:

```yaml
---
- name: Use student.bash_env role playbook
  hosts: devservers
  vars:
    default_prompt: '[\u on \h in \W dir]\$ '
  pre_tasks:
    - name: Ensure test user does not exist
      ansible.builtin.user:
        name: student2
        state: absent
        force: true
        remove: true

  roles:
    - student.bash_env

  post_tasks:
    - name: Create the test user
      ansible.builtin.user:
        name: student2
        state: present
        password: "{{ 'redhat' | password_hash }}"
```

You must create a user account to see the effects of the configuration change. The `pre_tasks` and `post_tasks` section of the playbook ensure that the `student2` user account is deleted and created each time the playbook is run.

When you run the `use-bash_env-role.yml` playbook, the `student2` account is created with a password of `redhat`.

**Note**: 

The `student2` password is generated using a filter. Filters take data and modify it; here, the `redhat` string is modified by passing it to the `password_hash` filter to convert the value into a protected password hash. By defaul, the hashing algorithm used is `sha512`.



Run the `use-bash_env-role.yml` playbook.

```
[student@workstation role-galaxy]$ ansible-navigator run -m stdout use-bash_env-role.yml 

PLAY [use student.bash_env role in this playbook] ******************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Ensure test user does not exist] *****************************************
changed: [servera.lab.example.com]

TASK [student.bash_env : put away .bashrc] *************************************
changed: [servera.lab.example.com]

TASK [student.bash_env : put away .bash_profile] *******************************
ok: [servera.lab.example.com]

TASK [student.bash_env : put away .vimrc] **************************************
ok: [servera.lab.example.com]

TASK [create the test user] ****************************************************
[WARNING]: The input password appears not to have been hashed. The 'password'
argument must be encrypted for this module to work properly.
changed: [servera.lab.example.com]

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[student@workstation role-galaxy]$ ssh student2@servera
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
```



Test2:

Run the playbook using the development version of the `student.bash_env` role.

The development version of the role is located in the `dev` branch of the Git repository. The development version of the role uses a new variable, `prompt_color`.

Before running the playbook, add the `prompt_color` variable to the `vars` section of the playbook and set its value to `blue`.

1. Update the `roles/requirements.yml` file, and set the `version` value to `dev`. Ensure that the `roles/requirements.yml` file contains the following content:

   ```yaml
   ---
   # requirements.yml
   
   - src: git@workstation.lab.example.com:student/bash_env
     scm: git
     version: dev
     name: student.bash_env
   ```

2. Modify the `~/role-galaxy/ansible.cfg` and add the `roles_path` setting to the `[defaults]` section of the file. This sets the default roles path and enable you to omit the `-p roles` option when typing `ansible-galaxy` commands.

   When completed, the file contains then following content:

   ```
   [defaults]
   inventory=inventory
   remote_user=devops
   roles_path=roles
   
   [privilege_escalation]
   become=true
   become_method=sudo
   become_user=root
   become_ask_pass=false
   ```

3. Remove the existing version of the `student.bash_env` role from the `roles` subdirectory.

   ```
   [student@workstation role-galaxy]$ ansible-galaxy role remove student.bash_env
   - successfully removed student.bash_env
   ```

4. Use the `ansible-galaxy role install` command to install the role by using the updated requirements file.

   ```
   [student@workstation role-galaxy]$ ansible-galaxy role install \
   > -r roles/requirements.yml
   Starting galaxy role install process
   - extracting student.bash_env to /home/student/role-galaxy/roles/student.bash_env
   - student.bash_env (dev) was installed successfully
   ```

5. Modify the `use-bash_env-role.yml` file. Add the `prompt_color` variable with a value of `blue` to the `vars` section of the playbook. Ensure that the file contains the following content:

   ```yaml
   ---
   - name: Use student.bash_env role playbook
     hosts: devservers
     vars:
       prompt_color: blue
       default_prompt: '[\u on \h in \W dir]\$ '
     pre_tasks:
   ...output omitted...
   ```

6. Run the `use-bash_env-role.yml` playbook again and see students2 became blue prompt.

   ![image-20240906225100680](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\image-20240906225100680.png)



### Getting Roles and Modules from Content Collections

#### Ansible Content Collections

With *Ansible Content Collections*, Ansible code updates are separated from updates to modules and plug-ins. An Ansible Content Collection provides a set of related modules, roles, and other plug-ins that you can use in your playbooks. This approach enables vendors and developers to maintain and distribute their collections at their own pace, independently of Ansible releases.

For example:

- The `redhat.insights` content collection provides modules and roles that you can use to register a system with Red Hat Insights for Red Hat Enterprise Linux.
- The `cisco.ios` content collection, supported and maintained by Cisco, provides modules and plug-ins that manage Cisco IOS network appliances.

You can also select a specific version of a collection (possibly an earlier or later one) or choose between a version of a collection supported by Red Hat or vendors, or one provided by the community.

Ansible 2.9 and later support Ansible Content Collections. Upstream Ansible unbundled most modules from the core Ansible code in Ansible Base 2.10 and Ansible Core 2.11 and placed them in collections. Red Hat Ansible Automation Platform 2.2 provides automation execution environments based on Ansible Core 2.13 that inherit this feature.

You can develop your own collections to provide custom roles and modules to your teams. course: *Developing Advanced Automation with Red Hat Ansible Automation Platform (DO374)*.



#### Namespaces for Ansible Content Collections

The namespace is the first part of a collection name. For example, all the collections that the Ansible community maintains are in the `community` namespace, and have names like `community.crypto`, `community.postgresql`, and `community.rabbitmq`. 

#### Selecting Sources of Ansible Content Collections

Regardless of whether you are using `ansible-navigator` with the minimal automation execution environment or `ansible-playbook` on bare metal Ansible Core, you always have at least one Ansible Content Collection available to you: `ansible.builtin`.

In addition, your automation execution environment might have additional automation execution environments built into it, for example, the default execution environment used by Red Hat Ansible Automation Platform 2.2, `ee-supported-rhel8`.

**If you need to have additional Ansible Content Collections, you can add them to the `collections` subdirectory of your Ansible project.** You might obtain Ansible Content Collections from several sources:

- Automation Hub

  Automation hub is a service provided by Red Hat to distribute Red Hat Certified Ansible Content Collections that are supported by Red Hat and ecosystem partners. You need a valid Red Hat Ansible Automation Platform subscription to access automation hub. Use the automation hub web UI at https://console.redhat.com/ansible/automation-hub/ to browse these collections.

- Private Automation Hub

  Your organization might have its own on-site private automation hub, and might also use that hub to distribute its own Ansible Content Collections. Private automation hub is included with Red Hat Ansible Automation Platform.

- Ansible Galaxy

  Ansible Galaxy is a community-supported website that hosts Ansible Content Collections that have been submitted by a variety of Ansible developers and users. Ansible Galaxy is a public library that provides no formal support guarantees and that is not curated by Red Hat. For example, the `community.crypto`, `community.postgresql`, and `community.rabbitmq` collections are all available from that platform.Use the Ansible Galaxy web UI at https://galaxy.ansible.com/ to search it for collections.

- Third-Party Git Repository or Archive File

  You can also download Ansible Content Collections from a Git repository or a local or remote `tar` archive file, much like you can download roles.

#### Installing Ansible Content Collections

The Ansible configuration `collections_paths` setting specifies a colon separated list of paths on the system where Ansible looks for installed collections.

You can set this directive in the `ansible.cfg` configuration file.

The following default value references the `collections_paths` directive.

```
~/.ansible/collections:/usr/share/ansible/collections
```

The following example uses the `ansible-galaxy collection install` command to download and install the `community.crypto` Ansible Content Collection. The `-p collections` option installs the collection in the local `collections` subdirectory.

```
[user@controlnode ~]$ ansible-galaxy collection install community.crypto -p collections
```


**Important**:

You must specify the `-p collections` option or `ansible-galaxy` installs the collection based on your current `collections_paths` setting, or into your `~/.ansible/collections/` directory on the control node by default. The `ansible-navigator` command does not load this directory into the automation execution environment, although this directory is available to Ansible commands that use the control node as the execution environment, such as `ansible-playbook`.

When you install the `community.crypto` collection, you could see a warning that your Ansible project's playbooks might not find the collection because the specified path is not part of the configured Ansible collections path.

You can safely ingore this warning because your Ansible project checks the local `collections` subdirectory **before** checking directories specified by the `collections_paths` setting and can use the collections stored there.

The command can also install a collection from a local or a remote `tar` archive, or a Git repository. **A Git repository must have a valid `galaxy.yml` or `MANIFEST.json` file that provides metadata about the collection, such as its namespace and version number.** 

For example, see the `community.general` collection at `https://github.com/ansible-collections/community.general`.

```
[user@controlnode ~]$ ansible-galaxy collection install \
> /tmp/community-dns-1.2.0.tar.gz -p collections
...output omitted...
[user@controlnode ~]$ ansible-galaxy collection install \
> http://www.example.com/redhat-insights-1.0.5.tar.gz -p collections
...output omitted...
[user@controlnode ~]$ ansible-galaxy collection install \
> git@git.example.com:organization/repo_name.git -p collections
```

#### Installing Ansible Content Collections with a Requirements File

If your Ansible project needs additional Ansible Content Collections, you can create a `collections/requirements.yml` file in the project directory that lists all the collections that the project requires. Automation controller detects this file and automatically installs the specified collections before running your playbooks.

A requirements file for Ansible Content Collections is a YAML file that consists of a `collections` dictionary key that has the list of collections to install as its value. Each list item can also specify the particular version of the collection to install, as shown in the following example:

```yaml
---
collections: 
  - name: community.crypto 

  - name: ansible.posix
    version: 1.2.0 

  - name: /tmp/community-dns-1.2.0.tar.gz 

  - name: http://www.example.com/redhat-insights-1.0.5.tar.gz 

  - name: git+https://github.com/ansible-collections/community.general.git 
    version: main
```

The `ansible-galaxy` command can then use the `collections/requirements.yml` file to install all those collections. Specify the requirements file with the `--requirements-file` (or `-r`) option, and use the `-p collections` option to install the Ansible Content Collection into the `collections` subdirectory.

```
[root@controlnode ~]# ansible-galaxy collection install \
> -r collections/requirements.yml -p collections
```

#### Configuring Ansible Content Collection Sources

By default, the `ansible-galaxy` command uses Ansible Galaxy at https://galaxy.ansible.com/ to download Ansible Content Collections. You might not want to use this command, preferring automation hub or your own private automation hub. Alternatively, you might want to try automation hub first, and then try Ansible Galaxy.

You can configure the sources that `ansible-galaxy` uses to get Ansible Content Collections in your Ansible project's `ansible.cfg` file. The relevant parts of that file might look like the following example:

```yaml
...output omitted...
[galaxy]
server_list = automation_hub, galaxy  

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/  
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token 
token=eyJh...Jf0o 

[galaxy_server.galaxy]
url=https://galaxy.ansible.com/
```

Instead of a token, you can use the `username` and `password` parameters to provide your customer portal username and password.

```
...output omitted...
[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
username=operator1
password=Sup3r53cR3t
...output omitted...
```

However, you might not want to expose your credentials in the `ansible.cfg` file because the file could potentially get committed when using version control.

It is preferable to remove the authentication parameters from the `ansible.cfg` file and define them in environment variables, as shown in the following example:

```
export ANSIBLE_GALAXY_SERVER_<server_id>_<key>=value
```

- `server_id`

  Server identifier in uppercase. The server identifier is the name you used in the `server_list` parameter and in the name of the `[galaxy_server.*`server_id`*]` section.

- `key`

  Name of the parameter in uppercase.

The following example provides the `token` parameter as an environment variable:

```
[user@controlnode ~]$ cat ansible.cfg
...output omitted...
[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
[user@controlnode ~]$ export \
> ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN='eyJh...Jf0o'
[user@controlnode ~]$ ansible-galaxy collection install ansible.posix \
> -p collections
```

#### Using Resources from Ansible Content Collections

You can use the `ansible-navigator collections` command in your Ansible project directory to list all the collections that are installed in your automation execution environment. This includes the Ansible Content Collections in your project's `collections` subdirectory.

You can select the line number of the collection in the interactive mode of `ansible-navigator` to view its contents. Then, you can select the line number of a module, role, or other plug-in to see its documentation. You can also use other tools, such as `ansible-navigator doc`, with the FQCN of a module to view that module's documentation.

The following playbook invokes the `mysql_user` module from the `community.mysql` collection for a task.

```yaml
---
- name: Create the operator1 user in the test database
  hosts: db.example.com

  tasks:
    - name: Ensure the operator1 database user is defined
      community.mysql.mysql_user:
        name: operator1
        password: Secret0451
        priv: '.:ALL'
        state: present
```

The following playbook uses the `organizations` role from the `redhat.satellite` collection.

```yaml
---
- name: Add the test organizations to Red Hat Satellite
  hosts: localhost

  tasks:
    - name: Ensure the organizations exist
      include_role:
        name: redhat.satellite.organizations
      vars:
        satellite_server_url: https://sat.example.com
        satellite_username: admin
        satellite_password: Sup3r53cr3t
        satellite_organizations:
          - name: test1
            label: tst1
            state: present
            description: Test organization 1
          - name: test2
            label: tst2
            state: present
            description: Test organization 2
```

#### References

[Ansible Automation Platform Certified Content](https://access.redhat.com/articles/3642632)

[Ansible Certified Content FAQ](https://access.redhat.com/articles/4916901)

[Using collections — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)

[Galaxy User Guide — Ansible Documentation](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html)



#### Example

Install the collection from tar or requirement.yml

```yaml
[student@workstation role-collections]$ ll
total 3040
-rw-r--r--. 1 student student     146 Sep  9 03:05 ansible.cfg
-rw-r--r--. 1 student student     419 Sep  9 03:05 bck.yml
-rw-r--r--. 1 student student 2235450 Sep  9 03:05 community-general-5.5.0.tar.gz
-rw-r--r--. 1 student student    6234 Sep  9 03:05 gls-utils-0.0.1.tar.gz
-rw-r--r--. 1 student student     199 Sep  9 03:05 inventory
-rw-r--r--. 1 student student     519 Sep  9 03:05 new_system.yml
-rw-r--r--. 1 student student   34218 Sep  9 03:05 redhat-insights-1.0.7.tar.gz
-rw-r--r--. 1 student student  808333 Sep  9 03:05 redhat-rhel_system_roles-1.19.3.tar.gz
-rw-r--r--. 1 student student     239 Sep  9 03:05 requirements.yml

[student@workstation role-collections]$ cat ansible.cfg 
[defaults]
inventory=./inventory
remote_user=devops

[privilege_escalation]
become=true
become_method=sudo
become_user=root
become_ask_pass=false


[student@workstation role-collections]$ cat bck.yml 
---
- name: Backup the system configuration
  hosts: servera.lab.example.com
  become: true
  gather_facts: false

  tasks:
    - name: Ensure the machine is up
      gls.utils.newping:
        data: pong

    - name: Ensure configuration files are saved
      ansible.builtin.include_role:
        name: gls.utils.backup
      vars:
        backup_id: backup_etc
        backup_files:
          - /etc/sysconfig
          - /etc/yum.repos.d
[student@workstation role-collections]$ cat inventory 
[controlnode]
workstation.lab.example.com


[na_datacenter]
servera.lab.example.com

[europe_datacenter]
serverb.lab.example.com


[database_servers]
servera.lab.example.com
serverb.lab.example.com

[student@workstation role-collections]$ cat new_system.yml 
---
- name: Configure the system
  hosts: servera.lab.example.com
  become: true
  gather_facts: true

  tasks:
    - name: Ensure the system is registered with Insights
      ansible.builtin.include_role:
        name: redhat.insights.insights_client
      vars:
        auto_config: false
        insights_proxy: http://proxy.example.com:8080

    - name: Ensure SELinux mode is Enforcing
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.selinux
      vars:
        selinux_state: enforcing
[student@workstation role-collections]$ cat requirements.yml 
---
collections:
  - name: /home/student/role-collections/redhat-insights-1.0.7.tar.gz
  - name: /home/student/role-collections/redhat-rhel_system_roles-1.19.3.tar.gz
  - name: /home/student/role-collections/community-general-5.5.0.tar.gz
```

```
[student@workstation role-collections]$ ansible-galaxy collection install gls-utils-0.0.1.tar.gz -p collections

# entering TUI to check collections
[student@workstation role-collections]$ ansible-navigator collections


[student@workstation role-collections]$ ansible-navigator run -m stdout bck.yml

[student@workstation role-collections]$ ansible-galaxy collection install -r requirements.yml -p collections


[student@workstation role-collections]$ ansible-galaxy collection list -p collections

# /usr/share/ansible/collections/ansible_collections
Collection               Version
------------------------ -------
redhat.rhel_system_roles 1.16.2 

# /home/student/role-collections/collections/ansible_collections
Collection               Version
------------------------ -------
community.general        5.5.0  
gls.utils                0.0.1  
redhat.insights          1.0.7  
redhat.rhel_system_roles 1.19.3 


[student@workstation role-collections]$ ansible-navigator run -m stdout new_system.yml --check
```







### Reusing Content with System Roles

#### System Roles

*System roles* are a set of Ansible roles that you can use to configure and manage various components, subsystems, and software packages included with Red Hat Enterprise Linux. System roles provide automation for many common system configuration tasks, including time synchronization, networking, firewall, tuning, and logging.

These roles are intended to provide an automation API that is consistent across multiple major and minor releases of Red Hat Enterprise Linux. The Knowledgebase article at `https://access.redhat.com/articles/3050101` documents the versions of Red Hat Enterprise Linux on your managed hosts that specific roles support.

##### Simplified Configuration Management

System roles can help you simplify automation across multiple versions of Red Hat Enterprise Linux. For example, the recommended time synchronization service is different in different versions of Red Hat Enterprise Linux.

- The `chronyd` service is preferred in RHEL 9.
- The `ntpd` service is preferred in RHEL 6.

You can use an Ansible Playbook that runs the `redhat.rhel_system_roles.timesync` role to configure time synchronization for managed hosts running either version of Red Hat Enterprise Linux.



##### Support for System Roles

System roles are provided as the Red Hat Certified Ansible Content Collection `redhat.rhel_system_roles` through automation hub for Red Hat Ansible Automation Platform customers, as part of their subscription.

In addition, the roles are provided as an RPM package (`rhel-system-roles`) with Red Hat Enterprise Linux 9 for use with the version of Ansible Core provided by that operating system. 

Some system roles are in "Technology Preview". These are tested and are stable, but might be subject to future changes that are incompatible with the current state of the role. Integration testing is recommended for playbooks that incorporate any "Technology Preview" role. Playbooks might require refactoring if role variables change in a future version of the role.

You can access documentation for the system roles in `ansible-navigator`, or by reviewing the documentation on the Red Hat Customer Portal at [*Administration and configuration tasks using System Roles in RHEL* ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/administration_and_configuration_tasks_using_system_roles_in_rhel/index).

#### Installing the System Roles Ansible Content Collection

You can install the `redhat.rhel_system_roles` Ansible Content Collection with the `ansible-galaxy collection install` command. An Ansible project can specify this dependency by creating a `collections/requirements.yml` file. The following example assumes that your project is set up to pull Ansible Content Collections from automation hub.

```yaml
---
collections:
  - name: redhat.rhel_system_roles
```

The following command installs any required collections for a given project into the project's `collections/` directory.

```
[user@demo demo-project]$ ansible-galaxy collection \
> install -p collections/ -r collections/requirements.yml
...output omitted...
```



Upstream development is done through the Linux System Roles project; their development versions of the system roles are provided through Ansible Galaxy as the `fedora.linux_system_roles` Ansible Content Collection. These roles are not supported by Red Hat.

#### Example: Time Synchronization Role

Suppose you need to configure NTP time synchronization on your web servers. You could write automation yourself to perform each of the necessary tasks. But the system roles collection includes a role that can perform the configuration, `redhat.rhel_system_roles.timesync`.

The documentation for this role describes all the variables that affect the role's behavior and provides three playbook snippets that illustrate different time synchronization configurations.

To manually configure NTP servers, the role has a variable named `timesync_ntp_servers`. This variable defines a list of NTP servers to use. Each item in the list is made up of one or more attributes. The two key attributes are `hostname` and `iburst`.

| Attribute  | Purpose                                                      |
| :--------- | :----------------------------------------------------------- |
| `hostname` | The hostname of an NTP server with which to synchronize.     |
| `iburst`   | A Boolean that enables or disables fast initial synchronization. Defaults to `no` in the role, but you should normally set this to `yes`. |

Given this information, the following example is a play that uses the `redhat.rhel_system_roles.timesync` role to configure managed hosts to get time from three NTP servers by using fast initial synchronization.

```yaml
- name: Time Synchronization Play
  hosts: webservers
  vars:
    timesync_ntp_servers:
      - hostname: 0.rhel.pool.ntp.org
        iburst: yes
      - hostname: 1.rhel.pool.ntp.org
        iburst: yes
      - hostname: 2.rhel.pool.ntp.org
        iburst: yes

  roles:
    - redhat.rhel_system_roles.timesync
```

This example sets the role variables in a `vars` section of the play, but a better practice might be to configure them as inventory variables for hosts or host groups.

Consider an example project with the following structure:

```
[user@demo demo-project]$ tree
.
├── ansible.cfg
├── group_vars
│   └── webservers
│       └── timesync.yml
├── inventory
└── timesync_playbook.yml
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#role-system-_example_time_synchronization_role-CO12-1) | Defines the time synchronization variables overriding the role defaults for hosts in group `webservers` in the inventory. This file would look something like:<br />*timesync_ntp_servers:  <br />*  *- hostname: 0.rhel.pool.ntp.org    <br />       iburst: yes  <br />  - hostname: 1.rhel.pool.ntp.org    <br />       iburst: yes  <br />  - hostname: 2.rhel.pool.ntp.org    <br />       iburst: yes* |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#role-system-_example_time_synchronization_role-CO12-2) | The content of the playbook simplifies to:                   |

```yaml
- name: Time Synchronization Play
  hosts: webservers
  roles:
    - redhat.rhel_system_roles.timesync
```

This structure cleanly separates the role, the playbook code, and configuration settings. The playbook code is simple, easy to read, and should not require complex refactoring. 

This structure also supports a dynamic, heterogeneous environment. Hosts with new time synchronization requirements might be placed in a new host group. Appropriate variables are defined in a YAML file and placed in the appropriate `group_vars` (or `host_vars`) subdirectory.

#### Example: SELinux Role

As another example, the `redhat.rhel_system_roles.selinux` role simplifies management of SELinux configuration settings. This role is implemented using the SELinux-related Ansible modules. The advantage of using this role instead of writing your own tasks is that it relieves you from the responsibility of writing those tasks. Instead, you provide variables to the role to configure it, and the maintained code in the role ensures your desired SELinux configuration is applied.

This role can perform the following tasks:

- Set enforcing or permissive mode
- Run `restorecon` on parts of the file system hierarchy
- Set SELinux Boolean values
- Set SELinux file contexts persistently
- Set SELinux user mappings

##### Calling the SELinux Role

Sometimes, the SELinux role must ensure that the managed hosts are rebooted in order to completely apply its changes. However, the role does not ever reboot hosts itself, enabling you to control how the reboot is handled. Therefore, it is a little more complicated than usual to properly use this role in a play.

The role sets a Boolean variable, `selinux_reboot_required`, to `true` and fails if a reboot is needed. You can use a `block`/`rescue` structure to recover from the failure by failing the play if that variable is not set to `true`, or rebooting the managed host and rerunning the role if it is `true`. The block in your play should look something like this:

```yaml
    - name: Apply SELinux role
      block:
        - include_role:
            name: redhat.rhel_system_roles.selinux
      rescue:
        - name: Check for failure for other reasons than required reboot
          ansible.builtin.fail:
          when: not selinux_reboot_required

        - name: Restart managed host
          ansible.builtin.reboot:

        - name: Reapply SELinux role to complete changes
          include_role:
            name: redhat.rhel_system_roles.selinux
```

##### Configuring the SELinux Role

The variables used to configure the `redhat.rhel_system_roles.selinux` role are described in the role's documentation. The following examples show some ways to use this role.

The `selinux_state` variable sets the mode that SELinux runs in. It can be set to `enforcing`, `permissive`, or `disabled`. If it is not set, the mode is not changed.

```yaml
selinux_state: enforcing
```

The `selinux_booleans` variable takes a list of SELinux Boolean values to adjust. Each item in the list is a dictionary of variables: the `name` of the Boolean, the `state` (whether it should be `on` or `off`), and whether the setting should be `persistent` across reboots.

This example sets `httpd_enable_homedirs` to `on` persistently:

```yaml
selinux_booleans:
  - name: 'httpd_enable_homedirs'
    state: 'on'
    persistent: 'yes'
```

The `selinux_fcontexts` variable takes a list of file contexts to persistently set (or remove), and works much like the `selinux fcontext` command.

The following example ensures the policy has a rule to set the default SELinux type for all files under `/srv/www` to `httpd_sys_content_t`.

```yaml
selinux_fcontexts:
  - target: '/srv/www(/.*)?'
    setype: 'httpd_sys_content_t'
    state: 'present'
```

The `selinux_restore_dirs` variable specifies a list of directories on which to run the `restorecon` command:

```yaml
selinux_restore_dirs:
  - /srv/www
```

The `selinux_ports` variable takes a list of ports that should have a specific SELinux type.

```yaml
selinux_ports:
  - ports: '82'
    setype: 'http_port_t'
    proto: 'tcp'
    state: 'present'
```

There are other variables and options for this role. See the role documentation for more information.

#### Using System Roles with Ansible Core Only

This section is relevant if you plan to use system roles without a Red Hat Ansible Automation Platform subscription, by using the version of Ansible Core that is provided with Red Hat Enterprise Linux 9.

That version of Ansible Core is only supported for use with system roles and other automation code provided by Red Hat. In addition, it does not include `ansible-navigator`, so you have to use tools like `ansible-playbook` that treat your control node as the execution environment to run your automation.

Because `ansible-playbook` does not use execution environments, the only Ansible Content Collection that you have available by default is `ansible.builtin` collection. Other packages included with Red Hat Enterprise Linux might add additional Ansible Content Collections to the control node.

##### Installing the System Roles RPM Package

Make sure that your control node is registered with Red Hat Subscription Manager and has a Red Hat Enterprise Linux subscription. You should also install the `ansible-core` RPM package.

To install the `rhel-system-roles` RPM package, make sure that the AppStream package repository is enabled. For Red Hat Enterprise Linux 9 on the x86_64 processor architecture, this is the `rhel-9-for-x86_64-appstream-rpms` repository. Then, you can install the package.

```
[user@controlnode ~]$ sudo dnf install rhel-system-roles
```

If you use Ansible Core from a basic Red Hat Enterprise Linux installation for your control node, and do not have a Red Hat Ansible Automation Platform subscription on that node, then your control node should be a fully updated installation of the most recent version of Red Hat Enterprise Linux. You should also use the most recent version of the `ansible-core` and `rhel-system-roles` packages.

After installation, the collection is installed in the `/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles` directory on your control node. The individual roles are also installed in the `/usr/share/ansible/roles` directory for backward compatibility:

```
[user@controlnode ~]$ ls -1F /usr/share/ansible/roles/
linux-system-roles.certificate@
linux-system-roles.cockpit@
linux-system-roles.crypto_policies@
linux-system-roles.firewall@
linux-system-roles.ha_cluster@
linux-system-roles.kdump@
linux-system-roles.kernel_settings@
linux-system-roles.logging@
linux-system-roles.metrics@
linux-system-roles.nbde_client@
linux-system-roles.nbde_server@
linux-system-roles.network@
linux-system-roles.postfix@
linux-system-roles.selinux@
linux-system-roles.ssh@
linux-system-roles.sshd@
linux-system-roles.storage@
linux-system-roles.timesync@
linux-system-roles.tlog@
linux-system-roles.vpn@
rhel-system-roles.certificate/
rhel-system-roles.cockpit/
rhel-system-roles.crypto_policies/
rhel-system-roles.firewall/
rhel-system-roles.ha_cluster/
rhel-system-roles.kdump/
rhel-system-roles.kernel_settings/
rhel-system-roles.logging/
rhel-system-roles.metrics/
rhel-system-roles.nbde_client/
rhel-system-roles.nbde_server/
rhel-system-roles.network/
rhel-system-roles.postfix/
rhel-system-roles.selinux/
rhel-system-roles.ssh/
rhel-system-roles.sshd/
rhel-system-roles.storage/
rhel-system-roles.timesync/
rhel-system-roles.tlog/
rhel-system-roles.vpn/
```

The corresponding upstream name of each role is linked to the system role. This enables individual roles to be referenced in a playbook by either name.



If you are using `ansible-playbook` to run your playbook, and your playbook refers to a system role that was installed using the RPM package's FQCN, you must use the `redhat.rhel_system_roles` version of its name. For example, you could refer to the `firewall` role as:

- `redhat.rhel_system_roles.firewall` (its FQCN in the collection)
- `rhel-system-roles.firewall` (its name as an independent role)
- `linux-system-roles.firewall` (its name as the upstream independent role)

You cannot use `fedora.linux_system_roles.firewall` because the `fedora.linux_system_roles` collection is not installed on the system.

In addition, the independent role names only work if `/usr/share/ansible/roles` is in your `roles_path` setting.

##### Accessing Documentation for System Roles

If you are working with system roles in Red Hat Enterprise Linux and do not have `ansible-navigator` available to you, there are other ways to get documentation about system roles.

The official documentation for system roles is located at [*Administration and configuration tasks using System Roles in RHEL* ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/administration_and_configuration_tasks_using_system_roles_in_rhel/index).

However, if you installed the system roles from the RPM package, documentation is also available under the `/usr/share/doc/rhel-system-roles/` directory.

```
[user@controlnode ~]$ ls -1 /usr/share/doc/rhel-system-roles/
certificate
cockpit
collection
crypto_policies
firewall
ha_cluster
kdump
kernel_settings
logging
metrics
nbde_client
nbde_server
network
postfix
selinux
ssh
sshd
storage
timesync
tlog
vpn
```

Each role's documentation directory contains a `README.md` file. The `README.md` file contains a description of the role, along with information on how to use it.

The `README.md` file also describes role variables that affect the behavior of the role. Often the `README.md` file contains a playbook snippet that demonstrates variable settings for a common configuration scenario.



##### Running Playbooks Without Automation Content Navigator

You can use the `ansible-playbook` command to run a playbook that uses system roles in Red Hat Enterprise Linux when you do not have Red Hat Ansible Automation Platform or `ansible-navigator``.

The syntax of `ansible-playbook` is very similar to `ansible-navigator run -m stdout`, and takes many of the same options.

```
[user@controlnode ~]$ ansible-playbook playbook.yml
```

#### References

[Red Hat Enterprise Linux (RHEL) System Roles](https://access.redhat.com/articles/3050101)

[Red Hat Enterprise Linux System Roles Ansible Collection](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/rhel_system_roles)

[Scope of support for the Ansible Core package included in the RHEL 9 and RHEL 8.6 and later AppStream repositories](https://access.redhat.com/articles/6325611)

[Using Ansible in RHEL 9](https://access.redhat.com/articles/6393321)

[Linux System Roles](https://linux-system-roles.github.io/)

[Linux System Roles Ansible Collection](https://galaxy.ansible.com/fedora/linux_system_roles)



#### Example

Add the `collections_paths` key to the `ansible.cfg` file, so that the `./collections` directory is searched.

```
[defaults]
inventory=./inventory
remote_user=devops
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections
```

install the `redhat.rhel_system_roles` collection from the tar

```
[student@workstation role-system]$ ansible-galaxy collection \
> install -p collections/ redhat-rhel_system_roles-1.19.3.tar.gz

[student@workstation role-system]$ ansible-galaxy collection list

# /home/student/role-system/collections/ansible_collections
Collection               Version
------------------------ -------
redhat.rhel_system_roles 1.19.3

```

Create the `configure_time.yml` playbook with one play that targets the `database_servers` host group and runs the `redhat.rhel_system_roles.timesync` role in its `roles` section.



Create the `group_vars/all` subdirectory.

```
[student@workstation role-system]$ mkdir -pv group_vars/all
mkdir: created directory 'group_vars'
mkdir: created directory 'group_vars/all'
```

Create a `group_vars/all/timesync.yml` file, adding variable definitions to satisfy the time synchronization requirements. The file now contains:

```yaml
---
#redhat.rhel_system_roles.timesync variables for all hosts

timesync_ntp_provider: chrony

timesync_ntp_servers:
  - hostname: classroom.example.com
    iburst: yes
```

Add two tasks to the `configure_time.yml` file to get and conditionally set the time zone for each host. Ensure that both tasks run after the `redhat.rhel_system_roles.timesync` role.



`configure_time.yml` file is like:

```yaml
---
- name: Time Synchronization
  hosts: database_servers

  roles:
    - redhat.rhel_system_roles.timesync

  post_tasks:
    - name: Get time zone
      ansible.builtin.command: timedatectl show
      register: current_timezone
      changed_when: false

    - name: Set time zone
      ansible.builtin.command: "timedatectl set-timezone {{ host_timezone }}"
      when: host_timezone not in current_timezone.stdout
      notify: reboot host

  handlers:
    - name: reboot host
      ansible.builtin.reboot:
```

For each data center, create a file named `timezone.yml` that contains an appropriate value for the `host_timezone` variable. Use the `timedatectl list-timezones` command to find the valid time zone string for each data center.

```
[student@workstation role-system]$ mkdir -pv \
> group_vars/{na_datacenter,europe_datacenter}

[student@workstation role-system]$ timedatectl list-timezones | grep Chicago
America/Chicago
[student@workstation role-system]$ timedatectl list-timezones | grep Helsinki
Europe/Helsinki

[student@workstation role-system]$ echo "host_timezone: America/Chicago" > \
> group_vars/na_datacenter/timezone.yml
[student@workstation role-system]$ echo "host_timezone: Europe/Helsinki" > \
> group_vars/europe_datacenter/timezone.yml

```

run the playbook

```
[student@workstation role-system]$ ansible-navigator run \
> -m stdout configure_time.yml
```

examine the result:

```
[student@workstation role-system]$ ssh servera date
Tue Sep 10 07:43:33 PM CDT 2024
[student@workstation role-system]$ ssh serverb date
Wed Sep 10 03:43:41 AM EEST 2024
```



### Chapter 7 Example

```yaml
[student@workstation role-review]$ ll
total 1540
-rw-r--r--.  1 student student    232 Sep 11 10:35 ansible.cfg
drwxr-xr-x. 10 student student    154 Sep 11 21:46 apache.developer_configs
-rw-r--r--.  1 student student 143360 Sep 11 10:35 apache.tar
drwxr-xr-x.  3 student student     33 Sep 11 10:42 collections
-rw-r--r--.  1 student student    595 Sep 11 10:35 developer.conf.j2
-rw-r--r--.  1 student student   1566 Sep 11 10:35 developer_tasks.yml
-rw-r--r--.  1 student student     83 Sep 11 10:35 inventory
-rw-r--r--.  1 student student    396 Sep 11 10:35 jdoe2.key.pub
-rw-r--r--.  1 student student    396 Sep 11 10:35 jdoe.key.pub
-rw-r--r--.  1 student student 808333 Sep 11 10:35 redhat-rhel_system_roles-1.19.3.tar.gz
-rw-r--r--.  1 student student    231 Sep 11 10:35 selinux.yml
-rw-r--r--.  1 student student    138 Sep 11 10:35 web_developers.yml
-rw-r--r--.  1 student student    715 Sep 11 22:49 web_dev_server.yml

[student@workstation role-review]$ cat ansible.cfg 
[defaults]
inventory=./inventory
remote_user=devops
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
[student@workstation role-review]$ 
[student@workstation role-review]$ 
[student@workstation role-review]$ cat inventory 
[controlnode]
workstation.lab.example.com

[dev_webserver]
servera.lab.example.com
[student@workstation role-review]$ 
[student@workstation role-review]$ 
[student@workstation role-review]$ cat developer.conf.j2 
#This is an Ansible Template
# 'item' is a user from the web_developers user list,
#  and defines:
#    - user_port
#    - username
#    - name

Listen {{ item['user_port'] }}

<VirtualHost *:{{ item['user_port'] }}>
    ServerName {{ inventory_hostname }}

    ErrorLog "logs/{{ item['username'] }}_error_log"
    CustomLog "logs/{{ item['username'] }}_access_log" common

    DocumentRoot /srv/{{ item['username'] }}/www


    <Directory /srv/{{ item['username'] }}/www >
        Options FollowSymLinks

        AllowOverride All
        Require all granted

    </Directory>

</VirtualHost>

[student@workstation role-review]$ 
[student@workstation role-review]$ 
[student@workstation role-review]$ cat developer_tasks.yml 
---
# tasks file for apache.developer_configs

- name: Create user accounts
  ansible.builtin.user:
    name: "{{ item['username'] }}"
    state: present
  loop: "{{ web_developers }}"

#Allows 'student' user to login to
# any of the user accounts - for the lab only.
# Jinja syntax for 'key' keyword available from the documentation.
- name: Give student access to all accounts
  ansible.posix.authorized_key:
    user: "{{ item['username'] }}"
    state: present
    key: "{{ lookup('file', '/home/student/role-review/'+ item['username'] + '.key.pub') }}"
  loop: "{{ web_developers }}"

- name: Create content directory
  ansible.builtin.file:
    path: /srv/{{ item['username'] }}/www
    state: directory
    owner: "{{ item['username'] }}"
  loop: "{{ web_developers }}"

- name: Create skeleton index.html if needed
  ansible.builtin.copy:
    content: "This is index.html for user: {{ item['name'] }} ({{ item['username'] }})\n"
    dest: /srv/{{ item['username'] }}/www/index.html
    force: false
    owner: "{{ item['username'] }}"
    group: root
    mode: 0664
  loop: "{{ web_developers }}"


- name: Set firewall port
  ansible.posix.firewalld:
    port: "{{ item['user_port'] }}/tcp"
    permanent: true
    state: enabled
  loop: "{{ web_developers }}"
  notify: restart firewalld

- name: Copy Per-Developer Config files
  ansible.builtin.template:
    src: developer.conf.j2
    dest: "/etc/httpd/conf.d/developer_{{ item['username'] }}.conf"
    owner: root
    group: root
    mode: 0644
  loop: "{{ web_developers }}"
  notify: restart apache

[student@workstation role-review]$ 

# install rhel roles
[student@workstation role-review]$ ansible-galaxy collection \
> install -p collections/ redhat-rhel_system_roles-1.19.3.tar.gz

[student@workstation role-review]$ ansible-galaxy collection list

#Make sure to install any roles that are dependencies of roles in your playbook.
the apache.developer_configs role that you write later in this lab depends on the infra.apache role.
Create a roles/requirements.yml file. This file installs the role from the Git repository at git@workstation.lab.example.com:infra/apache, use version v1.4, and name it infra.apache locally.

# create roles dir
[student@workstation role-review]$ mkdir -v roles
```

`roles/requirements.yml` file contains the following content:

```yaml
- name: infra.apache
  src: git@workstation.lab.example.com:infra/apache
  scm: git
  version: v1.4
```

```
[student@workstation role-review]$ ansible-galaxy role install \
> -r roles/requirements.yml -p roles
```

Initialize a new role named `apache.developer_configs` in the `roles` subdirectory.

```
[student@workstation roles]$ ansible-galaxy role init apache.developer_configs
```

Add the `infra.apache` role as a dependency for the new role, using the same information for name, source, version, and version control system as the `roles/requirements.yml` file.

Modify the `roles/apache.developer_configs/meta/main.yml` file of the `apache.developer_configs` role to reflect a dependency on the `infra.apache` role.

```yaml
dependencies:
  - name: infra.apache
    src: git@workstation.lab.example.com:infra/apache
    scm: git
    version: v1.4
```

The `developer_tasks.yml` file in the project directory contains tasks for the role. Move this file to the correct location in the role directory hierarchy for a tasks file that is used by this role, replacing the existing file in that location.

The `developer.conf.j2` file in the project directory is a Jinja2 template that is used by the tasks file. Move this file to the correct location for template files that are used by this role.

```
[student@workstation role-review]$ mv -v developer_tasks.yml \
> roles/apache.developer_configs/tasks/main.yml

[student@workstation role-review]$ mv -v developer.conf.j2 \
> roles/apache.developer_configs/templates/
renamed 'developer.conf.j2' -> 'roles/apache.developer_configs/templates/developer.conf.j2'
```



Review the `web_developers.yml` file.

```yaml
---
web_developers:
  - username: jdoe
    name: John Doe
    user_port: 9081
  - username: jdoe2
    name: Jane Doe
    user_port: 9082
```

Place the `web_developers.yml` file in the `group_vars/dev_webserver` subdirectory.

```
[student@workstation role-review]$ mkdir -pv group_vars/dev_webserver

[student@workstation role-review]$ mv -v web_developers.yml \
> group_vars/dev_webserver/
```

`/home/student/role-review/web_dev_server.yml` playbook contains the following content:

```yaml
---
- name: Configure Dev Web Server
  hosts: dev_webserver
  force_handlers: true
  roles:
    - apache.developer_configs
```

run the playbook

```
[student@workstation role-review]$ ansible-navigator run \
> -m stdout web_dev_server.yml

...
RUNNING HANDLER [infra.apache : restart apache] ********************************
fatal: [servera.lab.example.com]: FAILED! => {"changed": false, "msg": "Unable to restart service httpd: Job for httpd.service failed because the control process exited with error code.\nSee \"systemctl status httpd.service\" and \"journalctl -xeu httpd.service\" for details.\n"}
...
An error occurs when the httpd service is restarted. The httpd service daemon cannot bind to the non-standard HTTP ports, due to the SELinux context on those ports.

```

Apache HTTPD failed to restart in the preceding step because the network ports it uses for your developers are labeled with the wrong SELinux contexts.

You can use the provided variable file, `selinux.yml`, with the `redhat.rhel_system_roles.selinux` role to fix the issue.

Create a `pre_tasks` section for your play in the `web_dev_server.yml` playbook. In that section, use a task to include the `redhat.rhel_system_roles.selinux` role in a `block`/`rescue` structure, so that it is properly applied.

Move the `selinux.yml` file to the correct location so that its variables are set for the `dev_webserver` host group.

```yaml
[student@workstation role-review]$ cat selinux.yml
---
# variables used by redhat.rhel_system_roles.selinux

selinux_policy: targeted
selinux_state: enforcing

selinux_ports:
  - ports:
      - "9081"
      - "9082"
    proto: 'tcp'
    setype: 'http_port_t'
    state: 'present'

[student@workstation role-review]$ mv -v selinux.yml \
> group_vars/dev_webserver/
renamed 'selinux.yml' -> 'group_vars/dev_webserver/selinux.yml'
```

```yaml
---
- name: Configure Dev Web Server
  hosts: dev_webserver
  force_handlers: true
  roles:
    - apache.developer_configs
  pre_tasks:
    - name: Verify SELinux configuration
      block:
        - name: Apply SELinux role
          ansible.builtin.include_role:
            name: redhat.rhel_system_roles.selinux
      rescue:
        # Fail if failed for a different reason than selinux_reboot_required.
        - name: Handle general failure
          ansible.builtin.fail:
            msg: "SELinux role failed."
          when: not selinux_reboot_required

        - name: Restart managed host
          ansible.builtin.reboot:
            msg: "Ansible rebooting system for updates."

        - name: Reapply SELinux role to complete changes
          ansible.builtin.include_role:
            name: redhat.rhel_system_roles.selinux
```

run again

```
[student@workstation role-review]$ ansible-navigator run \
> -m stdout web_dev_server.yml

...
PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=21   changed=2    unreachable=0    failed=0    skipped=16   rescued=0    ignored=0
```

Test the configuration of the development web server. Verify that all endpoints are accessible and serving each developer's content.

```
[student@workstation role-review]$ curl servera
This is the production server on servera.lab.example.com
[student@workstation role-review]$ curl servera:9081
This is index.html for user: John Doe (jdoe)
[student@workstation role-review]$ curl servera:9082
This is index.html for user: Jane Doe (jdoe2)
```



---

## Chapter 8. Troubleshooting Ansible

### Troubleshooting Playbooks

#### Debugging Playbooks

The output provided by the `ansible-navigator run` command is a good starting point for troubleshooting issues with your plays and the hosts on which they run.

You can increase the verbosity of the output by adding one or more `-v` options. The `ansible-navigator run -v` command provides additional debugging information, with up to four total levels.



**Table 8.1. Verbosity Configuration**

| Option  | Description                                                  |
| :------ | :----------------------------------------------------------- |
| `-v`    | The output data is displayed.                                |
| `-vv`   | Both the output and input data are displayed.                |
| `-vvv`  | Includes information about connections to managed hosts.     |
| `-vvvv` | Includes additional information, such as the scripts that are executed on each remote host, and the user that is executing each script. |

#### Examining Values of Variables with the Debug Module

You can use the `ansible.builtin.debug` module to provide insight into what is happening in the play. You can create a task that uses this module to display the value for a given variable at a specific point in the play. 

The following examples use the `msg` and `var` settings inside `ansible.builtin.debug` tasks. This first example displays the value at run time of the `ansible_facts['memfree_mb']` fact as part of a message printed to the output of `ansible-navigator run`.

```yaml
- name: Display free memory
  ansible.builtin.debug:
    msg: "Free memory for this system is {{ ansible_facts['memfree_mb'] }}"
```

This second example displays the value of the `output` variable.

```yaml
- name: Display the "output" variable
  ansible.builtin.debug:
    var: output
    verbosity: 2
```

The `verbosity` parameter controls when the `ansible.builtin.debug` module is executed. The value correlates to the number of `-v` options that are specified when the playbook is run. For example, if `-vv` is specified, and `verbosity` is set to `2` for a task, then that task is included in the debug output. **The default value of the `verbosity` parameter is `0`.**

#### Reviewing Playbooks for Errors

Several issues can occur during a playbook run, many related to the syntax of either the playbook or any of the templates it uses, or due to connectivity issues with the managed hosts (for example, an error in the host name of the managed host in the inventory file).

A number of tools are available that you can use to review your playbook for syntax errors and other problems before you run it.

##### Checking Playbook Syntax for Problems

The `ansible-navigator run` command accepts the `--syntax-check` option, which tests your playbook for syntax errors instead of actually running it.

It is a good practice to validate the syntax of your playbook before using it or if you are having problems with it.

```
[student@demo ~]$ ansible-navigator run \
> -m stdout playbook.yml --syntax-check
```

##### Checking a Given Task in a Playbook

You can use the `ansible-navigator run` command with the `--step` option to step through a playbook, one task at a time.

The `ansible-navigator run --step` command interactively prompts for confirmation that you want each task to run. Press **Y** to confirm that you want the task to run, **N** to skip the task, or **C** to continue running the remaining tasks.

```
[student@demo ~]$ ansible-navigator run \
> -m stdout playbook.yml --step --pae false

PLAY [Managing errors playbook] **********************************************
Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue:
```

**Because Ansible prompts you for input when you use the `--step` option, you must disable playbook artifacts and use standard output mode.**

You can also start running a playbook from a specific task by using the `--start-at-task` option. Provide the name of a task as an argument to the `ansible-navigator run --start-at-task` command.

For example, suppose that your playbook contains a task named `Ensure {{ web_service }} is started`. Use the following command to run the playbook starting at that task:

```
[student@demo ~]$ ansible-navigator run \
> -m stdout playbook.yml --start-at-task "Ensure {{ web_service }} is started"
```

You can use the `ansible-navigator run --list-tasks` command to list the task names in your playbook.

##### Checking Playbooks for Issues and Following Good Practices

One of the best ways to make it easier for you to debug playbooks is for you to follow good practices when writing them in the first place. Some recommended practices for playbook development include:

- Use a concise description of the play's or task's purpose to name plays and tasks. The play name or task name is displayed when the playbook is executed. This also helps document what each play or task is supposed to accomplish, and possibly why it is needed.
- Use comments to add additional inline documentation about tasks.
- Make effective use of vertical white space. In general, organize task attributes vertically to make them easier to read.
- Consistent horizontal indentation is critical. Use spaces, not tabs, to avoid indentation errors. Set up your text editor to insert spaces when you press the `Tab` key to make this easier.
- Try to keep the playbook as simple as possible. Only use the features that you need.

See [*Good Practices for Ansible* ](https://redhat-cop.github.io/automation-good-practices/).

To help you follow good practices like these, Red Hat Ansible Automation Platform 2 provides a tool, `ansible-lint`, that uses a set of predefined rules to look for possible issues with your playbook. Not all the issues that it reports break your playbook, but a reported issue might indicate the presence of a more serious error.

**Important**:

The `ansible-lint` command is a Technology Preview in Red Hat Ansible Automation Platform 2.2. Red Hat does not yet fully support this tool; for details, see the Knowledgebase article ["What does a "Technology Preview" feature mean?"](https://access.redhat.com/solutions/21101).

For example, assume that you have the following playbook, `site.yml`:

```yaml
- name: Configure servers with Ansible tools
  hosts: all  #(1)

  tasks:
    - name: Make sure tools are installed
      package: #(2)
        name:  #(3)
          - ansible-doc
          - ansible-navigator  #(4)
```

Run the `ansible-lint site.yml` command to validate it. You might get the following output as a result:

```
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
WARNING  Listing 4 violation(s) that are fatal
yaml: trailing spaces (trailing-spaces) 
site.yml:2

fqcn-builtins: Use FQCN for builtin actions. 
site.yml:5 Task/Handler: Make sure tools are installed

yaml: trailing spaces (trailing-spaces) 
site.yml:7

yaml: too many blank lines (1 > 0) (empty-lines) 
site.yml:10

You can skip specific rules or tags by adding them to your configuration file:
# .config/ansible-lint.yml
warn_list:  # or 'skip_list' to silence them completely
  - fqcn-builtins  # Use FQCN for builtin actions.
  - yaml  # Violations reported by yamllint.

Finished with 4 failure(s), 0 warning(s) on 1 files.
```

This run of `ansible-lint` found four style issues:

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#troubleshooting-ansible-playbooks-content-_checking_playbooks_for_issues_and_following_good_practices-CO13-1) | Line 2 of the playbook (`hosts: all`) apparently has trailing white space, detected by the `yaml` rule. It is not a problem with the playbook directly, but many developers prefer not to have trailing white space in files stored in version control to avoid unnecessary differences as files are edited. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#troubleshooting-ansible-playbooks-content-_checking_playbooks_for_issues_and_following_good_practices-CO13-2) | Line 5 of the playbook (`package:`) does not use a FQCN for the module name on that task. It should be `ansible.builtin.package:` instead. This was detected by the `fqcn-builtins` rule. |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#troubleshooting-ansible-playbooks-content-_checking_playbooks_for_issues_and_following_good_practices-CO13-3) | Line 7 of the playbook also apparently has trailing white space. |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#troubleshooting-ansible-playbooks-content-_checking_playbooks_for_issues_and_following_good_practices-CO13-4) | The playbook ends with one or more blank lines, detected by the `yaml` rule. |

The `ansible-lint` tool uses a local configuration file, **which is either the `.ansible-lint` or `.config/ansible-lint.yml` file in the current directory.** You can edit this configuration file to convert rule failures to warnings (by adding them as a list to the `warn_list` directive) or skip the checks entirely (by adding them as a list to the `skip_list` directive).

If you have a syntax error in the playbook, `ansible-lint` reports it just like `ansible-navigator run --syntax-check` does.

After you correct these style issues, the `ansible-lint site.yml` report is as follows:

```
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
```

This is an advisory message that you can ignore, and the lack of other output indicates that `ansible-lint` did not detect any other style issues.

For more information on `ansible-lint`, see https://docs.ansible.com/lint.html and the `ansible-lint --help` command.

**Important**

The `ansible-lint` command evaluates your playbook based on the software on your workstation. It does not use the automation execution environment container that is used by `ansible-navigator`.

The `ansible-navigator` command has an experimental `lint` option that runs `ansible-lint` in your automation execution environment, but the `ansible-lint` tool needs to be installed inside the automation execution environment's container image for the option to work. This is currently not the case with the default execution environment. You need a custom execution environment to run `ansible-navigator lint` at this time.

In addition, the version of `ansible-lint` provided with Red Hat Ansible Automation Platform 2.2 assumes that your playbooks are using Ansible Core 2.13, which is the version currently used by the default execution environment. It does not support earlier Ansible 2.9 playbooks.



#### Reviewing Playbook Artifacts and Log Files

Red Hat Ansible Automation Platform can log the output of playbook runs that you make from the command line in a number of different ways.

- `ansible-navigator` can produce *playbook artifacts* that store information about runs of playbooks in JSON format.
- You can log information about playbook runs to a text file in a location on the system to which you can write.

##### Playbook Artifacts from Automation Content Navigator

The `ansible-navigator` command produces playbook artifact files by default each time you use it to run a playbook. These files record information about the playbook run, and can be used to review the results of the run when it completes, to troubleshoot issues, or be kept for compliance purposes.

Each playbook artifact file is named based on the name of the playbook you ran, followed by the word `artifact`, and then the time stamp of when the playbook was run, ending with the `.json` file extension.

For example, if you run the command `ansible-navigator run site.yml` at 20:00 UTC on July 22, 2022, the resulting file name of the artifact file could be:

```
site-artifact-2022-07-22T20:00:04.019343+00:00.json
```

You can review the contents of these files with the `ansible-navigator replay` command. If you include the `-m stdout` option, then the output of the playbook run is printed to your terminal as if it had just run. However, if you omit that option, you can examine the results of the run interactively.

For example, you run the following playbook, `site.yml`, and it fails but you do not know why. You run `ansible-navigator run site.yml --syntax-check` and the `ansible-lint` command, but neither command reports any issues.

```yaml
- name: Configure servers with Ansible tools
  hosts: all

  tasks:
    - name: Make sure tools are installed
      ansible.builtin.package:
        name:
          - ansible-doc
          - ansible-navigator
```

To troubleshoot further, you run `ansible-navigator replay` in interactive mode on the resulting artifact file, which opens the following output in your terminal:

![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\playbook-replay-1.png)

Figure 8.1: Initial replay screen

If you enter `:0` to view the play, the following output is printed:

![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\playbook-replay-2.png)

Figure 8.2: Play results by machine and task

It looks like the task `Make sure tools are installed` failed on both the `server-1.example.com` and `server-2.example.com` hosts. By entering `:2`, you can look at the failure for the `server-2.example.com` host:

![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\playbook-replay-3.png)

Figure 8.3: Task results for a specific machine

The task is attempting to use the `ansible.builtin.package` module to install the `ansible-doc` package, and that package is not available in the RPM package repositories used by the `server-2.example.com` host, so the task failed. (You might discover that the `ansible-doc` command is now provided as part of the `ansible-navigator` RPM package as the `ansible-navigator doc` command, and changing the task accordingly fixes the problem.)

Another useful thing to know is that you can look at the results of a successful `Gathering Facts` task and the debugging output includes the values of all the facts that were gathered:

![img](C:\Users\meij1\OneDrive - Dell Technologies\Xmind\Durian+\TENSAI\resources\playbook-replay-4.png)

Figure 8.4: Task results for successful fact gathering

This can help you debug issues involving Ansible facts without adding a task to the play that uses the `ansible.builtin.debug` module to print out fact values.



You might not want to save playbook artifacts for several reasons.

- You are concerned about sensitive information being saved in the log file.
- You need to provide interactive input, such as a password, to `ansible-navigator` for some reason.
- You do not want the files to clutter up the project directory.

You can keep the files from being generated by creating an `ansible-navigator.yml` file in the project directory that disables the playbook artifacts:

```yaml
ansible-navigator:
  playbook-artifact:
    enable: false
```

##### Logging Output to a Text File

Ansible provides a built-in logging infrastructure that can be configured through the `log_path` parameter in the `default` section of the `ansible.cfg` configuration file, or through the `$ANSIBLE_LOG_PATH` environment variable. The environment variable takes precedence over the configuration file if both are configured. If a logging path is configured, then Ansible stores output from `ansible-navigator` commands as text in the specified file. This mechanism also works with earlier tools such as `ansible-playbook`.

If you configure Ansible to write log files to the `/var/log` directory, then Red Hat recommends that you configure `logrotate` to manage them.

#### References

[Configuring Ansible — Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html)

[ansible.builtin.debug module — Print statements during execution — Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/debug_module.html)

[Tips and tricks — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

[Good Practices for Ansible](https://redhat-cop.github.io/automation-good-practices/)

[Ansible Lint Documentation](https://docs.ansible.com/lint.html)



#### Example

Using the commands to Find the errors in the playbook

```yaml
[student@workstation troubleshoot-playbook]$ ll
total 12
-rw-r--r--. 1 student student   78 Sep 17 10:44 inventory
-rw-r--r--. 1 student student  517 Sep 17 10:44 samba.conf.j2
-rw-r--r--. 1 student student 1131 Sep 17 10:44 samba.yml
[student@workstation troubleshoot-playbook]$ cat inventory 
[samba_servers]
servera.lab.example.com

[mailrelay]
servera.lab.example.com


[student@workstation troubleshoot-playbook]$ cat samba.conf.j2 
# {{ random_var }}
    [global]
      workgroup = KAMANSI
      server string = Samba Server Version %v
      log file = /var/log/samba/log.%m
      max log size = 50
      security = user
      passdb backend = tdbsam
      load printers = yes
      cups options = raw
    [homes]
      comment = Home Directories
      browseable = no
      writable = yes
    [printers]
      comment = All Printers
      path = /var/spool/samba
      browseable = no
      guest ok = no
      writable = no
      printable = yes



# look carefully on the playbook, several errors are there!!!!
[student@workstation troubleshoot-playbook]$ cat samba.yml 
---
- name: Install a samba server
  hosts: samba_servers
  user: devops
  become: true
  vars:
    install_state: installed
    random_var: This is colon: test

  tasks:
    - name: Install samba
      ansible.builtin.dnf:
        name: samba
        state: {{ install_state }}

    - name: Install firewalld
      ansible.builtin.dnf:
        name: firewalld
        state: installed

    - name: Debug install_state variable
      ansible.builtin.debug:
        msg: "The state for the samba service is {{ install_state }}"

    - name: Start firewalld
      ansible.builtin.service:
        name: firewalld
        state: started
        enabled: true

    - name: Configure firewall for samba
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        service: samba

     - name: Deliver samba config
       ansible.builtin.template:
         src: samba.j2
         dest: /etc/samba/smb.conf
         owner: root
         group: root
         mode: 0644

    - name: Start samba
      ansible.builtin.service:
        name: smb
        state: started
        enabled: true
```

Create a file named `ansible.cfg` in the current directory. Configure the `log_path` parameter to write Ansible logs to the `/home/student/troubleshoot-playbook/ansible.log` file. Configure the `inventory` parameter to use the `/home/student/troubleshoot-playbook/inventory` file deployed by the lab script.

The completed `ansible.cfg` file should contain the following:

```
[defaults]
log_path = /home/student/troubleshoot-playbook/ansible.log
inventory = /home/student/troubleshoot-playbook/inventory
```

After fixing all the errors in the playbook:

```yaml
---
- name: Install a samba server
  hosts: samba_servers
  user: devops
  become: true
  vars:
    install_state: installed
    random_var: "This is colon: test"

  tasks:
    - name: Install samba
      ansible.builtin.dnf:
        name: samba
        state: "{{ install_state }}"

    - name: Install firewalld
      ansible.builtin.dnf:
        name: firewalld
        state: installed

    - name: Debug install_state variable
      ansible.builtin.debug:
        msg: "The state for the samba service is {{ install_state }}"

    - name: Start firewalld
      ansible.builtin.service:
        name: firewalld
        state: started
        enabled: true

    - name: Configure firewall for samba
      ansible.posix.firewalld:
        state: enabled
        permanent: true
        immediate: true
        service: samba

    - name: Deliver samba config
      ansible.builtin.template:
        src: samba.conf.j2
        dest: /etc/samba/smb.conf
        owner: root
        group: root
        mode: 0644

    - name: Start samba
      ansible.builtin.service:
        name: smb
        state: started
        enabled: true
```



### Troubleshooting Ansible Managed Hosts

#### Troubleshooting Connections

##### Problems Authenticating to Managed Hosts

 You could see similar "permission denied" errors in the following situations:

- You try to connect as the wrong `remote_user` for your authentication credentials
- You connect as the correct `remote_user` but the authentication credentials are missing or incorrect



For example, you might see the following output when running a playbook that is designed to connect to the remote `root` user account:

```
[student@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

PLAY [Install a samba server] **************************************************

TASK [Gathering Facts] *********************************************************
fatal: [host.lab.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: developer@host: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}

PLAY RECAP *********************************************************************
host.lab.example.com    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
Please review the log for errors.
```

In this case, `ansible-navigator` is trying to connect as the `developer` user account, according to the preceding output. One reason this might happen is if `ansible.cfg` has been configured in the project to set the `remote_user` to the `developer` user instead of the `root` user.

Another reason you could see a "permission denied" error like this is if you do not have the correct SSH keys set up, or did not provide the correct password for that user.

```
[root@controlnode ~]# ansible-navigator run \
> -m stdout playbook.yml

TASK [Gathering Facts] *********************************************************
fatal: [host.lab.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: root@host: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}

PLAY RECAP *********************************************************************
host    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0

Please review the log for errors.
```

In the preceding example, the playbook is attempting to connect to the `host` machine as the `root` user but the SSH key for the `root` user on the `controlnode` machine has not been added to the `authorized_keys` file for the `root` user on the `host` machine.



##### Problems with Name or Address Resolution

A more subtle problem has to do with inventory settings. For a complex server with multiple network addresses, you might need to use a particular address or DNS name when connecting to that system. You might not want to use that address as the machine's inventory name for better readability. You can set a host inventory variable, `ansible_host`, that overrides the inventory name with a different name or IP address and be used by Ansible to connect to that host. This variable could be set in the `host_vars` file or directory for that host, or could be set in the inventory file itself.

For example, the following inventory entry configures Ansible to connect to `192.0.2.4` when processing the `web4.phx.example.com` host:

```
web4.phx.example.com ansible_host=192.0.2.4
```

This is a useful way to control how Ansible connects to managed hosts. However, it can also cause problems if the value of `ansible_host` is incorrect.



##### Problems with Privilege Escalation

If your playbook connects as a `remote_user` and then uses privilege escalation to become the `root` user (or some other user), make sure that `become` is set properly, and that you are using the correct value for the `become_user` directive. **The setting for `become_user` is `root` by default.**

If the remote user needs to provide a `sudo` password, you should confirm that you are providing the correct `sudo` password, and that `sudo` on the managed host is configured correctly.

```
[user@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

TASK [Gathering Facts] *********************************************************
fatal: [host]: FAILED! => {"msg": "Missing sudo password"}

PLAY RECAP *********************************************************************
host             : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

Please review the log for errors.
```

In the preceding example, the playbook is attempting to run `sudo` on the `host` machine but it fails. The `remote_user` is not set up to run `sudo` commands without a password on the `host` machine. Either `sudo` on the `host` machine is not properly configured, or it is supposed to require a `sudo` password and you neglected to provide one when running the playbook.

**Important**:

Normally, `ansible-navigator` runs as `root` inside its automation execution environment. However, the `root` user in the container has access to SSH keys provided by the user that ran `ansible-navigator` on the workstation. This can be slightly confusing when you are trying to debug `remote_user` and `become` directives, especially if you are used to the earlier `ansible-playbook` command that runs as the user on the workstation.



##### Problems with Python on Managed Hosts

For normal operation, Ansible requires a Python interpreter to be installed on managed hosts running Linux. Ansible attempts to locate a Python interpreter on each Linux managed host the first time a module is run on that host.

```
[user@controlnode ~]$ ansible-navigator run \
> -m stdout playbook.yml

TASK [Gathering Facts] *********************************************************
fatal: [host]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"ansible.legacy.setup": {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "failed": true, "module_stderr": "Shared connection to host closed.\r\n", "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127, "warnings": ["No python interpreters found for host host (tried ['python3.10', 'python3.9', 'python3.8', 'python3.7', 'python3.6', 'python3.5', '/usr/bin/python3', '/usr/libexec/platform-python', 'python2.7', 'python2.6', '/usr/bin/python', 'python'])"]}}, "msg": "The following modules failed to execute: ansible.legacy.setup\n"}

PLAY RECAP *********************************************************************
host    : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
Please review the log for errors.
```



#### Using Check Mode as a Testing Tool

You can use the `ansible-navigator run --check` command to run "smoke tests" on a playbook. This option runs the playbook, connecting to the managed hosts normally but without making changes to them.

If a module used within the playbook supports "check mode", then the changes that would have been made to the managed hosts are displayed but not performed. If check mode is not supported by a module, then `ansible-navigator` does not display the predicted changes, but the module still takes no action.

```
[student@demo ~]$ ansible-navigator run \
> -m stdout playbook.yml --check
```

**Important:**

The `ansible-navigator run --check` command might not work properly if your tasks use conditionals. One reason for this might be that the conditionals depend on some preceding task in the play actually running so that the condition evaluates correctly.

You can force tasks to always run in check mode or to always run normally with the `check_mode` setting. **If a task has `check_mode: true` set, it always runs in its check mode and does not perform any action, even if you do not pass the `--check` option to `ansible-navigator`. Likewise, if a task has `check_mode: false` set, it always runs normally, even if you pass `--check` to `ansible-navigator`.**

The following task always runs in check mode, and does not make changes to managed hosts.

```yaml
  tasks:
    - name: task always in check mode
      ansible.builtin.shell: uname -a
      check_mode: true
```

The following task always runs normally, even when started with `ansible-navigator run --check`.

```yaml
  tasks:
    - name: task always runs even in check mode
      ansible.builtin.shell: uname -a
      check_mode: false
```

This can be useful because you can run most of a playbook normally and test individual tasks with `check_mode: true`. Many plays use facts or set variables to conditionally run tasks. Conditional tasks might fail if a fact or variable is undefined, due to the task that collects them or sets them not executing on a managed node. You can use `check_mode: false` on tasks that gather facts or set variables but do not otherwise change the managed node. This enables the play to proceed further when using `--check` mode.

A task can determine if the playbook is running in check mode by testing the value of the magic variable `ansible_check_mode`. This Boolean variable is set to `true` if the playbook is running in check mode.

**Warning**:

Tasks that have `check_mode: false` set run even when the playbook is run with `ansible-navigator run --check`. Therefore, you cannot trust that the `--check` option makes no changes to managed hosts, without inspecting the playbook and any roles or tasks associated with it.



Note:

If you have older playbooks that use `always_run: true` to force tasks to run normally even in check mode, you need to replace that code with `check_mode: false` in Ansible 2.6 and later.



The `ansible-navigator` command also provides a `--diff` option. This option reports the changes made to the template files on managed hosts. If used with the `--check` option, those changes are displayed in the command's output but not actually made.

```
[student@demo ~]$ ansible-navigator run \
> -m stdout playbook.yml --check --diff
```



#### Testing with Modules

Some modules can provide additional information about the status of a managed host. The following list includes some Ansible modules that can be used to test and debug issues on managed hosts.

The `ansible.builtin.uri` module provides a way to verify that a RESTful API is returning the required content.

```yaml
  tasks:
    - ansible.builtin.uri:
        url: http://api.myapp.example.com
        return_content: true
      register: apiresponse

    - ansible.builtin.fail:
        msg: 'version was not provided'
      when: "'version' not in apiresponse.content"
```

The `ansible.builtin.script` module runs a script on managed hosts, and fails if the return code for that script is nonzero. The script must exist in the Ansible project and is transferred to and run on the managed hosts.

```
  tasks:
    - ansible.builtin.script: scripts/check_free_memory --min 2G
```

The `ansible.builtin.stat` module gathers facts for a file much like the `stat` command. You can use it to register a variable and then test to determine if the file exists or to get other information about the file. If the file does not exist, the `ansible.builtin.stat` task does not fail, but its registered variable reports `false` for `*['stat']['exists']`.

In this example, an application is still running if `/var/run/app.lock` exists, in which case the play should abort.

```yaml
  tasks:
    - name: Check if /var/run/app.lock exists
      ansible.builtin.stat:
        path: /var/run/app.lock
      register: lock

    - name: Fail if the application is running
      ansible.builtin.fail:
      when: lock['stat']['exists']
```

The `ansible.builtin.assert` module is an alternative to the `ansible.builtin.fail` module. The `ansible.builtin.assert` module supports a `that` option that takes a list of conditionals. **If any of those conditionals are false, the task fails.** You can use the `success_msg` and `fail_msg` options to customize the message it prints if it reports success or failure.

The following example repeats the preceding one, but uses `ansible.builtin.assert` instead of the `ansible.builtin.fail` module:

```yaml
  tasks:
    - name: Check if /var/run/app.lock exists
      ansible.builtin.stat:
        path: /var/run/app.lock
      register: lock

    - name: Fail if the application is running
      ansible.builtin.assert:
        that:
          - not lock['stat']['exists']
```



#### Running Ad Hoc Commands with Ansible

An ad hoc command is a way of executing a single Ansible task quickly, one that you do not need to save to run again later. They are simple, online operations that can be run without writing a playbook.

**Ad hoc commands do not run inside an automation execution environment container.** Instead, they run using Ansible software, roles, and collections installed directly on your workstation. To use ad hoc Ansible Core 2.13 commands, you need to install the `ansible-core` RPM package on your workstation.



Use the `ansible` command to run ad hoc commands:

```
[user@controlnode ~]$ ansible host-pattern -m module [-a 'module arguments'] \
> [-i inventory]
```

The `*`host-pattern`*` argument is used to specify the managed hosts against which the ad hoc command should be run. The `-i` option is used to specify a different inventory location to use from the default in the current Ansible configuration file. The `-m` option specifies the module that Ansible should run on the targeted hosts. The `-a` option takes a list of arguments for the module as a quoted string.

**Note**:

If you use the `ansible` command but do not specify a module with the `-m` option, the `ansible.builtin.command` module is used by default. It is always best to specify the module you intend to use, even if you intend to use the `ansible.builtin.command` module.

Ansible ad hoc commands can be useful, but should be kept to troubleshooting and one-time use cases. For example, if you are aware of multiple pending network changes, it is more efficient to create a playbook with an `ansible.builtin.ping` task that you can run multiple times, compared to typing out a one-time use ad hoc command multiple times.



##### Testing Managed Hosts Using Ad Hoc Commands

```
$ ansible [pattern] -m [module] -a "[module options]"
```

The `-a` option accepts options either through the `key=value` syntax or a JSON string starting with `{` and ending with `}` for more complex option structure. You can learn more about [patterns](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html#intro-patterns) and [modules](https://docs.ansible.com/ansible/latest/plugins/module.html#module-plugins) on other pages

The following examples illustrate some tests that can be made on a managed host using ad hoc commands.

You have used the `ansible.builtin.ping` module to test whether you can connect to managed hosts. Depending on the options that you pass, you can also use it to test whether privilege escalation and credentials are correctly configured.

```
[student@demo ~]$ ansible demohost -m ansible.builtin.ping
demohost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[student@demo ~]$ ansible demohost -m ansible.builtin.ping --become
demohost | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "module_stderr": "sudo: a password is required\n",
    "module_stdout": "",
    "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error",
    "rc": 1
}
```

This example returns the current available space on the disks configured on the `demohost` managed host. That can be useful to confirm that the file system on the managed host is not full.

```
[student@demo ~]$ ansible demohost -m ansible.builtin.command -a 'df'
```

This example returns the current available free memory on the `demohost` managed host.

```
[student@demo ~]$ ansible demohost -m ansible.builtin.command -a 'free -m'
```

```yaml
# Rebooting servers
$ ansible atlanta -a "/sbin/reboot"
# Rebooting probably requires privilege escalation. You can connect to the server as username and run the command as the root user by using the become keyword:
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]

# Managing files
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
# create directory
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
# delete files
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c state=absent"

#Managing packages
$ ansible webservers -m ansible.builtin.yum -a "name=acme-1.5 state=present/latest/absent"

#Managing users and groups
$ ansible all -m ansible.builtin.user -a "name=foo password=<encrypted password here>"
$ ansible all -m ansible.builtin.user -a "name=foo state=absent"

# Managing services
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=started/restarted/stopped"

# Gathering facts
$ ansible all -m ansible.builtin.setup

# Check mode
$  ansible all -m copy -a "content=foo dest=/root/bar.txt" -C
# Enabling check mode (-C or --check) in the above command means Ansible does not actually create or update the /root/bar.txt file on any remote systems

# remote at webservers running 'systemctl status httpd'with user devops and use -b（or --become）to get root privelege
[student@workstation troubleshoot-review]$ ansible webservers -u devops -b \
> -m command -a 'systemctl status httpd'
```

#### References

[Check Mode ("Dry Run") — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html)

[Testing Strategies — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/test_strategies.html)

#### Example

1. Run the `samba.yml` playbook. The first task fails with an error related to an SSH connection problem.

   ```yaml
   # samba.yml
   ---
   - name: Install a samba server
     hosts: samba_servers
     user: devops
     become: true
     vars:
       install_state: installed
       random_var: "This is colon: test"
   
     tasks:
       - name: Install samba
         ansible.builtin.dnf:
           name: samba
           state: "{{ install_state }}"
   
       - name: Install firewalld
         ansible.builtin.dnf:
           name: firewalld
           state: installed
   
       - name: Debug install_state variable
         ansible.builtin.debug:
           msg: "The state for the samba service is {{ install_state }}"
   
       - name: Start firewalld
         ansible.builtin.service:
           name: firewalld
           state: started
           enabled: true
   
       - name: Configure firewall for samba
         ansible.posix.firewalld:
           state: enabled
           permanent: true
           immediate: true
           service: samba
   
       - name: Deliver samba config
         ansible.builtin.template:
           src: samba.conf.j2
           dest: /etc/samba/smb.conf
           owner: root
           group: root
           mode: 0644
   
       - name: Start samba
         ansible.builtin.service:
           name: smb
           state: started
           enabled: true
   ```

   

   ```
   [student@workstation troubleshoot-host]$ ansible-navigator run \
   > -m stdout samba.yml
   
   PLAY [Install a samba server] **************************************************
   
   TASK [Gathering Facts] *********************************************************
   fatal: [servera.lab.exammple.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host servera.lab.exammple.com port 22: Connection timed out", "unreachable": true}
   
   PLAY RECAP *********************************************************************
   servera.lab.exammple.com   : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
   Please review the log for errors.
   ```

2. Make sure that you can connect to the `servera.lab.example.com` managed host as the `devops` user using SSH, and that the correct SSH keys are in place. Log off again when you have finished.

   ```
   [student@workstation troubleshoot-host]$ ssh devops@servera.lab.example.com
   ...output omitted...
   [devops@servera ~]$ exit
   logout
   Connection to servera.lab.example.com closed.
   ```

   That is working normally.

3. Test to see if you can run modules on the `servera.lab.example.com` managed host by using an ad hoc command that runs the `ansible.builtin.ping` module.

   ```
   [student@workstation troubleshoot-host]$ ansible servera.lab.example.com \
   > -m ansible.builtin.ping
   servera.lab.example.com | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   ```

   Based on the preceding output, that is also working, and successfully connected to the managed host.

   This should suggest to you that the problem is not with the SSH configuration and credentials, or with the ad hoc command that you used. So the question now is why the ad hoc command worked and the `ansible-navigator` command did not. There might be a problem with the play in the playbook, or with the inventory.

4. Rerun the `samba.yml` playbook with `-vvvv` to get more information about the run. An error is issued because the `servera.lab.example.com` managed host is not reachable.

   ```
   [student@workstation troubleshoot-host]$ ansible-navigator run \
   > -m stdout -vvvv samba.yml
   ansible-playbook [core 2.13.0]
   ...output omitted...
   
   PLAYBOOK: samba.yml ************************************************************
   Positional arguments: /home/student/troubleshoot-host/samba.yml
   verbosity: 4
   connection: smart
   timeout: 10
   become_method: sudo
   tags: ('all',)
   inventory: ('/home/student/troubleshoot-host/inventory',)
   forks: 5
   1 plays in /home/student/troubleshoot-host/samba.yml
   
   PLAY [Install a samba server] **************************************************
   
   TASK [Gathering Facts] *********************************************************
   task path: /home/student/troubleshoot-host/samba.yml:2
   <servera.lab.exammple.com> ESTABLISH SSH CONNECTION FOR USER: devops
   ...output omitted...
   fatal: [servera.lab.exammple.com]: UNREACHABLE! => {
       "changed": false,
       "msg": "Failed to connect to the host via ssh: OpenSSH_8.0p1, OpenSSL 1.1.1k  FIPS 25 Mar 2021\r\ndebug1: Reading configuration data /home/runner/.ssh/config\r\ndebug1: /home/runner/.ssh/config line 1: Applying options for *\r\ndebug1: Reading configuration data /etc/ssh/ssh_config\r\ndebug3: /etc/ssh/ssh_config line 52: Including file /etc/ssh/ssh_config.d/05-redhat.conf depth 0\r\ndebug1: 
   ....omitted.....
   debug1: Connecting to servera.lab.exammple.com [3.130.253.23] port 22.\r\ndebug2: fd 3 setting O_NONBLOCK\r\ndebug1: connect to address 3.130.253.23 port 22: Connection timed out\r\ndebug1: Connecting to servera.lab.exammple.com [3.130.204.160] port 22.\r\ndebug2: fd 3 setting O_NONBLOCK\r\ndebug1: connect to address 3.130.204.160 port 22: Connection timed out\r\nssh: connect to host servera.lab.exammple.com port 22: Connection timed out",
       "unreachable": true
   }
   
   PLAY RECAP *********************************************************************
   servera.lab.exammple.com   : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
   Please review the log for errors.
   ```

5. Investigate the `inventory` file for errors.

   If you look at the `[samba_servers]` group, `servera.lab.example.com` is misspelled (with an extra `m`). Correct this error as shown below:

   ```
   [student@workstation troubleshoot-host]$ cat inventory 
   [samba_servers]
   servera.lab.exammple.com   # bad here
   
   [mailrelay]
   servera.lab.example.com
   
   # ==> changed to
   [samba_servers]
   servera.lab.example.com
   ...output omitted...
   ```

6. Run the playbook again and all tasks should succeed.



### Chapter 8 Example

**Instructions**

In the `/home/student/troubleshoot-review` directory, there is a playbook named `secure-web.yml`. This playbook contains one play that is supposed to set up Apache HTTPD with TLS/SSL for hosts in the `webservers` group. The `serverb.lab.example.com` node is supposed to be the only host in the `webservers` group right now. Ansible can connect to that host using the remote `devops` account and SSH keys that have already been set up. That user can also become `root` on the managed host without a `sudo` password.

Unfortunately, several problems exist that you need to fix before you can run the playbook successfully.

```yaml
[student@workstation troubleshoot-review]$ ll
total 20
-rw-r--r--. 1 student student   33 Sep 20 10:15 ansible.cfg
-rw-r--r--. 1 student student   21 Sep 20 10:15 index.html
-rw-r--r--. 1 student student   74 Sep 20 10:15 inventory
-rw-r--r--. 1 student student 2674 Sep 20 10:15 secure-web.yml
-rw-r--r--. 1 student student  604 Sep 20 10:15 vhosts.conf

[student@workstation troubleshoot-review]$ cat ansible.cfg 
[defaults]
inventory = inventory

[student@workstation troubleshoot-review]$ cat index.html 
This is a test page.

[student@workstation troubleshoot-review]$ cat inventory 
[webservers]
serverb.lab.example.com ansible_host=serverc.lab.example.com

[student@workstation troubleshoot-review]$ cat secure-web.yml 
---
# start of secure web server playbook
- name: Create secure web service
  hosts: webservers
  remote_user: students
  vars:
    random_var: This is colon: test
    rule:
      - http
      - https

  tasks:
    - block:
        - name: Install web server packages
          ansible.builtin.dnf:
            name: {{ item }}
            state: latest
          notify:
            - Restart services
          loop:
            - httpd
            - mod_ssl

        - name: Install httpd config files
          ansible.builtin.copy:
            src: vhosts.conf
            dest: /etc/httpd/conf.d/vhosts.conf
            backup: true
            owner: root
            group: root
            mode: 0644
          register: vhosts_config
          notify:
            - Restart services

        - name: Create ssl certificate
          ansible.builtin.command: openssl req -new -nodes -x509 -subj "/C=US/ST=North Carolina/L=Raleigh/O=Example Inc/CN=serverb.lab.example.com" -days 120 -keyout /etc/pki/tls/private/serverb.lab.example.com.key -out /etc/pki/tls/certs/serverb.lab.example.com.crt -extensions v3_ca
          args:
            creates: /etc/pki/tls/certs/serverb.lab.example.com.crt

         - name: Start and enable web services
           ansible.builtin.service:
             name: httpd
             state: started
             enabled: true

        - name: Open ports for http and https
          ansible.posix.firewalld:
            service: "{{ item }}"
            immediate: true
            permanent: true
            state: enabled
          loop: "{{ rule }}"

        - name: Deliver content
          ansible.builtin.copy:
            dest: /var/www/vhosts/serverb-secure/
            src: index.html

        - name: Check httpd syntax
          ansible.builtin.command: /sbin/httpd -t
          register: httpd_conf_syntax
          failed_when: "'Syntax OK' not in httpd_conf_syntax.stderr"

        - name: Httpd_conf_syntax variable
          ansible.builtin.debug:
            msg: "The httpd_conf_syntax variable value is {{ httpd_conf_syntax }}"

        - name: Check httpd status
          ansible.builtin.command: systemctl is-active httpd
          register: httpd_status
          changed_when: httpd_status.rc != 0
          notify:
            - Restart services

      rescue:
        - name: Recover original httpd config
          ansible.builtin.file:
            path: /etc/httpd/conf.d/vhosts.conf
            state: absent
          notify:
            - Restart services

  handlers:
    - name: Restart services
      ansible.builtin.service:
        name: httpd
        state: restarted

# end of secure web play
[student@workstation troubleshoot-review]$ 
[student@workstation troubleshoot-review]$ cat vhosts.conf 
<VirtualHost serverb.lab.example.com>
    ServerAdmin webmaster@foob.example.com
    ServerName serverb.lab.example.com
    ErrorLog logs/serverb-ssl.error.log
    CustomLog logs/serverb-secure.common.log common
    DocumentRoot /var/www/vhosts/serverb-secure/

    SSLEngine On
    SSLCertificateFile /etc/pki/tls/certs/serverb.lab.example.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/serverb.lab.example.com.key

    <Directory /var/www/vhosts/serverb-secure>
        Options +Indexes +followsymlinks +includes
        Order allow,deny
        allow from all
    </Directory>
</VirtualHost>

[student@workstation troubleshoot-review]$ 
```

many errors will show, after correcting all errors:

the yml would be:

```yaml
[student@workstation troubleshoot-review]$ cat inventory 
[webservers]
serverb.lab.example.com 


[student@workstation troubleshoot-review]$ cat secure-web.yml 
---
# start of secure web server playbook
- name: Create secure web service
  hosts: webservers
  remote_user: devops
  become: true
  vars:
    random_var: "This is colon: test"
    rule:
      - http
      - https

  tasks:
    - block:
        - name: Install web server packages
          ansible.builtin.dnf:
            name: "{{ item }}"
            state: latest
          notify:
            - Restart services
          loop:
            - httpd
            - mod_ssl

        - name: Install httpd config files
          ansible.builtin.copy:
            src: vhosts.conf
            dest: /etc/httpd/conf.d/vhosts.conf
            backup: true
            owner: root
            group: root
            mode: 0644
          register: vhosts_config
          notify:
            - Restart services

        - name: Create ssl certificate
          ansible.builtin.command: openssl req -new -nodes -x509 -subj "/C=US/ST=North Carolina/L=Raleigh/O=Example Inc/CN=serverb.lab.example.com" -days 120 -keyout /etc/pki/tls/private/serverb.lab.example.com.key -out /etc/pki/tls/certs/serverb.lab.example.com.crt -extensions v3_ca
          args:
            creates: /etc/pki/tls/certs/serverb.lab.example.com.crt

        - name: Start and enable web services
          ansible.builtin.service:
            name: httpd
            state: started
            enabled: true

        - name: Open ports for http and https
          ansible.posix.firewalld:
            service: "{{ item }}"
            immediate: true
            permanent: true
            state: enabled
          loop: "{{ rule }}"

        - name: Deliver content
          ansible.builtin.copy:
            dest: /var/www/vhosts/serverb-secure/
            src: index.html

        - name: Check httpd syntax
          ansible.builtin.command: /sbin/httpd -t
          register: httpd_conf_syntax
          failed_when: "'Syntax OK' not in httpd_conf_syntax.stderr"

        - name: Httpd_conf_syntax variable
          ansible.builtin.debug:
            msg: "The httpd_conf_syntax variable value is {{ httpd_conf_syntax }}"

        - name: Check httpd status
          ansible.builtin.command: systemctl is-active httpd
          register: httpd_status
          changed_when: httpd_status.rc != 0
          notify:
            - Restart services

      rescue:
        - name: Recover original httpd config
          ansible.builtin.file:
            path: /etc/httpd/conf.d/vhosts.conf
            state: absent
          notify:
            - Restart services

  handlers:
    - name: Restart services
      ansible.builtin.service:
        name: httpd
        state: restarted

# end of secure web play
```

---

## Chapter 9 Automating Linux Administration Tasks

### Managing Software and Subscriptions

#### Managing Packages with Ansible

The `ansible.builtin.dnf` Ansible module uses `dnf` on the managed hosts to handle package operations. The following playbook installs the `httpd` package on the `servera.lab.example.com` managed host:

```yaml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the httpd packages
      ansible.builtin.dnf:
        name: httpd    
        state: present 
 
# The `state` keyword indicates the expected state of the package on the managed host:`present`Ansible installs the package if it is not already installed.`absent`Ansible removes the package if it is installed.`latest`Ansible updates the package if it is not al
```

The following table compares some uses of the `ansible.builtin.dnf` Ansible module with the equivalent `dnf` command.

| Ansible task                                                 | DNF command                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `- name: Install httpd  ansible.builtin.dnf:    name: httpd    state: present` | `dnf install httpd`                                          |
| `- name: Install or upgrade httpd  ansible.builtin.dnf:    name: httpd    state: latest` | `dnf upgrade httpd` or `dnf install httpd` if the package is not yet installed. |
| `- name: Upgrade all packages  ansible.builtin.dnf:    name: '*'    state: latest` | `dnf upgrade`                                                |
| `- name: Remove httpd  ansible.builtin.dnf:    name: httpd    state: absent` | `dnf remove httpd`                                           |
| `- name: Install Development Tools  ansible.builtin.dnf:    name: '@Development Tools' state: present` With the `ansible.builtin.dnf` Ansible module, you must prefix group names with the at sign (`@`). Remember that you can retrieve the list of groups with the `dnf group list` command. | `dnf group install "Development Tools"`                      |
| `- name: Remove Development Tools  ansible.builtin.dnf:    name: '@Development Tools'    state: absent` | `dnf group remove "Development Tools"`                       |
| `- name: Install perl DNF module  ansible.builtin.dnf:    name: '@perl:5.26/minimal' state: present`To manage a package module, prefix its name with the at sign (`@`). The syntax is the same as with the `dnf` command. For example, you can omit the profile part to use the default profile: `@perl:5.26`. Remember that you can list the available package modules with the `dnf module list` command. | `dnf module install perl:5.26/minimal`.                      |

Run the `ansible-navigator doc ansible.builtin.dnf` command for additional parameters and playbook examples.

##### Optimizing Multiple Package Installation

To operate on several packages, the `name` keyword accepts a list. The following playbook installs three packages on the `servera.lab.example.com` managed host.

```yaml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the packages
      ansible.builtin.dnf:
        name:
          - httpd
          - mod_ssl
          - httpd-tools
        state: present
```

With this syntax, Ansible installs the packages in a single DNF transaction. This is equivalent to running the `dnf install httpd mod_ssl httpd-tools` command.

A commonly seen but **less efficient** and slower version of this task is to use a loop.

```yaml
---
- name: Install the required packages on the web server
  hosts: servera.lab.example.com
  tasks:
    - name: Install the packages
      ansible.builtin.dnf:
        name: "{{ item }}""
        state: present
      loop:
        - httpd
        - mod_ssl
        - httpd-tools
```

**Avoid using this method because it requires the module to perform three individual transactions; one for each package.**

##### Gathering Facts about Installed Packages

The `ansible.builtin.package_facts` Ansible module collects the installed package details on managed hosts. It sets the `ansible_facts['packages']` variable with the package details.

The following playbook calls the `ansible.builtin.package_facts` module, and the `ansible.builtin.debug` module to display the content of the `ansible_facts['packages']` variable and the version of the installed `NetworkManager` package.

```yaml
---
- name: Display installed packages
  hosts: servera.lab.example.com
  gather_facts: false
  tasks:
    - name: Gather info on installed packages
      ansible.builtin.package_facts:
        manager: auto

    - name: List installed packages
      ansible.builtin.debug:
        var: ansible_facts['packages']

    - name: Display NetworkManager version
      ansible.builtin.debug:
        var: ansible_facts['packages']['NetworkManager'][0]['version'] 
      when: ansible_facts['packages']['NetworkManager'] is defined
```

When run, the playbook displays the package list and the version of the `NetworkManager` package:

```
[user@controlnode ~]$ ansible-navigator run -m stdout lspackages.yml

PLAY [Display installed packages] **********************************************

TASK [Gather info on installed packages] ***************************************
ok: [servera.lab.example.com]

TASK [List installed packages] *************************************************
ok: [servera.lab.example.com] => {
    "ansible_facts['packages']": {
        "NetworkManager": [
            {
                "arch": "x86_64",
                "epoch": 1,
                "name": "NetworkManager",
                "release": "4.el9_0",
                "source": "rpm",
                "version": "1.36.0"
            }
        ],
...output omitted...
        "zstd": [
            {
                "arch": "x86_64",
                "epoch": null,
                "name": "zstd",
                "release": "2.el9",
                "source": "rpm",
                "version": "1.5.1"
            }
        ]
    }
}

TASK [Display NetworkManager version] ******************************************
ok: [servera.lab.example.com] => {
    "ansible_facts['packages']['NetworkManager'][0]['version']": "1.36.0"
}

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=3    changed=0    unreachable=0    failed=0  ...
```

##### Reviewing Alternative Modules to Manage Packages

For other package managers, Ansible usually provides a dedicated module. The `ansible.builtin.apt` module uses the APT package tool available on Debian and Ubuntu. The `ansible.windows.win_package` module can install software on Microsoft Windows systems.

The following playbook uses conditionals to select the appropriate module in an environment composed of Red Hat Enterprise Linux systems running major versions 7, 8, and 9.

```yaml
---
- name: Install the required packages on the web servers
  hosts: webservers
  tasks:
    - name: Install httpd on RHEL 8 and 9
      ansible.builtin.dnf:
        name: httpd
        state: present
      when:
        - "ansible_facts['distribution'] == 'RedHat'"
        - "ansible_facts['distribution_major_version'] >= '8'`"

    - name: Install httpd on RHEL 7 and earlier
      `ansible.builtin.yum:
        name: httpd
        state: present
      when:
        - "ansible_facts['distribution'] == 'RedHat'"
        - "ansible_facts['distribution_major_version'] <= `'7'`"
```

As an alternative, the generic `ansible.builtin.package` module automatically detects and uses the package manager available on the managed hosts. With the `ansible.builtin.package` module, you can rewrite the previous playbook as follows:

```yaml
---
- name: Install the required packages on the web servers
  hosts: webservers
  tasks:
    - name: Install httpd
      ansible.builtin.package:
        name: httpd
        state: present
```

However, the `ansible.builtin.package` module does not support all the features that the more specialized modules provide.

Also, operating systems often have different names for the packages they provide. For example, the package that installs the Apache HTTP Server is `httpd` on Red Hat Enterprise Linux and `apache2` on Ubuntu.

In that situation, you still need a conditional for selecting the correct package name depending on the operating system of the managed host.

#### Registering and Managing Systems with Red Hat Subscription Management

You can entitle your Red Hat Enterprise Linux systems to product subscriptions by using a few different methods:

- You can use the `subscription-manager` command.

- On Red Hat Enterprise Linux 9.2 systems and later, you can use the `rhel-system-roles.rhc` role available from the `rhel-system-roles` RPM (version 1.21.1 and later).

- You can use the `redhat.rhel_system_roles.rhc` role from the `redhat.rhel_system_roles` collection (version 1.21.1 and later).

  

  The `registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel8` automation execution environment contains the `redhat.rhel_system_roles` collection. You can also install the `redhat.rhel_system_roles` collection in your Ansible project and then use the `ee-supported-rhel8` automation execution environment available for Ansible Automation Platform 2.2 or 2.3.

##### Managing Red Hat Subscription Management from the Command Line

Without Ansible, you can use the `subscription-manager` command to register a system:

```
[user@host ~]$ sudo subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: yourusername
Password: yourpassword
...output omitted...
```

The following command attaches a subscription using a pool ID. You can list the pools available to your account with the `subscription-manager list --available` command.

```
[user@host ~]$ sudo subscription-manager attach --pool=poolID
```

After you register a system and attach a subscription, you can use the `subscription-manager` command to enable Red Hat software repositories on the system. You might use the `subscription-manager repos --list` command to identify available repositories and then use the `subscription-manager repos enable` command to enable repositories:

```
[user@host ~]$ sudo subscription-manager repos \
> --enable "rhel-9-for-x86_64-baseos-rpms" \
> --enable "rhel-9-for-x86_64-appstream-rpms"
```

##### Managing Red Hat Subscription Management by Using a Role

Whether you use the `rhel-system-roles.rhc` role from the `rhel-system-roles` RPM or the `redhat.rhel_system_roles.rhc` role from the `redhat.rhel_system_roles` collection, the steps for managing Red Hat subscription management are essentially the same.

1. Create a play that includes the desired role:

   ```yaml
   ---
   - name: Register systems
     hosts: all
     become: true
     tasks:
       - name: Include the rhc role
         ansible.builtin.include_role:
           name: redhat.rhel_system_roles.rhc
   ```

2. Define variables for the role. You might define these variables in the play, in a `group_vars` directory, in a `host_vars` directory, or in a separate variable file:

   ```yaml
   ---
   rhc_state: present 
   rhc_auth: 
     login:
       username: yourusername
       password: yourpassword
   rhc_insights: 
     state: present
   rhc_repositories: 
     - name: rhel-9-for-x86_64-baseos-rpms
       state: enabled
     - name: rhel-9-for-x86_64-appstream-rpms
       state: enabled
   ```

   | The `rhc_state` variable specifies if the system should be connected (or registered) to Red Hat. Valid values are `present`, `absent`, and `reconnect`. When set to either `present` or `reconnect`, the role attempts to automatically attach a subscription. |
   | :----------------------------------------------------------- |
   | The `rhc_auth` variable defines additional variables related to authenticating to Red Hat subscription management, such as the `rhc_auth['login']` and `rhc_auth['activation_keys']` variables. One option for authentication is to specify your username and password. If you use this option, then you might consider protecting these variables with Ansible Vault. A second option is to define activation keys for your organization. |
   | The `rhc_insights` variable defines additional variables related to Red Hat Insights. By default, the `rhc_insights['state']` variable has a value of `present`, which enables Red Hat Insights integration. Additional variables are available for Red Hat Insights when the `rhc_insights['state']` variable has a value of `present`. Set the `rhc_insights['state']` variable to `absent` to disable Red Hat Insights integration. |
   | The `rhc_repositories` variable defines a list of repositories to either enable or disable. |

3. After you create the playbook and define variables for the role, run the playbook to apply the configuration.

#### Configuring an RPM Package Repository(yum_repository)

To enable support for a third-party Yum repository on a managed host, Ansible provides the `ansible.builtin.yum_repository` module.

##### Declaring an RPM Package Repository

When run, the following playbook declares a new Yum repository on `servera.lab.example.com`.

```yaml
---
- name: Configure the company YUM/DNF repositories
  hosts: servera.lab.example.com
  tasks:
    - name: Ensure Example Repo exists
      ansible.builtin.yum_repository:
        file: example   
        name: example-internal
        description: Example Inc. Internal YUM/DNF repo
        baseurl: http://materials.example.com/yum/repository/
        enabled: true
        gpgcheck: true   
        state: present  
```

| The `file` keyword specifies the name of the file to create under the `/etc/yum.repos.d/` directory. The module automatically adds the `.repo` extension to that name. |
| :----------------------------------------------------------- |
| Typically, software providers digitally sign RPM packages using GPG keys. By setting the `gpgcheck` keyword to `true`, the RPM system verifies package integrity by confirming that the package was signed by the appropriate GPG key. The RPM system does not install any package whose GPG signature does not match. Use the `ansible.builtin.rpm_key` Ansible module, described later in this section, to install the required GPG public key. |
| When you set the `state` keyword to `present`, Ansible creates or updates the `.repo` file. When `state` is set to `absent`, Ansible deletes the file. |

The resulting `/etc/yum.repos.d/example.repo` file on `servera.lab.example.com` is as follows:

```
[example-internal]
async = 1
baseurl = http://materials.example.com/yum/repository/
enabled = 1
gpgcheck = 1
name = Example Inc. Internal YUM/DNF repo
```

The `ansible.builtin.yum_repository` module exposes most of the repository configuration parameters as keywords. Run the `ansible-navigator doc ansible.builtin.yum_repository` command for additional parameters and playbook examples.

Some third-party repositories provide the configuration file and the GPG public key as part of an RPM package that can be downloaded and installed using the `dnf install` command.

For example, the *Extra Packages for Enterprise Linux (EPEL)* project provides the `https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm` package that deploys the `/etc/yum.repos.d/epel.repo` configuration file.

For this repository, use the `ansible.builtin.dnf` module to install the EPEL package instead of the `ansible.builtin.yum_repository` module.

##### Importing an RPM GPG Key

When the `gpgcheck` keyword is set to `true` in the `ansible.builtin.yum_repository` module, you also need to install the GPG key on the managed host. The `ansible.builtin.rpm_key` module in the following example deploys the GPG public key hosted on a remote web server to the `servera.lab.example.com` managed host.

```yaml
---
- name: Configure the company YUM/DNF repositories
  hosts: servera.lab.example.com
  tasks:
    - name: Deploy the GPG public key
      ansible.builtin.rpm_key:
        key: http://materials.example.com/yum/repository/RPM-GPG-KEY-example
        state: present

    - name: Ensure Example Repo exists
      ansible.builtin.yum_repository:
        file: example
        name: example-internal
        description: Example Inc. Internal YUM/DNF repo
        baseurl: http://materials.example.com/yum/repository/
        enabled: true
        gpgcheck: true
        state: present
```

#### References

`dnf`(8), `yum.conf`(5), and `subscription-manager`(8) man pages

[ansible.builtin.dnf module - Manages Packages with the DNF Package Manager - Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html)

[ansible.builtin.package_facts module - Package Information as Facts - Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_facts_module.html)

[Introduction to the rhel-system-roles.rhc Role](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/automating_system_administration_by_using_rhel_system_roles/index#using-the-rhc-system-role-to-register-the-system_automating-system-administration-by-using-rhel-system-roles)

[Using the redhat.rhel_system_roles.rhc Collection Role](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/rhel_system_roles/content/role/rhc/)

[ansible.builtin.yum_repository module - Add or Remove YUM Repositories - Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_repository_module.html)

[ansible.builtin.rpm_key module - Adds or Removes a GPG Key from the RPM DB - Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/rpm_key_module.html)

#### Example

Write a playbook to ensure that the `simple-agent` package is installed on all managed hosts. The playbook must also ensure that all managed hosts are configured to use the internal Yum repository.

The repository is located at `http://materials.example.com/yum/repository`. All RPM packages in the repository are signed with a GPG key pair. The GPG public key for the repository packages is available at `http://materials.example.com/yum/repository/RPM-GPG-KEY-example`.

```yaml
[student@workstation system-software]$ cat ansible.cfg 
[defaults]
inventory=inventory
remote_user=devops

#Try me...
#callback_whitelist=timer

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[student@workstation system-software]$ cat inventory 
servera.lab.example.com
```

```yaml
# repo_playbook.yml
---
- name: Config yum repo for installing simple-agent
  hosts: all
  gather_facts: false
  vars: 
    custom_pkg: simple-agent
  tasks:
    - name: Gather information about installed packages
      ansible.builtin.package_facts:
        manager: auto
    
    # Check if the simple-agent installed and the version of it
    - name: Display custom package version
      ansible.builtin.debug:
        var: ansible_facts['packages'][custom_pkg][0]['version']
      when: ansible_facts['packages'][custom_pkg] is defined

    - name: Check Example Repo exists
      ansible.builtin.yum_repository:
        file: example
        name: example-internal
        baseurl: http://materials.example.com/yum/repository
        description: Example Inc. Internal YUM repo
        enabled: true
        gpgcheck: true
        state: present

    - name: Ensure Repo RPM key is installed
      ansible.builtin.rpm_key:
        key: http://materials.example.com/yum/repository/RPM-GPG-KEY-example
        state: present
        
    - name: Install Example package
      ansible.builtin.dnf:
        name: '{{ custom_pkg }}'
        state: present
	
	# Gather ansible_facts again after the packages is installed
    - name: Gather information about installed packages
      ansible.builtin.package_facts:
        manager: auto
    
    # show the simple-agent information
    - name: Display custom package version
      ansible.builtin.debug:
        var: ansible_facts['packages'][custom_pkg]
      when: custom_pkg in ansible_facts['packages']

```

Result:

```json
[student@workstation system-software]$ ansible-navigator run -m stdout repo_playbook.yml 
PLAY [Config yum repo for installing simple-agent] *****************************

TASK [Gather information about installed packages] *****************************
ok: [servera.lab.example.com]

TASK [Display custom package version] ******************************************
skipping: [servera.lab.example.com]   # skipped due to the package not installed yet

TASK [Check Example Repo exists] ***********************************************
ok: [servera.lab.example.com]

TASK [Ensure Repo RPM key is installed] ****************************************
ok: [servera.lab.example.com]

TASK [Install Example package] *************************************************
changed: [servera.lab.example.com]

TASK [Gather information about installed packages] *****************************
ok: [servera.lab.example.com]

TASK [Display custom package version] ******************************************
ok: [servera.lab.example.com] => {
    "ansible_facts['packages'][custom_pkg]": [
        {
            "arch": "x86_64",
            "epoch": null,
            "name": "simple-agent",
            "release": "1.el9",
            "source": "rpm",
            "version": "1.0"
        }
    ]
}

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=6    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```



### Managing Users and Authentication


#### The User Module

The Ansible `ansible.builtin.user` module lets you create, configure, and remove user accounts on managed hosts. You can remove or add a user, set a user's home directory, set the UID for system user accounts, manage passwords, and assign a user to supplementary groups. 

To create a user that can log in to the machine, you need to provide a hashed password for the password parameter. See ["How do I generate encrypted passwords for the user module?"](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module) for information on how to hash a password.

The following example demonstrates the `ansible.builtin.user` module:

```yaml
- name: Create devops_user if missing, make sure is member of correct groups
  ansible.builtin.user:
    name: devops_user 
    shell: /bin/bash 
    groups: sys_admins, developers 
    append: true
```

| The `name` parameter is the only option required by the `ansible.builtin.user` module. Its value is the name of the service account or user account to create, remove, or modify. |
| ------------------------------------------------------------ |
| The `shell` parameter sets the user's shell.                 |
| The `groups` parameter, when used with the `append` parameter, tells the machine to append the supplementary groups `sys_admins` and `developers` to this user. If you do not use the `append` parameter then the groups provided overwrite a user's existing supplementary groups. To set the primary group for a user, use the `group` option. |

**Note**

The `ansible.builtin.user` module also provides information in return values, such as the user's home directory and a list of groups that the user is a member of. These return values can be registered into a variable and used in subsequent tasks. More information is available in the documentation for the module.



**Table 9.1. Commonly Used Parameters for the User Module**

| Parameter     | Comments                                                     |
| :------------ | :----------------------------------------------------------- |
| `comment`     | Optionally sets the description of a user account.           |
| `group`       | Optionally sets the user's primary group.                    |
| `groups`      | Optionally sets a list of supplementary groups for the user. When set to a null value, all groups except the primary group are removed. |
| `home`        | Optionally sets the user's home directory location.          |
| `create_home` | Optionally takes a Boolean value of `true` or `false`. A home directory is created for the user if the value is set to `true`. |
| `system`      | Optionally takes a Boolean value of `true` or `false`. When creating an account, this makes the user a system account if the value is set to `true`. This setting cannot be changed on existing users. |
| `uid`         | Sets the UID number of the user.                             |
| `state`       | If set to `present`, create the account if it is missing (the default setting). If set to `absent`, remove the account if it is present. |

##### Use the User Module to Generate an SSH Key

The `ansible.builtin.user` module can generate an SSH key if called with the `generate_ssh_key` parameter.

The following example demonstrates how the `ansible.builtin.user` module generates an SSH key:

```yaml
- name: Create an SSH key for user1
  ansible.builtin.user:
    name: user1
    generate_ssh_key: true 
    ssh_key_bits: 2048 
    ssh_key_file: .ssh/id_my_rsa 
```

| The `generate_ssh_key` parameter accepts a Boolean value that specifies whether to generate an SSH key for the user. This does not overwrite an existing SSH key unless the `force` parameter is provided with the `true` value. |
| ------------------------------------------------------------ |
| The `ssh_key_bits` parameter sets the number of bits in the new SSH key. |
| The `ssh_key_file` parameter specifies the file name for the new SSH private key (the public key adds the `.pub` suffix). |



#### The Group Module

The `ansible.builtin.group` module adds, deletes, and modifies groups on the managed hosts. The managed hosts need to have the `groupadd`, `groupdel`, and `groupmod` commands available, which are provided by the `shadow-utils` package in Red Hat Enterprise Linux 9. For Microsoft Windows managed hosts, use the `win_group` module.

The following example demonstrates how the `ansible.builtin.group` module creates a group:

```yaml
- name: Verify that the auditors group exists
  ansible.builtin.group:
    name: auditors 
    state: present 
```



**Table 9.2. Parameters for the Group Module**

| Parameter | Comments                                                     |
| :-------- | :----------------------------------------------------------- |
| `gid`     | This parameter sets the GID number to for the group. If omitted, the number is automatically selected. |
| `local`   | This parameter forces the use of local command alternatives (instead of commands that might change central authentication sources) on platforms that implement it. |
| `name`    | This parameter sets the name of the group to manage.         |
| `state`   | This parameter determines whether the group should be `present` or `absent` on the remote host. |
| `system`  | If this parameter is set to `true`, then the group is created as a system group (typically, with a GID number below 1000). |

#### The Known Hosts Module

The `ansible.builtin.known_hosts` module manages SSH host keys by adding or removing them on managed hosts. This ensures that managed hosts can automatically establish the authenticity of SSH connections to other managed hosts, ensuring that users are not prompted to verify a remote managed host's SSH fingerprint the first time they connect to it.

The following example demonstrates how the `ansible.builtin.known_hosts` module copies a host key to a managed host:

```yaml
- name: Copy host keys to remote servers
  ansible.builtin.known_hosts:
    path: /etc/ssh/ssh_known_hosts 
    name: servera.lab.example.com 
    key: servera.lab.example.com,172.25.250.10 ssh-rsa ASDeararAIUHI324324 
```

| The `path` parameter specifies the path to the `known_hosts` file to edit. If the file does not exist, then it is created. |
| ------------------------------------------------------------ |
| The `name` parameter specifies the name of the host to add or remove. The name must match the hostname or IP address of the key being added. |
| The `key` parameter is the SSH public host key as a string in a specific format. For example, the value for the `key` parameter must be in the format `<hostname[,IP]> ssh-rsa <pubkey>` for an RSA public host key (found in a host's `/etc/ssh/ssh_host_rsa_key.pub` key file), or `<hostname[,IP]> ssh-ed25519 <pubkey>` for an Ed25519 public host key (found in a host's `/etc/ssh/ssh_host_ed25519_key.pub` key file). |

The following example demonstrates how to use the `lookup` plug-in to populate the `key` parameter from an existing file in the Ansible project:

```yaml
- name: Copy host keys to remote servers
  ansible.builtin.known_hosts:
    path: /etc/ssh/ssh_known_hosts
    name: serverb
    key: "{{ lookup('ansible.builtin.file', 'pubkeys/serverb') }}" 
```

| This Jinja2 expression uses the `lookup` function with the `ansible.builtin.file` lookup plug-in to load the content of the `pubkeys/serverb` key file from the Ansible project as the value of the `key` option. You can list available lookup plug-ins using the `ansible-navigator doc -l -t lookup` command |
| ------------------------------------------------------------ |

The following play is an example that uses some advanced techniques to construct an `/etc/ssh/ssh_known_hosts` file for all managed hosts in the inventory. There might be more efficient ways to accomplish this, because it runs a nested loop on all managed hosts.

It uses the `ansible.builtin.slurp` module to get the content of the RSA and Ed25519 SSH public host keys in Base64 format, and then processes the values of the registered variable with the `b64decode` and `trim` filters to convert those values back to plain text.

```yaml
- name: Configure /etc/ssh/ssh_known_hosts files
  hosts: all

  tasks:
    - name: Collect RSA keys
      ansible.builtin.slurp:
        src: /etc/ssh/ssh_host_rsa_key.pub
      register: rsa_host_keys

    - name: Collect Ed25519 keys
      ansible.builtin.slurp:
        src: /etc/ssh/ssh_host_ed25519_key.pub
      register: ed25519_host_keys

    - name: Deploy known_hosts
      ansible.builtin.known_hosts:
        path: /etc/ssh/ssh_known_hosts
        name: "{{ item[0] }}" 
        key: "{{ hostvars[ item[0] ]['ansible_facts']['fqdn'] }} {{ hostvars[ item[0] ][ item[1] ]['content'] | b64decode | trim }}" 
        state: present
      with_nested:
        - "{{ ansible_play_hosts }}" 
        - [ 'rsa_host_keys', 'ed25519_host_keys' ] 
```

| `item[0]` is an inventory hostname from the list in the `ansible_play_hosts` variable. |
| ------------------------------------------------------------ |
| `item[1]` is the string `rsa_host_keys` or `ed25519_host_keys`. The `b64decode` filter converts the value stored in the variable from Base64 to plain text, and the `trim` filter removes an unnecessary newline. This is all one line starting with `key`, and there is a single space between the two Jinja2 expressions. |
| `ansible_play_hosts` is a list of the hosts remaining in the play at this point, taken from the inventory and removing hosts with failed tasks. The play must retrieve the RSA and Ed25519 public host keys for each of the other hosts when it constructs the `known_hosts` file on each host in the play. |
| This is a two-item list of the two variables that the play uses to store host keys. |

**Note**

Lookup plug-ins and filters are covered in more detail in the course *DO374: Developing Advanced Automation with Red Hat Ansible Automation Platform*.

#### The Authorized Key Module

The `ansible.posix.authorized_key` module manages SSH authorized keys for user accounts on managed hosts.

/home/your_username/.ssh/authorized_keys

The following example demonstrates how to use the `ansible.posix.authorized_key` module to add an SSH key to a managed host:

```yaml
- name: Set authorized key
  ansible.posix.authorized_key:
    user: user1 
    state: present 
    key: "{{ lookup('ansible.builtin.file', 'files/user1/id_rsa.pub') }}" 
```

| The `user` parameter specifies the username of the user whose `authorized_keys` file is modified on the managed host. |
| ------------------------------------------------------------ |
| The `state` parameter accepts the `present` or `absent` value with `present` as the default. |
| The `key` parameter specifies the SSH public key to add or remove. In this example, the `lookup` function uses the `ansible.builtin.file` lookup plug-in to load the contents of the `files/user1/id_rsa.pub` file in the Ansible project as the value for `key`. As an alternative, you can provide a URL to a public key file as this value. |

#### Configuring Sudo Access for Users and Groups

In Red Hat Enterprise Linux 9, you can configure access for a user or group to run `sudo` commands without requiring a password prompt.

The following example demonstrates how to use the `ansible.builtin.lineinfile` module to provide a group with `sudo` access to the `root` account without prompting the group members for a password:

```yaml
- name: Modify sudo to allow the group01 group sudo without a password
  ansible.builtin.lineinfile:
    path: /etc/sudoers.d/group01 
    state: present 
    create: true 
    mode: 0440 
    line: "%group01 ALL=(ALL) NOPASSWD: ALL" 
    validate: /usr/sbin/visudo -cf %s 
```

| The `path` parameter specifies the file to modify in the `/etc/sudoers.d/` directory. It is a good practice to match the file name with the name of the user or group you are providing access to. This makes it easier for future reference. |
| ------------------------------------------------------------ |
| The `state` parameter accepts the `present` or `absent` value. The default value is `present`. |
| The `create` parameter takes a Boolean value and specifies if the file should be created if it does not already exist. The default value for the `create` parameter is `false`. |
| The `mode` parameter specifies the permissions on the `sudoers` file. |
| The `line` parameter specifies the line to add to the file. The format is specific, and an example can be found in the `/etc/sudoers` file under the "Same thing but without a password" comment. If you are configuring `sudo` access for a group, then you need to add a percent sign (`%`) to the beginning of the group name. If you are configuring `sudo` access for a user, then do not add the percent sign. |
| The `validate` parameter specifies the command to run to verify that the file is correct. When the `validate` parameter is present, the file is created in a temporary file path and the provided command validates the temporary file. If the validate command succeeds, then the temporary file is copied to the path specified in the `path` parameter and the temporary file is removed. |

An example of the `sudo` validation command can be found in the examples section of the output from the `ansible-navigator doc ansible.builtin.lineinfile` command.

#### References

[Users Module Ansible Documentation](http://docs.ansible.com/ansible/latest/modules/user_module.html#user-module)

[How do I generate encrypted passwords for the user module](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module)

[Group Module Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/group_module.html#group-module)

[SSH Known Hosts Module Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/known_hosts_module.html#known-hosts-module)

[Authorized Key Module Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html#authorized-key-module)

[The Lookup Plugin Ansible Documentation](https://docs.ansible.com/ansible/latest/plugins/lookup.html?highlight=lookup)

[Using Filters to Manipulate Data](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)

[The Line in File Module Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)



#### Example

- Create a new user group.
- Manage users by using the `ansible.builtin.user` module.
- Populate SSH authorized keys by using the `ansible.posix.authorized_key` module.
- Modify the `/etc/ssh/sshd_config` file and a configuration file in `/etc/sudoers.d` by using the `ansible.builtin.lineinfile` module.

```
[student@workstation system-users]$ tree
.
├── ansible.cfg
├── ansible-navigator.log
├── files
│   ├── user1.key.pub
│   ├── user2.key.pub
│   ├── user3.key.pub
│   ├── user4.key.pub
│   └── user5.key.pub
├── inventory
├── users.yml
└── vars
    └── users_vars.yml
```

contents:

```
[student@workstation system-users]$ cat ansible.cfg 
[defaults]
remote_user=devops
inventory=./inventory


[privilege_escalation]
become=yes
become_method=sudo

[student@workstation system-users]$ cat inventory 
[webservers]
servera.lab.example.com

[student@workstation system-users]$ cat vars/users_vars.yml 
---
users:
  - username: user1
    groups: webadmin
  - username: user2
    groups: webadmin
  - username: user3
    groups: webadmin
  - username: user4
    groups: webadmin
  - username: user5
    groups: webadmin
```

The final playbook file as :

```yaml
---
- name: Adding users
  hosts: webservers
  vars_files:
    - vars/users_vars.yml

  tasks:
    - name: Creating user group
      ansible.builtin.group:
        name: webadmin
        state: present

    - name: Create uses from vars file
      ansible.builtin.user:
        name: "{{ item['username'] }}"
        groups: "{{ item['groups'] }}"
      loop: "{{ users }}"

    - name: Populate the SSH pub keys
      ansible.posix.authorized_key:
        user: "{{ item['username'] }}"
        state: present
        key: "{{ lookup('file', 'files/'+item['username']+'.key.pub') }}"
      loop: "{{ users }}"
    
    - name: Modify to make sudo without passwd
      ansible.builtin.lineinfile:
        path: /etc/sudoers.d/webadmin
        state: present
        create: true
        mode: 0440
        line: "%webadmin ALL=(ALL) NOPASSWD: ALL"
        validate: /usr/sbin/visudo -cf %s
f
    - name: Disable root login via SSH
      ansible.builtin.lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: "^PermitRootLogin"
        line: "PermitRootLogin no"
      notify: Restart sshd

  handlers:
    - name: Restart sshd
      ansible.builtin.service:
        name: sshd
        state: restarted
```

test:

```
[student@workstation system-users]$ ssh user1@servera

[student@workstation system-users]$ ssh root@servera
root@servera's password: 
Permission denied, please try again.
```



### Managing the Boot Process and Scheduled Processes

#### Scheduling Jobs for Future Execution

- The `at` command schedules jobs that run once at a specified time. 
- The Cron subsystem schedules jobs to run on a recurring schedule, either in a user's personal `crontab` file, in the system Cron configuration in `/etc/crontab`, or as a file in `/etc/cron.d`. 
- The `systemd` subsystem also provides timer units that can start service units on a set schedule.

##### Scheduling Jobs That Run One Time

Quick one-time scheduling is done with the `ansible.posix.at` module. You create the job to run at a future time, and it is held until that time to execute.



**Table 9.3. Options for the `ansible.posix.at` Module**

| Option        | Comments                                                     |
| :------------ | :----------------------------------------------------------- |
| `command`     | The command to schedule to run in the future.                |
| `count`       | The integer number of units from now that the job should run. (Must be used with `units`.) |
| `units`       | Specifies whether `count` is measured in minutes, hours, days, or weeks. |
| `script_file` | An existing script file to schedule to run in the future.    |
| `state`       | The default value (`present`) adds a job; `absent` removes a matching job if present. |
| `unique`      | If set to `true`, then if a matching job is already present, a new job is not added. |

In the following example, the task shown uses `at` to schedule the `userdel -r tempuser` command to run in 20 minutes.

```yaml
- name: Remove tempuser
  ansible.posix.at:
    command: userdel -r tempuser
    count: 20
    units: minutes
    unique: true
```

##### Scheduling Repeating Jobs with Cron

You can configure a command that runs on a repeating schedule by using Cron. To set up Cron jobs, use the `ansible.builtin.cron` module. The `name` option is mandatory, and is inserted in the `crontab` as a description of the repeating job. It is also used by the module to determine if the Cron job already exists, or which Cron job to modify or delete.

Some commonly used parameters for the `ansible.builtin.cron` module include:



**Table 9.4. Options for the `ansible.builtin.cron` Module**

| Options                                     | Comments                                                     |
| :------------------------------------------ | :----------------------------------------------------------- |
| `name`                                      | The comment identifying the Cron job.                        |
| `job`                                       | The command to run.                                          |
| `minute`, `hour`, `day`, `month`, `weekday` | The value for the field in the time specification for the job in the `crontab` entry. If not set, `"*"` (all values) is assumed. |
| `state`                                     | If set to `present`, it creates the Cron job (the default); `absent` removes it. |
| `user`                                      | The Cron job runs as this user. If `cron_file` is not specified, the job is set in that user's `crontab` file. |
| `cron_file`                                 | If set, create a system Cron job in `cron_file`. You must specify `user` and a time specification. If you use a relative path, then the file is created in `/etc/cron.d`. |

This first example task creates a Cron job in the `testing` user's personal `crontab` file. It runs their personal `backup-home-dir` script at 16:00 every Friday. You could log in as that user and run `crontab -l` after running the playbook to confirm that it worked.

```yaml
- name: Schedule backups for my home directory
  ansible.builtin.cron:
    name: Backup my home directory
    user: testing
    job: /home/testing/bin/backup-home-dir
    minute: 0
    hour: 16
    weekday: 5
```

In the following example, the task creates a system Cron job in the `/etc/cron.d/flush_bolt` file that runs a command as `root` to flush the Bolt cache every morning at 11:45.

```yaml
- name: Schedule job to flush the Bolt cache
  ansible.builtin.cron:
    name: Flush Bolt cache
    cron_file: flush_bolt
    user: "root"
    minute: 45
    hour: 11
    job: "php ./app/nut cache:clear"
```

**Warning**:

Do not use `cron_file` to modify the `/etc/crontab` file. The file you specify must only be maintained by Ansible and should only contain the entry specified by the task.

##### Controlling Systemd Timer Units

The `ansible.builtin.systemd` module can be used to enable or disable existing `systemd` timer units that run recurring jobs (usually `systemd` service units that eventually exit).

The following example disables and stops the `systemd` timer that automatically populates the `dnf` package cache on Red Hat Enterprise Linux 9.

```yaml
- name: Disable dnf makecache
  ansible.builtin.systemd:
    name: dnf-makecache.timer
    state: stopped
    enabled: false
```

#### Managing Services

You can choose between two modules to manage services or reload daemons: `ansible.builtin.systemd` and `ansible.builtin.service`.

The `ansible.builtin.service` module is intended to work with a number of service-management systems, including `systemd`, Upstart, SysVinit, BSD `init`, and others. Because it provides a generic interface to the initialization system, it offers a basic set of options to start, stop, restart, and enable services and other daemons.

```yaml
- name: Start and enable nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

The `ansible.builtin.systemd` module is designed to work with `systemd` only, but it offers additional configuration options specific to that system and service manager.

The following example that uses `ansible.builtin.systemd` is equivalent to the preceding example that used `ansible.builtin.service`:

```yaml
- name: Start nginx
  ansible.builtin.systemd:
    name: nginx
    state: started
    enabled: true
```

The next example reloads the `httpd` daemon, but before it does that it runs `systemctl daemon-reload` to reload the entire `systemd` configuration.

```yaml
- name: Reload web server
  ansible.builtin.systemd:
    name: httpd
    state: reloaded
    daemon_reload: true
```

#### Setting the Default Boot Target

The `ansible.builtin.systemd` module cannot set the default boot target. You can use the `ansible.builtin.command` module to set the default boot target.

```yaml
- name: Change default systemd target
  hosts: all
  gather_facts: false

  vars:
    systemd_target: "multi-user.target" 

  tasks:
    - name: Get current systemd target
      ansible.builtin.command:
        cmd: systemctl get-default 
      # Because this is just gathering information, the task should never report `changed`.
      changed_when: false    
      register: target 

    - name: Set default systemd target
      ansible.builtin.command:
        cmd: systemctl set-default {{ systemd_target }} 
      when: systemd_target not in target['stdout'] 
      # This is the only task in this play that requires `root` access.
      become: true 
```

#### Rebooting Managed Hosts

You can use the dedicated `ansible.builtin.reboot` module to reboot managed hosts during playbook execution. This module reboots the managed host, and waits until the managed host comes back up before continuing with playbook execution. The module determines that a managed host is back up by waiting until Ansible can run a command on the managed host.

The following simple example immediately triggers a reboot:

```yaml
- name: Reboot now
  ansible.builtin.reboot:
```

By default, the playbook waits up to 600 seconds before deciding that the reboot failed, and another 600 seconds before deciding that the test command failed. You can adjust this value so that the timeouts are each 180 seconds. For example:

```yaml
- name: Reboot, shorten timeout
  ansible.builtin.reboot:
    reboot_timeout: 180
```

Some other useful options to the module include:



**Table 9.5. Options for the `ansible.builtin.reboot` Module**

| Options            | Comments                                                     |
| :----------------- | :----------------------------------------------------------- |
| `pre_reboot_delay` | The time to wait before reboot. On Linux, this is measured in minutes, and if less than 60, is rounded down to 0. |
| `msg`              | The message to display to users before reboot.               |
| `test_command`     | The command used to determine whether the managed host is usable and ready for more Ansible tasks after reboot. The default is `whoami`. |

#### References

[ansible.posix.at module - Schedule the execution of a command or script file via the at command — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/posix/at_module.html)

[ansible.builtin.cron module - Manage cron.d and crontab entries — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html)

[ansible.builtin.reboot module - Reboot a machine — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/reboot_module.html)

[ansible.builtin.service module - Manage services — Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/service_module.html)

[ansible.builtin.systemd module - Manage systemd units — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)





### Managing the Boot Process and Scheduled Processes

#### Scheduling Jobs for Future Execution

- The `at` command schedules jobs that run once at a specified time. 
- The Cron subsystem schedules jobs to run on a recurring schedule, either in a user's personal `crontab` file, in the system Cron configuration in `/etc/crontab`, or as a file in `/etc/cron.d`. 
- The `systemd` subsystem also provides timer units that can start service units on a set schedule.

##### Scheduling Jobs That Run One Time

Quick one-time scheduling is done with the `ansible.posix.at` module. You create the job to run at a future time, and it is held until that time to execute.

**Table 9.3. Options for the `ansible.posix.at` Module**

| Option        | Comments                                                     |
| :------------ | :----------------------------------------------------------- |
| `command`     | The command to schedule to run in the future.                |
| `count`       | The integer number of units from now that the job should run. (Must be used with `units`.) |
| `units`       | Specifies whether `count` is measured in minutes, hours, days, or weeks. |
| `script_file` | An existing script file to schedule to run in the future.    |
| `state`       | The default value (`present`) adds a job; `absent` removes a matching job if present. |
| `unique`      | If set to `true`, then if a matching job is already present, a new job is not added. |

In the following example, the task shown uses `at` to schedule the `userdel -r tempuser` command to run in 20 minutes.

```yaml
- name: Remove tempuser
  ansible.posix.at:
    command: userdel -r tempuser
    count: 20
    units: minutes
    unique: true
    state: present
```

##### Scheduling Repeating Jobs with Cron

You can configure a command that runs on a repeating schedule by using Cron. To set up Cron jobs, use the `ansible.builtin.cron` module. The `name` option is mandatory, and is inserted in the `crontab` as a description of the repeating job. It is also used by the module to determine if the Cron job already exists, or which Cron job to modify or delete.

Some commonly used parameters for the `ansible.builtin.cron` module include:

**Table 9.4. Options for the `ansible.builtin.cron` Module**

| Options                                     | Comments                                                     |
| :------------------------------------------ | :----------------------------------------------------------- |
| `name`                                      | The comment identifying the Cron job.                        |
| `job`                                       | The command to run.                                          |
| `minute`, `hour`, `day`, `month`, `weekday` | The value for the field in the time specification for the job in the `crontab` entry. If not set, `"*"` (all values) is assumed. |
| `state`                                     | If set to `present`, it creates the Cron job (the default); `absent` removes it. |
| `user`                                      | The Cron job runs as this user. If `cron_file` is not specified, the job is set in that user's `crontab` file. |
| `cron_file`                                 | If set, create a system Cron job in `cron_file`. You must specify `user` and a time specification. If you use a relative path, then the file is created in `/etc/cron.d`. |

This first example task creates a Cron job in the `testing` user's personal `crontab` file. It runs their personal `backup-home-dir` script at 16:00 every Friday. You could log in as that user and run `crontab -l` after running the playbook to confirm that it worked.

```yaml
- name: Schedule backups for my home directory
  ansible.builtin.cron:
    name: Backup my home directory
    user: testing
    job: /home/testing/bin/backup-home-dir
    minute: 0
    hour: 16
    weekday: 5
```

In the following example, the task creates a system Cron job in the `/etc/cron.d/flush_bolt` file that runs a command as `root` to flush the Bolt cache every morning at 11:45.

```yaml
- name: Schedule job to flush the Bolt cache
  ansible.builtin.cron:
    name: Flush Bolt cache
    cron_file: flush_bolt
    user: "root"
    minute: 45
    hour: 11
    job: "php ./app/nut cache:clear"
```

**Warning**:

Do not use `cron_file` to modify the `/etc/crontab` file. The file you specify must only be maintained by Ansible and should only contain the entry specified by the task.

##### Controlling Systemd Timer Units

The `ansible.builtin.systemd` module can be used to enable or disable existing `systemd` timer units that run recurring jobs (usually `systemd` service units that eventually exit).

The following example disables and stops the `systemd` timer that automatically populates the `dnf` package cache on Red Hat Enterprise Linux 9.

```yaml
- name: Disable dnf makecache
  ansible.builtin.systemd:
    name: dnf-makecache.timer
    state: stopped
    enabled: false
```



#### Managing Services

You can choose between two modules to manage services or reload daemons: `ansible.builtin.systemd` and `ansible.builtin.service`.

The `ansible.builtin.service` module is intended to work with a number of service-management systems, including `systemd`, Upstart, SysVinit, BSD `init`, and others. Because it provides a generic interface to the initialization system, it offers a basic set of options to start, stop, restart, and enable services and other daemons.

```yaml
- name: Start and enable nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

The `ansible.builtin.systemd` module is designed to work with `systemd` only, but it offers additional configuration options specific to that system and service manager.

The following example that uses `ansible.builtin.systemd` is equivalent to the preceding example that used `ansible.builtin.service`:

```yaml
- name: Start nginx
  ansible.builtin.systemd:
    name: nginx
    state: started
    enabled: true
```

The next example reloads the `httpd` daemon, but before it does that it runs `systemctl daemon-reload` to reload the entire `systemd` configuration.

```yaml
- name: Reload web server
  ansible.builtin.systemd:
    name: httpd
    state: reloaded
    daemon_reload: true
```

#### Setting the Default Boot Target

The `ansible.builtin.systemd` module cannot set the default boot target. You can use the `ansible.builtin.command` module to set the default boot target.

```yaml
- name: Change default systemd target
  hosts: all
  gather_facts: false

  vars:
    systemd_target: "multi-user.target" 

  tasks:
    - name: Get current systemd target
      ansible.builtin.command:
        cmd: systemctl get-default 
      changed_when: false 
      register: target 

    - name: Set default systemd target
      ansible.builtin.command:
        cmd: systemctl set-default {{ systemd_target }} 
      when: systemd_target not in target['stdout'] 
      become: true 
```

#### Rebooting Managed Hosts

You can use the dedicated `ansible.builtin.reboot` module to reboot managed hosts during playbook execution. This module reboots the managed host, and waits until the managed host comes back up before continuing with playbook execution. The module determines that a managed host is back up by waiting until Ansible can run a command on the managed host.

The following simple example immediately triggers a reboot:

```yaml
- name: Reboot now
  ansible.builtin.reboot:
```

By default, the playbook waits up to 600 seconds before deciding that the reboot failed, and another 600 seconds before deciding that the test command failed. You can adjust this value so that the timeouts are each 180 seconds. For example:

```yaml
- name: Reboot, shorten timeout
  ansible.builtin.reboot:
    reboot_timeout: 180
```

Some other useful options to the module include:



**Table 9.5. Options for the `ansible.builtin.reboot` Module**

| Options            | Comments                                                     |
| :----------------- | :----------------------------------------------------------- |
| `pre_reboot_delay` | The time to wait before reboot. On Linux, this is measured in minutes, and if less than 60, is rounded down to 0. |
| `msg`              | The message to display to users before reboot.               |
| `test_command`     | The command used to determine whether the managed host is usable and ready for more Ansible tasks after reboot. The default is `whoami`. |

#### References

[ansible.posix.at module - Schedule the execution of a command or script file via the at command — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/posix/at_module.html)

[ansible.builtin.cron module - Manage cron.d and crontab entries — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html)

[ansible.builtin.reboot module - Reboot a machine — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/reboot_module.html)

[ansible.builtin.service module - Manage services — Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/service_module.html)

[ansible.builtin.systemd module - Manage systemd units — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)



Example:

```
[student@workstation system-process]$ cat ansible.cfg 
[defaults]
remote_user = devops
inventory = ./inventory

[privilege_escalation]
become = yes
become_method = sudo

[student@workstation system-process]$ cat inventory 
[webservers]
servera.lab.example.com
```



1. Create the `create_crontab_file.yml` playbook in the working directory. 

   Configure the playbook to use the `ansible.builtin.cron` module to create a `crontab` file named `/etc/cron.d/add-date-time` that schedules a recurring Cron job. The job should run as the `devops` user every two minutes starting at 09:00 and ending at 16:59 from Monday through Friday. The job should append the current date and time to the `/home/devops/my_datetime_cron_job` file.

   ```yaml
   ---
   - name: Create a crontab file
     hosts: webservers
     become: true
     
     tasks:
       - name: building jobs
         ansible.builtin.cron:
           name: add date and time to a file
           job: date >> /home/devops/my_date_time_cron_job 
           minute: "*/2"
           hour: 9-16
           weekday: 1-5
           user: devops
           cron_file: add-date-time
           state: present
   ```

   

2. Create the `remove_cron_job.yml` playbook in the working directory. Configure the playbook to use the `ansible.builtin.cron` module to remove the `Add date and time to a file` Cron job from the `/etc/cron.d/add-date-time` `crontab` file.

   ```yaml
   ---
   - name: Remove a crontab file
     hosts: webservers
     become: true
     
     tasks:
       - name: building jobs
         ansible.builtin.cron:
           name: add date and time to a file
           user: devops
           cron_file: add-date-time
           state: absent
   ```

   

3. Create the `schedule_at_task.yml` playbook in the working directory. Configure the playbook to use the `ansible.posix.at` module to schedule a task that runs one minute in the future. The task should run the `date` command and redirect its output to the `/home/devops/my_at_date_time` file.  Use the `unique: true` option to ensure that if the command already exists in the `at` queue, a new task is not added.

   ```yaml
   ---
   - name: How to use AT on a task
     hosts: webservers
     become: true
     become_user: devops
   
     tasks:
       - name: AT task in the future
         ansible.posix.at:
           command: date >> /home/devops/my_at_date_time
           count: 1
           units: minutes
           unique: true
           state: present
   ```

   

4. Create the `set_default_boot_target_graphical.yml` playbook in the working directory. Write a play in the playbook to set the default `systemd` target to `graphical.target`.

   ```yaml
   ---
   - name: Set the default boot target graphical
     hosts: webservers
     become: true
     vars:
       new_target: "graphical.target"
     tasks:
       - name: Get current target
         ansible.builtin.command: 
           cmd: systemctl get-default
         changed_when: false
         register: default_target
   
       - name: Change to new target mode
         ansible.builtin.command: 
           cmd: systemctl set-default {{ new_target }}
         when: new_target not in default_target.stdout
         become: true
   ```

   

### Managing Storage

#### Mounting Existing File Systems

Use the `ansible.posix.mount` module to mount an existing file system. The most common parameters are：

- the `path` parameter, which specifies the path to mount the file system to
- the `src` parameter, which specifies the device (this could be a device name, UUID, or NFS volume)
- the `fstype` parameter, which specifies the file system type
- the `state` parameter, which accepts the `absent`, `mounted`, `present`, `unmounted`, or `remounted` values.

The following example task mounts the NFS share available at `172.25.250.100:/share` on the `/nfsshare` directory on the managed hosts.

```yaml
- name: Mount NFS share
  ansible.posix.mount:
    path: /nfsshare
    src: 172.25.250.100:/share
    fstype: nfs
    opts: defaults
    dump: '0'
    passno: '0'
    state: mounted
```

#### Configuring Storage with the Storage System Role

Red Hat Ansible Automation Platform provides the `redhat.rhel_system_roles.storage` system role to configure local storage devices on your managed hosts. It can manage file systems on unpartitioned block devices, and format and create logical volumes on LVM physical volumes based on unpartitioned block devices.

The `redhat.rhel_system_roles.storage` role formally supports managing file systems and mount entries for two use cases:

- Unpartitioned devices (whole-device file systems)
- LVM on unpartitioned whole-device physical volumes

If you have other use cases, then you might need to use other modules and roles to implement them.



##### Managing a File System on an Unpartitioned Device

To create a file system on an unpartitioned block device with the `redhat.rhel_system_roles.storage` role, define the `storage_volumes` variable. The `storage_volumes` variable contains a list of storage devices to manage.

The following dictionary items are available in the `storage_volumes` variable:



**Table 9.6. Parameters for the `storage_volumes` Variable**

| Parameter       | Comments                                                     |
| :-------------- | :----------------------------------------------------------- |
| `name`          | The name of the volume.                                      |
| `type`          | This value must be `disk`.                                   |
| `disks`         | Must be a list of exactly one item; the unpartitioned block device. |
| `mount_point`   | The directory on which the file system is mounted.           |
| `fstype`        | The file system type to use. (`xfs`, `ext4`, or `swap`.)     |
| `mount_options` | Custom mount options, such as `ro` or `rw`.                  |

The following example play creates an XFS file system on the `/dev/vdg` device, and mounts it on `/opt/extra`.

```yaml
- name: Example of a simple storage device
  hosts: all

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_volumes:
        - name: extra
          type: disk
          disks:
            - /dev/vdg
          fs_type: xfs
          mount_point: /opt/extra
```

##### Managing LVM with the Storage Role

To create an LVM volume group with the `redhat.rhel_system_roles.storage` role, define the `storage_pools` variable. The `storage_pools` variable contains a list of pools (LVM volume groups) to manage.

The dictionary items inside the `storage_pools` variable are used as follows:

- The `name` variable is the name of the volume group.
- The `type` variable must have the value `lvm`.
- The `disks` variable is the list of block devices that the volume group uses for its storage.
- The `volumes` variable is the list of logical volumes in the volume group.

The following entry creates the volume group `vg01` with the `type` key set to the value `lvm`. The volume group's physical volume is the `/dev/vdb` disk.

```yaml
---
- name: Configure storage on webservers
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: vg01
          type: lvm
          disks:
            - /dev/vdb
```

The `disks` option only supports **unpartitioned block devices** for your LVM physical volumes.

To create logical volumes, populate the `volumes` variable, nested under the `storage_pools` variable, with a list of logical volume names and their parameters. Each item in the list is a dictionary that represents a single logical volume within the `storage_pools` variable.

Each logical volume list item has the following dictionary variables:

- `name`: The name of the logical volume.
- `size`: The size of the logical volume.
- `mount_point`: The directory used as the mount point for the logical volume's file system.
- `fs_type`: The logical volume's file system type.
- `state`: Whether the logical volume should exist using the `present` or `absent` values.

The following example creates two logical volumes, named `lvol01` and `lvol02`. The `lvol01` logical volume is 128 MB in size, formatted with the `xfs` file system, and is mounted at `/data`. The `lvol02` logical volume is 256 MB in size, formatted with the `xfs` file system, and is mounted at `/backup`.

```yaml
---
- name: Configure storage on webservers
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: vg01
          type: lvm
          disks:
            - /dev/vdb
          volumes:
            - name: lvol01
              size: 128m
              mount_point: "/data"
              fs_type: xfs
              state: present
            - name: lvol02
              size: 256m
              mount_point: "/backup"
              fs_type: xfs
              state: present
```

In the following example entry, if the `lvol01` logical volume is already created with a size of 128 MB, then the logical volume and file system are enlarged to 256 MB, assuming that the space is available within the volume group.

```yaml
          volumes:
            - name: lvol01
              size: 256m
              mount_point: "/data"
              fs_type: xfs
              state: present
```

##### Configuring Swap Space

You can use the `redhat.rhel_system_roles.storage` role to create logical volumes that are formatted as swap spaces. The role creates the logical volume, the swap file system type, adds the swap volume to the `/etc/fstab` file, and enables the swap volume immediately.

The following playbook example creates the `lvswap` logical volume in the `vgswap` volume group, adds the swap volume to the `/etc/fstab` file, and enables the swap space.

```yaml
---
- name: Configure a swap volume
  hosts: all

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: vgswap
          type: lvm
          disks:
            - /dev/vdb
          volumes:
            - name: lvswap
              size: 512m
              fs_type: swap
              state: present
```

#### Managing Partitions and File Systems with Tasks

You can manage partitions and file systems on your storage devices without using the system role. However, the most convenient modules for doing this are currently unsupported by Red Hat, which can make this more complicated.

##### Managing Partitions

If you want to partition your storage devices without using the system role, your options are a bit more complex.

- The unsupported `community.general.parted` module in the `community.general` Ansible Content Collection can perform this task.
- You can use the `ansible.builtin.command` module to run the partitioning commands on the managed hosts. However, you need to take special care to make sure the commands are idempotent and do not inadvertently destroy data on your existing storage devices.

For example, the following task creates a GPT disk label and a `/dev/sda1` partition on the `/dev/sda` storage device only if `/dev/sda1` does not already exist:

```yaml
- name: Ensure that /dev/sda1 exists
  ansible.builtin.command:
    cmd: parted --script mklabel gpt mkpart primary 1MiB 100%
    creates: /dev/sda1
```

This depends on the fact that if the `/dev/sda1` partition exists, then a Linux system creates a `/dev/sda1` device file for it automatically.

##### Managing File Systems

The easiest way to manage file systems without using the system role might be the `community.general.filesystem` module. However, Red Hat does not support this module, so you use it at your own risk.

As an alternative, you can use the `ansible.builtin.command` module to run commands to format file systems. However, you should use some mechanism to make sure that the device you are formatting does not already contain a file system, to ensure idempotency of your play, and to avoid accidental data loss. One way to do that might be to review storage-related facts gathered by Ansible to determine if a device appears to be formatted with a file system.

#### Ansible Facts for Storage Configuration

Ansible facts gathered by `ansible.builtin.setup` contain useful information about the storage devices on your managed hosts.

##### Facts about Block Devices

The `ansible_facts['devices']` fact includes information about all the storage devices available on the managed host. This includes additional information such as the partitions on each device, or each device's total size.

The following playbook gathers and displays the `ansible_facts['devices']` fact for each managed host.

```yaml
---
- name: Display storage facts
  hosts: all

  tasks:
    - name: Display device facts
      ansible.builtin.debug:
        var: ansible_facts['devices']
```

This fact contains a dictionary of variables named for the devices on the system. Each named device variable itself has a dictionary of variables for its value, which represent information about the device. For example, if you have the `/dev/sda` device on your system, you can use the following Jinja2 expression (all on one line) to determine its size in bytes:

```jinja2
{{ ansible_facts['devices']['sda']['sectors'] * ansible_facts['devices']['sda']['sectorsize'] }}
```



**Table 9.7. Selected Facts from a Device Variable Dictionary**

| Fact         | Comments                                                     |
| :----------- | :----------------------------------------------------------- |
| `host`       | A string that identifies the controller to which the block device is connected. |
| `model`      | A string that identifies the model of the storage device, if applicable. |
| `partitions` | A dictionary of block devices that are partitions on this device. Each dictionary variable has as its value a dictionary structured like any other device (including values for `sectors`, `size`, and so on). |
| `sectors`    | The number of storage sectors the device contains.           |
| `sectorsize` | The size of each sector in bytes.                            |
| `size`       | A human-readable rough calculation of the device size.       |

For example, you could find the size of `/dev/sda1` from the following fact:

```
ansible_facts['devices']['sda']['partitions']['sda1']['size']
```

##### Facts about Device Links

The `ansible_facts['device_links']` fact includes all the links available for each storage device. If you have multipath devices, you can use this to help determine which devices are alternative paths to the same storage device, or are multipath devices.

The following playbook gathers and displays the `ansible_['device_links']` fact for all managed hosts.

```yaml
---
- name: Gather device link facts
  hosts: all

  tasks:
    - name: Display device link facts
      ansible.builtin.debug:
        var: ansible_facts['device_links']
```

##### Facts about Mounted File Systems

The `ansible_facts['mounts']` fact provides information about the currently mounted devices on the managed host. For each device, this includes the mounted block device, its file system's mount point, mount options, and so on.

The following playbook gathers and displays the `ansible_facts['mounts']` fact for managed hosts.

```yaml
---
- name: Gather mounts
  hosts: all

  tasks:
    - name: Display mounts facts
      ansible.builtin.debug:
        var: ansible_facts['mounts']
```

The fact contains a list of dictionaries for each mounted file system on the managed host.



**Table 9.8. Selected Variables from the Dictionary in a Mounted File System List Item**

| Variable          | Comments                                                     |
| :---------------- | :----------------------------------------------------------- |
| `mount`           | The directory on which this file system is mounted.          |
| `device`          | The name of the block device that is mounted.                |
| `fstype`          | The type of file system the device is formatted with (such as `xfs`). |
| `options`         | The current mount options in effect.                         |
| `size_total`      | The total size of the device.                                |
| `size_available`  | How much space is free on the device.                        |
| `block_size`      | The size of blocks on the file system.                       |
| `block_total`     | How many blocks are in the file system.                      |
| `block_available` | How many blocks are free in the file system.                 |
| `inode_available` | How many inodes are free in the file system.                 |

For example, you can determine the free space on the root (`/`) file system on each managed host with the following play:

```yaml
- name: Print free space on / file system
  hosts: all
  gather_facts: true 

  tasks:
    - name: Display free space
      ansible.builtin.debug:
        msg: >
          The root file system on {{ ansible_facts['fqdn'] }} has
          {{ item['block_available'] * item['block_size'] / 1000000 }}
          megabytes free. 
      loop: "{{ ansible_facts['mounts'] }}" 
      when: item['mount'] == '/' 
```

#### References

[mount - Control active and configured mount points — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html)

[Roles — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)

[Managing local storage using RHEL System Roles — Red Hat Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/administration_and_configuration_tasks_using_system_roles_in_rhel/managing-local-storage-using-rhel-system-roles_assembly_updating-packages-to-enable-automation-for-the-rhel-system-roles)

#### Example

Files:

```
[student@workstation system-storage]$ ll
total 804
-rw-r--r--. 1 student student    192 Oct  7 23:30 ansible.cfg
drwxrwxr-x. 2 student student     22 Oct  7 23:30 collections
-rw-r--r--. 1 student student   1186 Oct  7 23:30 get-storage.yml
-rw-r--r--. 1 student student     37 Oct  7 23:30 inventory
-rw-r--r--. 1 student student 808333 Oct  7 23:30 redhat-rhel_system_roles-1.19.3.tar.gz
[student@workstation system-storage]$ cat ansible.cfg 
[defaults]
remote_user=devops
inventory=./inventory
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become=yes
become_method=sudo

[student@workstation system-storage]$ cat get-storage.yml 
---
- name: View storage configuration
  hosts: webservers

  tasks:

    - name: Retrieve physical volumes
      ansible.builtin.command: pvs
      register: pv_output

    - name: Display physical volumes
      ansible.builtin.debug:
        msg: "{{ pv_output['stdout_lines'] }}"

    - name: Retrieve volume groups
      ansible.builtin.command: vgs
      register: vg_output

    - name: Display volume groups
      ansible.builtin.debug:
        msg: "{{ vg_output['stdout_lines'] }}"

    - name: Retrieve logical volumes
      ansible.builtin.command: lvs
      register: lv_output

    - name: Display logical volumes
      ansible.builtin.debug:
        msg: "{{ lv_output['stdout_lines'] }}"

    - name: Retrieve mounted logical volumes
      ansible.builtin.shell: "mount | grep lv"
      register: mount_output

    - name: Display mounted logical volumes
      ansible.builtin.debug:
        msg: "{{ mount_output['stdout_lines'] }}"

    - name: Retrieve /etc/fstab contents
      ansible.builtin.command: cat /etc/fstab
      register: fstab_output

    - name: Display /etc/fstab contents
      ansible.builtin.debug:
        msg: "{{ fstab_output['stdout_lines'] }}"

[student@workstation system-storage]$ cat inventory 
[webservers]
servera.lab.example.com


[student@workstation system-storage]$ ll collections/
total 0
```

Write a playbook to:

- Use the `/dev/vdb` device as an LVM physical volume, contributing space to the volume group `apache-vg`.
- Create two logical volumes named `content-lv` (64 MB in size) and `logs-lv` (128 MB in size), both backed by the `apache-vg` volume group.
- Create an XFS file system on both logical volumes.
- Mount the `content-lv` logical volume on the `/var/www` directory.
- Mount the `logs-lv` logical volume on the `/var/log/httpd` directory.

```yaml
---
- name: Managing storage via role
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: apache-vg
          type: lvm
          disks:
            - /dev/vdb
          volumes:
            - name: content-lv
              size: 64m
              fs_type: xfs
              mount_point: "/var/www"
              state: present
            
            - name: logs-lv
              size: 128m
              fs_type: xfs
              mount_point: "/var/log/httpd"
              state: present
```

Run the `get-storage.yml` playbook provided in the project directory to verify that the storage has been properly configured on the managed hosts in the `webservers` group. 

```
[student@workstation system-storage]$ ansible-navigator run -m stdout get-storage.yml 

PLAY [View storage configuration] **********************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Retrieve physical volumes] ***********************************************
changed: [servera.lab.example.com]

TASK [Display physical volumes] ************************************************
ok: [servera.lab.example.com] => {
    "msg": [
        "  PV         VG        Fmt  Attr PSize    PFree  ",
        "  /dev/vdb   apache-vg lvm2 a--  1020.00m 828.00m"
    ]
}

TASK [Retrieve volume groups] **************************************************
changed: [servera.lab.example.com]

TASK [Display volume groups] ***************************************************
ok: [servera.lab.example.com] => {
    "msg": [
        "  VG        #PV #LV #SN Attr   VSize    VFree  ",
        "  apache-vg   1   2   0 wz--n- 1020.00m 828.00m"
    ]
}

TASK [Retrieve logical volumes] ************************************************
changed: [servera.lab.example.com]

TASK [Display logical volumes] *************************************************
ok: [servera.lab.example.com] => {
    "msg": [
        "  LV         VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert",
        "  content-lv apache-vg -wi-ao----  64.00m                                                    ",
        "  logs-lv    apache-vg -wi-ao---- 128.00m                                                    "
    ]
}

TASK [Retrieve mounted logical volumes] ****************************************
changed: [servera.lab.example.com]

TASK [Display mounted logical volumes] *****************************************
ok: [servera.lab.example.com] => {
    "msg": [
        "/dev/mapper/apache--vg-content--lv on /var/www type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)",
        "/dev/mapper/apache--vg-logs--lv on /var/log/httpd type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)"
    ]
}

TASK [Retrieve /etc/fstab contents] ********************************************
changed: [servera.lab.example.com]

TASK [Display /etc/fstab contents] *********************************************
ok: [servera.lab.example.com] => {
    "msg": [
        "UUID=5e75a2b9-1367-4cc8-bb38-4d6abc3964b8\t/boot\txfs\tdefaults\t0\t0",
        "UUID=fb535add-9799-4a27-b8bc-e8259f39a767\t/\txfs\tdefaults\t0\t0",
        "UUID=7B77-95E7\t/boot/efi\tvfat\tdefaults,uid=0,gid=0,umask=077,shortname=winnt\t0\t2",
        "/dev/mapper/apache--vg-content--lv /var/www xfs defaults 0 0",
        "/dev/mapper/apache--vg-logs--lv /var/log/httpd xfs defaults 0 0"
    ]
}

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=11   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```



### Managing Network Configuration

#### Configuring Networking with the Network System Role

The `redhat.rhel_system_roles.network` system role provides a way to automate the configuration of network interfaces and network-related settings on Red Hat Enterprise Linux managed hosts.

This role supports the configuration of Ethernet interfaces, bridge interfaces, bonded interfaces, VLAN interfaces, MACVLAN interfaces, InfiniBand interfaces, and wireless interfaces.

The role is configured by using two variables: `network_provider` and `network_connections`.

```yaml
---
network_provider: nm
network_connections:
  - name: ens4
    type: ethernet
    ip:
      address:
        - 172.25.250.30/24
```

The `network_provider` variable configures the back-end provider, 

-  `nm` (NetworkManager) on Red Hat Enterprise Linux 7 and later.

- `initscripts` on Red Hat Enterprise Linux 6 systems in Extended Lifecycle Support (ELS),  that provider requires that the legacy `network` service is available on the managed hosts.

The `network_connections` variable configures the different connections. It takes as a value a list of dictionaries, each of which represents settings for a specific connection. Use the interface name as the connection name.

The following table lists the options for the `network_connections` variable.



**Table 9.9. Selected Options for the `network_connections` Variable**

| Option name        | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| `name`             | For NetworkManager, identifies the connection profile (the `connection.id` option). For `initscripts`, identifies the configuration file name (`/etc/sysconfig/network-scripts/ifcfg-*`name`*`). |
| `state`            | The runtime state of a connection profile. Either `up`, if the connection profile is active, or `down` if it is not. |
| `persistent_state` | Identifies if a connection profile is persistent. Either `present` if the connection profile is persistent (the default), or `absent` if it is not. |
| `type`             | Identifies the connection type. Valid values are `ethernet`, `bridge`, `bond`, `team`, `vlan`, `macvlan`, `infiniband`, and `wireless`. |
| `autoconnect`      | Determines if the connection automatically starts. Set to `yes` by default. |
| `mac`              | Restricts the connection to be used on devices with this specific MAC address. |
| `interface_name`   | Restricts the connection profile to be used by a specific interface. |
| `zone`             | Configures the `firewalld` zone for the interface.           |
| `ip`               | Determines the IP configuration for the connection. Supports the options `address` to specify a list of static IPv4 or IPv6 addresses on the interface, `gateway4` or `gateway6` to specify the IPv4 or IPv6 default router, and `dns` to configure a list of DNS servers. |

The `ip` variable in turn takes a dictionary of variables for its settings. Not all of these need to be used. A connection might just have an `address` setting with a single IPv4 address, or it might skip the address setting and have `dhcp4: yes` set to enable DHCPv4 addressing.



**Table 9.10. Selected Options for the `ip` Variable**

| Option name | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `address`   | A list of static IPv4 or IPv6 addresses and netmask prefixes for the connection. |
| `gateway4`  | Sets a static address of the default IPv4 router.            |
| `gateway6`  | Sets a static address of the default IPv6 router.            |
| `dns`       | A list of DNS name servers for the connection.               |
| `dhcp4`     | Use DHCPv4 to configure the interface.                       |
| `auto6`     | Use IPv6 autoconfiguration to configure the interface.       |

This is a minimal example `network_connections` variable to configure and immediately activate a static IPv4 address for the `enp1s0` interface:

```yaml
network_connections:
- name: enp1s0
  type: ethernet
  ip:
    address:
      - 192.0.2.25/24
  state: up
```

If you were dynamically configuring the interface using DHCP and SLAAC, you might use the following settings instead:

```yaml
network_connections:
- name: enp1s0
  type: ethernet
  ip:
    dhcp4: true
    auto6: true
  state: up
```

The next example temporarily deactivates an existing network interface:

```yaml
network_connections:
- name: enp1s0
  type: ethernet
  state: down
```

To delete the configuration for `enp1s0` entirely, you would write the variable as follows:

```yaml
network_connections:
- name: enp1s0
  type: ethernet
  state: down
  persistent_state: absent
```

The following example uses some of these options to set up the interface `eth0` with a static IPv4 address, set a static DNS name server, and place the interface in the `external` zone for `firewalld`:

```yaml
network_connections:
- name: eth0 
    persistent_state: present  
    type: ethernet  
    autoconnect: yes  
    mac: 00:00:5e:00:53:5d  
    ip:
      address:
        - 172.25.250.40/24  
      dns:
        - 8.8.8.8 
    zone: external  
```

The following example play sets `network_connections` as a play variable and then calls the `redhat.rhel_system_roles.network` role:

```yaml
- name: NIC Configuration
  hosts: webservers
  vars:
    network_connections:
      - name: ens4
        type: ethernet
        ip:
          address:
            - 172.25.250.30/24
  roles:
    - redhat.rhel_system_roles.network
```

You can specify variables for the network role with the `vars` clause, as in the previous example, as role variables. Alternatively, you can create a YAML file with those variables under the `group_vars` or `host_vars` directories, depending on your use case.

You can use this role to set up 802.11 wireless connections, VLANs, bridges, and other more complex network configurations. See the role's documentation for more details and examples.

#### Configuring Networking with Modules

In addition to the `redhat.rhel_system_roles.network` system role, Ansible includes modules that support the configuration of the hostname and firewall on a system.

The `ansible.builtin.hostname` module sets the hostname for a managed host without modifying the `/etc/hosts` file. This module uses the `name` parameter to specify the new hostname, as in the following task:

```yaml
- name: Change hostname
  ansible.builtin.hostname:
    name: managedhost1
```

The `ansible.posix.firewalld` module supports the management of `firewalld` on managed hosts.

This module supports the configuration of `firewalld` rules for services and ports. It also supports the zone management, including the association or network interfaces and rules to a specific zone.

The following task shows how to create a `firewalld` rule for the `http` service on the default zone (`public`). The following task configures the rule as permanent, and makes sure it is active:

```yaml
- name: Enabling http rule
  ansible.posix.firewalld:
    service: http
    permanent: true
    state: enabled
```

This following task configures the `eth0` in the `external` `firewalld` zone:

```yaml
- name: Moving eth0 to external
  ansible.posix.firewalld:
    zone: external
    interface: eth0
    permanent: true
    state: enabled
```

The following table lists some parameters for the `ansible.posix.firewalld` module.

| Parameter name | Description                                                  |
| :------------- | :----------------------------------------------------------- |
| `interface`    | Interface name to manage with `firewalld`.                   |
| `port`         | Port or port range. Uses the port/protocol or port-port/protocol format. |
| `rich_rule`    | Rich rule for `firewalld`.                                   |
| `service`      | Service name to manage with `firewalld`.                     |
| `source`       | Source network to manage with `firewalld`.                   |
| `zone`         | `firewalld` zone.                                            |
| `state`        | Enable or disable a `firewalld` configuration.               |
| `type`         | Type of device or network connection.                        |
| `permanent`    | Change persists across reboots.                              |
| `immediate`    | If the changes are set to `permanent`, then apply them immediately. |

#### Ansible Facts for Network Configuration

Ansible collects a number of facts that are related to each managed host's network configuration. For example, a list of the network interfaces on a managed host are available in the `ansible_facts['interfaces']` fact.

The following playbook gathers and displays the available interfaces for a host:

```yaml
---
- name: Obtain interface facts
  hosts: host.lab.example.com

  tasks:
    - name: Display interface facts
      ansible.builtin.debug:
        var: ansible_facts['interfaces']
```

The preceding playbook produces the following list of the network interfaces:

```
PLAY [Obtain interface facts] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [host.lab.example.com]

TASK [Display interface facts] *************************************************
ok: [host.lab.example.com] => {
    "ansible_facts['interfaces']": [
        "eth2",
        "eth1",
        "eth0",
        "lo"
    ]
}

PLAY RECAP *********************************************************************
host.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

The output in the previous example shows that four network interfaces are available on the `host.lab.example.com` managed host: `lo`, `eth2`, `eth1`, and `eth0`.

You can retrieve additional information about the configuration for a specific network interface from the `ansible_facts['*`NIC_name`*']` fact. For example, the following play displays the configuration for the `eth0` network interface by printing the value of the `ansible_facts['eth0']` fact.

```yaml
- name: Obtain eth0 facts
  hosts: host.lab.example.com

  tasks:
    - name: Display eth0 facts
      ansible.builtin.debug:
        var: ansible_facts['eth0']
```

The preceding playbook produces the following output:

```
PLAY [Obtain eth0 facts] *******************************************************

TASK [Gathering Facts] *********************************************************
ok: [host.lab.example.com]

TASK [Display eth0 facts] ******************************************************
ok: [host.lab.example.com] => {
    "ansible_facts['eth0']": {
        "active": true,
        "device": "eth0",
        "features": {
...output omitted...
        },
        "hw_timestamp_filters": [],
        "ipv4": {
            "address": "172.25.250.10",
            "broadcast": "172.25.250.255",
            "netmask": "255.255.255.0",
            "network": "172.25.250.0",
            "prefix": "24"
        },
        "ipv6": [
            {
                "address": "fe80::82a0:2335:d88a:d08f",
                "prefix": "64",
                "scope": "link"
            }
        ],
        "macaddress": "52:54:00:00:fa:0a",
        "module": "virtio_net",
        "mtu": 1500,
        "pciid": "virtio0",
        "promisc": false,
        "speed": -1,
        "timestamping": [],
        "type": "ether"
    }
}

PLAY RECAP *********************************************************************
host.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

The preceding output displays additional configuration details, such as the IP address configuration both for IPv4 and IPv6, the associated device, and the type of interface.

The following table lists some other useful network-related facts.

| Fact name                             | Description                                                  |
| :------------------------------------ | :----------------------------------------------------------- |
| `ansible_facts['dns']`                | A list of the DNS name server IP addresses and the search domains. |
| `ansible_facts['domain']`             | The subdomain for the managed host.                          |
| `ansible_facts['all_ipv4_addresses']` | All the IPv4 addresses configured on the managed host.       |
| `ansible_facts['all_ipv6_addresses']` | All the IPv6 addresses configured on the managed host.       |
| `ansible_facts['fqdn']`               | The fully qualified domain name (FQDN) of the managed host.  |
| `ansible_facts['hostname']`           | The unqualified hostname (the part of the hostname before the first period in the FQDN). |
| `ansible_facts['nodename']`           | The hostname of the managed host as reported by the system.  |

**Note**

Ansible also provides the `inventory_hostname` "magic variable" which includes the hostname as configured in the Ansible inventory file.

#### References

[Knowledgebase: Red Hat Enterprise Linux (RHEL) System Roles](https://access.redhat.com/articles/3050101)

[Linux System Roles](https://linux-system-roles.github.io/)

[linux-system-roles/network at GitHub](https://github.com/linux-system-roles/network)

[ansible.builtin.hostname Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/hostname_module.html)

[ansible.posix.firewalld Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html)

[nmcli usage](https://blog.csdn.net/2401_83621634/article/details/138295045)

#### Example

```
[student@workstation system-network]$ ll
total 804
-rw-r--r--. 1 student student    236 Oct  8 01:36 ansible.cfg
drwxrwxr-x. 2 student student     22 Oct  8 01:36 collections
-rw-r--r--. 1 student student    180 Oct  8 01:36 get-eth1.yml
-rw-r--r--. 1 student student     37 Oct  8 01:36 inventory
-rw-r--r--. 1 student student 808333 Oct  8 01:36 redhat-rhel_system_roles-1.19.3.tar.gz
[student@workstation system-network]$ cat ansible.cfg 
[defaults]
remote_user = devops
inventory = ./inventory
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
[student@workstation system-network]$ cat inventory 
[webservers]
servera.lab.example.com
[student@workstation system-network]$ 
[student@workstation system-network]$ ll collections/
total 0
[student@workstation system-network]$ cat get-eth1.yml 
---
- name: Obtain network info for webservers
  hosts: webservers

  tasks:

    - name: Display eth1 info
      ansible.builtin.debug:
        var: ansible_facts['eth1']['ipv4']
```

Install the `redhat.rhel_system_roles` Ansible Content Collection from the `redhat-rhel_system_roles-1.19.3.tar.gz` file to the `collections` directory in the project directory.

Create a playbook that uses the `redhat.rhel_system_roles.network` role to configure the network interface `eth1` on `servera.lab.example.com` with the `172.25.250.30/24` IP address and network prefix.

- Create a new variable file named `network_config.yml` in the `group_vars/webservers` directory to define the `network_connections` role variable for the `webservers` group.

- The value of that variable must configure a network connection for the `eth1` network interface that assigns it the static IP address and network prefix `172.25.250.30/24`.

```yaml
# network.yml
---
- name: Configure network via system role
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.network
    
    
# network_config.yml under group_vars/webservers
---
network_provider: nm
network_connections:
  - name: eth1
    type: ethernet
    ip:
      address:
        - 172.25.250.30/24
```

Check result:

```
[student@workstation system-network]$ ansible-navigator run -m stdout get-eth1.yml 

PLAY [Obtain network info for webservers] **************************************

TASK [Gathering Facts] *********************************************************
ok: [servera.lab.example.com]

TASK [Display eth1 info] *******************************************************
ok: [servera.lab.example.com] => {
    "ansible_facts['eth1']['ipv4']": {
        "address": "172.25.250.30",
        "broadcast": "172.25.250.255",
        "netmask": "255.255.255.0",
        "network": "172.25.250.0",
        "prefix": "24"
    }
}

PLAY RECAP *********************************************************************
servera.lab.example.com    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```



### Chapter 9 Example

Create playbooks for configuring a software repository, users and groups, logical volumes, cron jobs, and additional network interfaces on a managed host.

pre-requirement:

```
[student@workstation system-review]$ ll
total 800
-rw-r--r--. 1 student student    236 Oct  8 22:26 ansible.cfg
drwxrwxr-x. 2 student student     22 Oct  8 22:26 collections
-rw-r--r--. 1 student student     37 Oct  8 22:26 inventory
-rw-r--r--. 1 student student 808333 Oct  8 22:26 redhat-rhel_system_roles-1.19.3.tar.gz

[student@workstation system-review]$ cat ansible.cfg 
[defaults]
remote_user = devops
inventory = ./inventory
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
[student@workstation system-review]$ 
[student@workstation system-review]$ cat inventory 
[webservers]
serverb.lab.example.com

[student@workstation system-review]$ ll collections/
total 0
```

```yaml
[student@workstation system-review]$ cat repo_playbook.yml 
---
- name: Installing software from YUM
  hosts: webservers
  vars:
    package: rhelver
  tasks:
    - name: Configing the YUM repository and install packages
      ansible.builtin.yum_repository:
        file: example
        name: example-internal
        baseurl: http://materials.example.com/yum/repository
        description: Example Inc. Internal YUM repo
        enabled: true
        gpgcheck: true
        state: present

    - name: Ensure Repo RPM key is installed
      ansible.builtin.rpm_key:
        key: http://materials.example.com/yum/repository/RPM-GPG-KEY-example
        state: present

    - name: Installing the package
      ansible.builtin.dnf:
        name: "{{ package }}"
        state: present

    - name: Gather information about the package
      ansible.builtin.package_facts:
        manager: auto

    - name: Show the package information
      ansible.builtin.debug:
        var: ansible_facts['packages'][package]
      when: package in ansible_facts['packages']
```

```yaml
[student@workstation system-review]$ cat users.yml 
---
- name: Creating user group
  hosts: webservers
  vars:
    users:
      - username: ops1
        groups: webadmin
      - username: ops2
        groups: webadmin

  tasks:
    - name: Creating the spcific user group
      ansible.builtin.group:
        name: webadmin
        state: present

    - name: Adding users to group
      ansible.builtin.user:
        name: "{{ item['username'] }}"
        groups: "{{ item['groups'] }}"
        append: true
      loop: "{{ users }}"

    - name: Gather the result
      ansible.builtin.command: "grep webadmin /etc/group"
      register: result

    - name: Show the result
      ansible.builtin.debug:
        msg: "{{result['stdout_lines']}}"
```

```yaml
[student@workstation system-review]$ cat storage.yml 
---
- name: Configuring LVM using System Roles
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: apache-vg
          type: lvm
          disks:
            - /dev/vdb
          
          volumes:
            - name: content-lv
              size: 64m
              fs_type: xfs
              mount_point: "/var/www"
              state: present
            - name: logs-lv
              size: 128m
              fs_type: xfs
              mount_point: "/var/log/httpd"
              state: present

  post_tasks:
    - name: Gathering LVM vol information
      ansible.builtin.command: 
        cmd: grep lv /etc/fstab
      register: result

    - name: Show the result
      ansible.builtin.debug:
        msg: "{{ result['stdout_lines'] }}"
```

```yaml
[student@workstation system-review]$ cat create_crontab_file.yml 
---
- name: Creating cron jobs
  hosts: webservers

  tasks:
    - name: Scheduling a cron job
      ansible.builtin.cron:
        name: Check disk usage
        job: df >> /home/devops/disk_usage
        user: devops
        cron_file: disk_usage
        minute: "*/2"
        hour: 9-16
        weekday: 1-5

    - name: Gather result
      ansible.builtin.command:
        cmd: cat /etc/cron.d/disk_usage
      register: result

    - name: Show result
      ansible.builtin.debug:
        msg: "{{ result['stdout_lines'] }}"
```

```yaml
[student@workstation system-review]$ cat network_playbook.yml 
---
- name: Config NIC via system roles
  hosts: webservers
  gather_facts: true
 
  roles:
    - name: redhat.rhel_system_roles.network
      network_provider: nm
      network_connections:
        - name: eth1
          type: ethernet
          ip:
            address:
              - 172.25.250.40/24

  post_tasks:
    - name: Gathering netowrk information
      ansible.builtin.setup:
      
    - name: Show the result
      ansible.builtin.debug:
        var: ansible_facts['eth1']
```

#### Summary

- The `ansible.builtin.yum_repository` module configures a Yum repository on a managed host. For repositories that use public keys, you can verify that the key is available with the `ansible.builtin.rpm_key` module.

- The `ansible.builtin.user` and `ansible.builtin.group` modules create users and groups respectively on a managed host.

- The `ansible.builtin.known_hosts` module configures SSH known hosts for a server and the `ansible.posix.authorized_key` modules configures authorized keys for user authentication.

- The `ansible.builtin.cron` module configures system or user Cron jobs on managed hosts.

- The `ansible.posix.at` module configures One-off `at` jobs on managed hosts.

- The `redhat.rhel_system_roles` Red Hat Certified Ansible Content Collection includes two particularly useful system roles: `storage`, which supports the configuration of LVM logical volumes, and `network`, which enables the configuration of network interfaces and connections.

  

  

---

## Chapter 10. Comprehensive Review

Reviewing *Red Hat Enterprise Linux Automation with Ansible*

Before beginning the comprehensive review for this course, you should be comfortable with the topics covered in each chapter. Do not hesitate to ask the instructor for extra guidance or clarification on these topics.

### Summary

#### [Chapter 1, *Introducing Ansible*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch01)

Describe the fundamental concepts of Ansible and how it is used, and install development tools from Red Hat Ansible Automation Platform.

- Describe the motivation for automating Linux administration tasks with Ansible, fundamental Ansible concepts, and the basic architecture of Ansible.
- Install Ansible on a control node and describe the distinction between community Ansible and Red Hat Ansible Automation Platform.

#### [Chapter 2, *Implementing an Ansible Playbook*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch02)

Create an inventory of managed hosts, write a simple Ansible Playbook, and run the playbook to automate tasks on those hosts.

- Describe Ansible inventory concepts and manage a static inventory file.
- Describe where Ansible configuration files are located, how Ansible selects them, and edit them to apply changes to default settings.
- Write a basic Ansible Playbook and run it using the automation content navigator.
- Write a playbook that uses multiple plays with per-play privilege escalation, and effectively use automation content navigator to find new modules in available Ansible Content Collections and use them to implement tasks for a play.

#### [Chapter 3, *Managing Variables and Facts*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch03)

Write playbooks that use variables to simplify management of the playbook and facts to reference information about managed hosts.

- Create and reference variables that affect particular hosts or host groups, the play, or the global environment, and describe how variable precedence works.
- Encrypt sensitive variables using Ansible Vault, and run playbooks that reference Vault-encrypted variable files.
- Reference data about managed hosts using Ansible facts, and configure custom facts on managed hosts.

#### [Chapter 4, *Implementing Task Control*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch04)

Manage task control, handlers, and task errors in Ansible Playbooks.

- Use loops to write efficient tasks and use conditions to control when to run tasks.
- Implement a task that runs only when another task changes the managed host.
- Control what happens when a task fails, and what conditions cause a task to fail.

#### [Chapter 5, *Deploying Files to Managed Hosts*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch05)

Deploy, manage, and adjust files on hosts managed by Ansible.

- Create, install, edit, and remove files on managed hosts, and manage the permissions, ownership, SELinux context, and other characteristics of those files.
- Deploy files to managed hosts that are customized by using Jinja2 templates.

#### [Chapter 6, *Managing Complex Plays and Playbooks*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch06)

Write playbooks for larger, more complex plays and playbooks.

- Write sophisticated host patterns to efficiently select hosts for a play.
- Manage large playbooks by importing or including other playbooks or tasks from external files, either unconditionally or based on a conditional test.

#### [Chapter 7, *Simplifying Playbooks with Roles and Ansible Content Collections*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch07)

Use Ansible Roles and Ansible Content Collections to develop playbooks more quickly and to reuse Ansible code.

- Describe the purpose of an Ansible Role, its structure, and how roles are used in playbooks.
- Create a role in a playbook's project directory and run it as part of one of the plays in the playbook.
- Select and retrieve roles from external sources such as Git repositories or Ansible Galaxy, and use them in your playbooks.
- Obtain a set of related roles, supplementary modules, and other content from an Ansible Content Collection and use them in a playbook.
- Write playbooks that take advantage of system roles for Red Hat Enterprise Linux to perform standard operations.

#### [Chapter 8, *Troubleshooting Ansible*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch08)

Troubleshoot playbooks and managed hosts.

- Troubleshoot generic issues with a new playbook and repair them.
- Troubleshoot failures on managed hosts when running a playbook.

#### [Chapter 9, *Automating Linux Administration Tasks*](https://rol.redhat.com/rol/app/courses/rh294-9.0/pages/ch09)

Automate common Linux system administration tasks with Ansible.

- Subscribe systems, configure software channels and repositories, enable module streams, and manage RPM packages on managed hosts.
- Manage Linux users and groups, configure SSH, and modify Sudo configuration on managed hosts.
- Manage service startup, schedule processes with `at`, `cron`, and `systemd`, reboot managed hosts with `reboot`, and control the default boot target on managed hosts.
- Partition storage devices, configure LVM, format partitions or logical volumes, mount file systems, and add swap spaces.
- Configure network settings and name resolution on managed hosts, and collect network-related Ansible facts.



### LABs

#### Lab: Deploying Ansible

**Specifications**

- Install the automation content navigator on `workstation` so that it can serve as the control node. The Yum repository containing the package has been configured on `workstation` for you.
- Your Ansible project directory is `/home/student/review-cr1`.
- On the control node, create the `/home/student/review-cr1/inventory` inventory file. The inventory must contain a group called `dev` that consists of the `servera.lab.example.com` and `serverb.lab.example.com` managed hosts.
- Create an Ansible configuration file named `/home/student/review-cr1/ansible.cfg`. This configuration file must use the `/home/student/review-cr1/inventory` file as the project inventory file.
- Log in to your private automation hub at `utility.lab.example.com` from the command line before attempting to run automation content navigator, so that you can pull automation execution environment images from its container registry. Your username is `admin` and your password is `redhat`.
- Create a configuration file for automation content navigator named `/home/student/review-cr1/ansible-navigator.yml`. This configuration file must set the default automation execution environment image to `utility.lab.example.com/ee-supported-rhel8:latest`, and automation content navigator must only pull this image from the container repository if the image is missing on your control node.
- Create a playbook named `users.yml` in the project directory. It must contain one play that runs on managed hosts in the `dev` group. Its play must use one task to add the users `joe` and `sam` to all managed hosts in the `dev` group. Run the `users.yml` playbook and confirm that it works.
- Inspect the existing `packages.yml` playbook. In the play in that playbook, define a play variable named `packages` with a list of two packages as its value: `httpd` and `mariadb-server`. Run the `packages.yml` playbook and confirm that both of those packages are installed on the managed hosts on which the playbook ran.
- Add a task to the `packages.yml` playbook that installs the `redis` package if the total swap space on the managed host is greater than 10 MB. Run the `packages.yml` playbook again after adding this task.
- Troubleshoot the existing `verify_user.yml` playbook. It is supposed to verify that the `sam` user was created successfully, and it is not supposed to create the `sam` user if it is missing. Run the playbook with the `--check` option and resolve any errors. Repeat this process until you can run the playbook with the `--check` option and it passes, and then run the `verify_user.yml` playbook normally.

```
[student@workstation review-cr1]$ cat ansible.cfg 
[defaults]
inventory = ./inventory

[privilege_escalation]
become = true
become_method = sudo
become_user = root

[student@workstation review-cr1]$ cat ansible-navigator.yml 
---
ansible-navigator:
  execution-environment:
    image: utility.lab.example.com/ee-supported-rhel8:latest
    pull:
      policy: missing

  playbook-artifact:
    enable: true
  mode: stdout
  
[student@workstation review-cr1]$ cat inventory 
[dev]
servera.lab.example.com
serverb.lab.example.com
```

```yaml
# packages.yml 
---
- name: Install packages
  hosts: dev
  vars:
    packages:
      - httpd
      - mariadb-server
  tasks:
    - name: show the ansible_facts
      ansible.builtin.debug:
        var: ansible_facts['swaptotal_mb']

    - name: Install redis if swap space low
      ansible.builtin.dnf:
        name: redis
        state: present
      when: ansible_facts['swaptotal_mb'] > 10
    - name: Install the required packages
      ansible.builtin.dnf:
        name: "{{ packages }}"
        state: latest
```

```yaml
# users.yml
---
- name: Add users
  hosts: dev

  tasks:

    - name: Add the users joe and sam
      ansible.builtin.user:
        name: "{{ item }}"
      loop:
        - joe
        - sam
```

```yaml
# verify_user.yml 
---
- name: Verify the sam user was created
  hosts: dev

  tasks:

    - name: Verify the sam user exists
      ansible.builtin.user:
        name: sam
      check_mode: true
      register: sam_check

    - name: Sam was created
      ansible.builtin.debug:
        msg: "Sam was created"
      when: sam_check['changed'] == false

    - name: Output sam user status to file
      ansible.builtin.lineinfile:
        path: /home/student/verify.txt
        line: "Sam was created"
        create: true
      when: sam_check['changed'] == false
```



#### Lab: Creating Playbooks

**Specifications**

- Create the playbooks specified by this activity in the `/home/student/review-cr2` project directory.
- Create a playbook named `dev_deploy.yml` with one play that runs on the `webservers` host group (which contains the `servera.lab.example.com` and `serverb.lab.example.com` managed hosts). Enable privilege escalation for the play. Add the following tasks to the play:
  - Install the `httpd` package.
  - Start the `httpd` service and enable it to start on boot.
  - Deploy the `templates/vhost.conf.j2` template to `/etc/httpd/conf.d/vhost.conf` on the managed hosts. This task should notify the `Restart httpd` handler.
  - Copy the `files/index.html` file to the `/var/www/vhosts/*`hostname`*` directory on the managed hosts. Ensure that the destination directory is created if it does not already exist.
  - Configure the firewall to allow the `httpd` service.
  - Add a `Restart httpd` handler to the play that restarts the `httpd` service.
- Create a playbook named `get_web_content.yml` with one play named `Test web content` that runs on the `workstation` managed host. This playbook tests whether the `dev_deploy.yml` playbook was run successfully and ensures that the web server is serving content. Enable privilege escalation for the play. Structure the play as follows:
  - Create a `block` and `rescue` task named `Retrieve web content and write to error log on failure`.
  - Inside the `block`, create a task named `Retrieve web content` that uses the `ansible.builtin.uri` module to return content from `http://serverb.lab.example.com`. Register the results in a variable named `content`.
  - Inside the `rescue` clause, create a task named `Write to error file` that writes the value of the `content` variable to the `/home/student/review-cr2/error.log` file if the `block` fails. The task must create the `error.log` file if it does not already exist.
- Create a new `site.yml` playbook that imports the plays from both the `dev_deploy.yml` and the `get_web_content.yml` playbooks.
- After you have completed the rest of the specifications, run the `site.yml` playbook. Make sure that all three playbooks run successfully.

```jinja2
[student@workstation review-cr2]$ tree 
.
├── ansible.cfg
├── files
│   └── index.html
├── inventory
└── templates
    └── vhost.conf.j2

2 directories, 4 files
[student@workstation review-cr2]$ cat ansible.cfg 
[defaults]
remote_user = devops
inventory = ./inventory

[privilege_escalation]
become_user = root
become_method = sudo

[student@workstation review-cr2]$ cat files/index.html 
This is a test page.

[student@workstation review-cr2]$ cat inventory 
workstation

[webservers]
servera.lab.example.com
serverb.lab.example.com

[student@workstation review-cr2]$ cat templates/vhost.conf.j2 
# {{ ansible_managed }}

<VirtualHost *:80>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
    Options +Indexes +FollowSymlinks +Includes
    Order allow,deny
    Allow from all
    </Directory>
</VirtualHost>
```

```yaml
# dev_deploy.yml 
---
- name: Lab deploying playbook
  hosts: webservers
  become: true

  tasks:
    - name: Installing package
      ansible.builtin.dnf:
        name: httpd, firewalld
        state: present

    - name: Starting service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Deploy template
      ansible.builtin.template:
        src: templates/vhost.conf.j2
        dest: '/etc/httpd/conf.d/vhost.conf'
        owner: root
        group: root
        mode: 0644
      notify: 
        - Restart httpd
    
    - name: Ensure destination web directory is there
      ansible.builtin.file:
        name: "/var/www/vhosts/{{ ansible_facts['hostname'] }}"
        state: directory
        owner: root
        group: root
        mode: 0755

    - name: Copy index file
      ansible.builtin.copy:
        src: files/index.html
        dest: "/var/www/vhosts/{{ ansible_facts['hostname'] }}"
        owner: root
        group: root
        mode: 0644

    - name: Starting firewall
      ansible.builtin.service:
        name: firewalld
        state: started

    - name: Config firewall for apache
      ansible.posix.firewalld:
        service: http
        permanent: true
        immediate: true
        state: enabled

  handlers:
    - name: Restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
```

```yaml
# get_web_content.yml 
---
- name: Test web content
  hosts: webservers
  become: true

  tasks:
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://serverb.lab.example.com
            return_content: true
          register: content

        - name: Show the content
          ansible.builtin.debug:
            var: content

      rescue:
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/review-cr2/error.log
            line: "{{ content }}"
            create: true
            state: present
```

```yaml
# site.yml 
---
- name: Deploy web servers
  ansible.builtin.import_playbook: dev_deploy.yml
  
- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
```



#### Lab: Managing Linux Hosts and Using System Roles

```
[student@workstation review-cr3]$ ll
total 804
-rw-r--r--. 1 student student    192 Oct 10 23:43 ansible.cfg
drwxrwxr-x. 2 student student     22 Oct 10 23:43 collections
-rw-r--r--. 1 student student     37 Oct 10 23:43 inventory
-rw-r--r--. 1 student student    808 Oct 10 23:43 pass-vault.yml
-rw-r--r--. 1 student student 808333 Oct 10 23:43 redhat-rhel_system_roles-1.19.3.tar.gz
[student@workstation review-cr3]$ cat ansible.cfg 
[defaults]
remote_user=devops
inventory=./inventory
collections_paths=./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become=yes
become_method=sudo
[student@workstation review-cr3]$ tree collections/
collections/

0 directories, 0 files
[student@workstation review-cr3]$ 
[student@workstation review-cr3]$ cat inventory 
[webservers]
servera.lab.example.com
[student@workstation review-cr3]$ 
[student@workstation review-cr3]$ cat pass-vault.yml 
$ANSIBLE_VAULT;1.1;AES256
36343762643130643961343163303465373863653933333231343232663832326263613137383432
6263343264633238383561323863393431376365633634320a336338613362303664323934306662
38383632386633313562623738343764626563326137306235373464626163363436363037666432
3762343461653130310a663061363135383239313465306164626239616431646636316466636630
35663839336131636166306138626430623338663030306232383738386430333262663737353039
35663865343635383831386530383063646630653339373062353439373861613739626533303135
36356536363539376236663962366265663663313064343938326365383166623364646338363865
38623038323262636161393064373536613430613265336634633862323131316636646437396535
66373738343432353036633731636438653263383166616338343137346362646561383930396534
3961393030376565636631333730626666633362366565366438
```

```yaml
# storage.yml 
---
- name: Creating LVM via system roles
  hosts: webservers

  roles:
    - name: redhat.rhel_system_roles.storage
      storage_pools:
        - name: vg_web
          type: lvm
          disks:
            - /dev/vdb
          volumes:
            - name: lv_content
              size: 128m
              fs_type: xfs
              mount_point: '/var/www/html/content'
              state: present

            
            - name: lv_uploads
              size: 256m
              fs_type: xfs
              mount_point: '/var/www/html/uploads'
              state: present

  post_tasks:
    - name: Gather information of LVM
      ansible.builtin.command: cat /etc/fstab
      register: result

    - name: Show the result
      debug:
        var: result['stdout_lines']

# dev-users.yml 
---
- name: user management
  hosts: webservers
  vars_files:
    - pass-vault.yml

  tasks:
    
    - name: Create group if not exist
      ansible.builtin.group:
        name: webdev
        state: present

    - name: Create the developer if not exist
      ansible.builtin.user:
        name: developer
        state: present
        groups: webdev
        password: "{{ pwhash }}"


    - name: Manage Sudo for the group
      ansible.builtin.lineinfile:
        path: /etc/sudoers.d/webdev
        state: present
        create: true
        mode: 0440
        line: "%webdev ALL=(ALL) NOPASSWD: ALL"
        validate: /usr/sbin/visudo -cf %s

# network.yml 
---
- name: Config network via system roles
  hosts: webservers

  roles: 
    - name: redhat.rhel_system_roles.network
      network_provider: nm
      network_connections:
        - name: eth1
          type: ethernet
          ip:
            address:
              - 172.25.250.45/24


# log-rotate.yml 
---
- name: Cron job to rotate logs
  hosts: webservers

  tasks:
    - name: creating a cron job
      ansible.builtin.cron:
        name: rotating the httpd logs
        user: devops
        job: logrotate -f /etc/logrotate.d/httpd
        cron_file: rotate_web
        minute: 00
        hour: 00
        state: present

# site.yml 
---
- name: storage part
  ansible.builtin.import_playbook: storage.yml

- name: User part
  ansible.builtin.import_playbook: dev-users.yml

- name: network part
  ansible.builtin.import_playbook: network.yml

- name: log part
  ansible.builtin.import_playbook: log-rotate.yml
```



#### Lab: Creating Roles

- The `review-cr4` directory contains your Ansible project for this activity.

- Convert the `ansible-httpd.yml` playbook in the project directory into a new Ansible role named `ansible-httpd`. The new role must be created in the `/home/student/review-cr4/roles/ansible-httpd` directory.

- Copy any variables, tasks, templates, files, and handlers that were used in or by the playbook into the appropriate files or directories in the new role. Copy the playbook variables to the `roles/ansible-httpd/defaults/main.yml` file.

- Update the `meta/main.yml` file in the role with the following content:

  | Variable    | Value                  |
  | :---------- | :--------------------- |
  | author      | Red Hat Training       |
  | description | example role for RH294 |
  | company     | Red Hat                |
  | license     | BSD                    |

- Edit the `roles/ansible-httpd/README.md` file so that it provides the following information about the role:

  ```
  ansible-httpd
  =========
  Example ansible-httpd role from "Red Hat Enterprise Linux Automation with Ansible" (RH294)
  
  Role Variables
  --------------
  
  * `web_package`: the RPM package
  * `web_service`: the systemd service
  * `web_config_file`: the path to the main configuration file
  * `web_root`: the path to an index.html file
  * `web_fw_service`: the name of a firewalld service
  
  Dependencies
  ------------
  
  None.
  
  Example Playbook
  ----------------
  
      - hosts: servers
        roles:
          - ansible-httpd
  
  License
  -------
  
  BSD
  
  Author Information
  ------------------
  
  Red Hat (training@redhat.com)
  ```

- Remove any unused directories and files within the role.

- In the project directory, write a `site.yml` playbook that runs the new `ansible-httpd` role on the managed hosts in the `webdev` inventory group.

- Run the `site.yml` playbook.

```yaml
[student@workstation review-cr4]$ ll
total 12
-rw-r--r--. 1 student student  147 Oct 11 04:14 ansible.cfg
-rw-r--r--. 1 student student 1256 Oct 11 04:14 ansible-httpd.yml
drwxrwxr-x. 2 student student   24 Oct 11 04:14 files
-rw-r--r--. 1 student student   57 Oct 11 04:14 inventory
drwxrwxr-x. 2 student student   27 Oct 11 04:14 templates

[student@workstation review-cr4]$ cat ansible.cfg 
[defaults]
remote_user=devops
inventory=./inventory


[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[student@workstation review-cr4]$ cat inventory 
[webdev]
serverb.lab.example.com
serverc.lab.example.com

[student@workstation review-cr4]$ cat ansible-httpd.yml 
----
---
- name: HTTPD server is installed
  hosts: webdev
  vars:
    web_package: httpd
    web_service: httpd
    web_config_file: /etc/httpd/conf/httpd.conf
    web_root: /var/www/html/index.html
    web_fw_service: http

  tasks:
    - name: Packages are installed
      ansible.builtin.dnf:
        name: "{{ web_package }}"
        state: present

    - name: Ensure service is started
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: started
        enabled: true

    - name: Deploy configuration file
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: "{{ web_config_file }}"
        owner: root
        group: root
        mode: '0644'
        setype: httpd_config_t
      notify: Restart httpd

    - name: Deploy index.html file
      ansible.builtin.copy:
        src: files/index.html
        dest: "{{ web_root }}"
        owner: root
        group: root
        mode: '0644'

    - name: Web port is open
      ansible.posix.firewalld:
        service: "{{ web_fw_service }}"
        permanent: true
        state: enabled
        immediate: true

  handlers:
    - name: Restart httpd
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: restarted

[student@workstation review-cr4]$ tree files/
files/
└── index.html

0 directories, 1 file
[student@workstation review-cr4]$ cat files/index.html 
This website was deployed with Ansible

[student@workstation review-cr4]$ tree templates/
templates/
└── httpd.conf.j2

0 directories, 1 file
```

Creating the roles folder

```
[student@workstation review-cr4]$ tree roles/
roles/
└── ansible-httpd
    ├── ansible-navigator.log
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── site-artifact-2024-10-11T10:24:41.361686+00:00.json
    ├── tasks
    │   └── main.yml
    └── templates
        └── httpd.conf.j2
```

```yaml
# roles/ansible-httpd/defaults/main.yml 
---
# defaults file for ansible-httpd
web_package: httpd
web_service: httpd
web_config_file: /etc/httpd/conf/httpd.conf
web_root: /var/www/html/index.html
web_fw_service: http
```

```yaml
# roles/ansible-httpd/handlers/main.yml 
---
# handlers file for ansible-httpd
- name: Restart httpd
  ansible.builtin.service:
    name: "{{ web_service }}"
    state: restarted
```

```yaml
# roles/ansible-httpd/tasks/main.yml 
---
# tasks file for ansible-httpd

- name: Packages are installed
  ansible.builtin.dnf:
    name: "{{ web_package }}"
    state: present

- name: Ensure service is started
  ansible.builtin.service:
    name: "{{ web_service }}"
    state: started
    enabled: true

- name: Deploy configuration file
  ansible.builtin.template:
    src: templates/httpd.conf.j2
    dest: "{{ web_config_file }}"
    owner: root
    group: root
    mode: '0644'
    setype: httpd_config_t
  notify: Restart httpd

- name: Deploy index.html file
  ansible.builtin.copy:
    src: files/index.html
    dest: "{{ web_root }}"
    owner: root
    group: root
    mode: '0644'

- name: Web port is open
  ansible.posix.firewalld:
    service: "{{ web_fw_service }}"
    permanent: true
    state: enabled
    immediate: true
```

```jinja2
# roles/ansible-httpd/templates/httpd.conf.j2 
# Ansible managed
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/var/www/html"
<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
<Files ".ht*">
    Require all denied
</Files>
ErrorLog "logs/error_log"
LogLevel warn
<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
</IfModule>
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
<IfModule mime_module>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>
AddDefaultCharset UTF-8
<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>
EnableSendfile on
IncludeOptional conf.d/*.conf
```

```yaml
# roles/ansible-httpd/meta/main.yml 
galaxy_info:
  author: Red Hat Training
  description: example role for RH294
  company: Red Hat

  # If the issue tracker for your role is not on github, uncomment the
  # next line and provide a value
  # issue_tracker_url: http://example.com/issue/tracker

  # Choose a valid license ID from https://spdx.org - some suggested licenses:
  # - BSD-3-Clause (default)
  # - MIT
  # - GPL-2.0-or-later
  # - GPL-3.0-only
  # - Apache-2.0
  # - CC-BY-4.0
  license: BSD (GPL-2.0-or-later, MIT, etc)

  min_ansible_version: 2.1

  # If this a Container Enabled role, provide the minimum Ansible Container version.
  # min_ansible_container_version:

  #
  # Provide a list of supported platforms, and for each platform a list of versions.
  # If you don't wish to enumerate all versions for a particular platform, use 'all'.
  # To view available platforms and versions (or releases), visit:
  # https://galaxy.ansible.com/api/v1/platforms/
  #
  # platforms:
  # - name: Fedora
  #   versions:
  #   - all
  #   - 25
  # - name: SomePlatform
  #   versions:
  #   - all
  #   - 1.0
  #   - 7
  #   - 99.99

  galaxy_tags: []
    # List tags for your role here, one per line. A tag is a keyword that describes
    # and categorizes the role. Users find roles by searching for tags. Be sure to
    # remove the '[]' above, if you add tags to this list.
    #
    # NOTE: A tag is limited to a single word comprised of alphanumeric characters.
    #       Maximum 20 tags per role.

dependencies: []
  # List your role dependencies here, one per line. Be sure to remove the '[]' above,
  # if you add dependencies to this list.
```

```yaml
# site.yml 
---
- name: Running the roles created
  hosts: webdev

  roles:
    - name: ansible-httpd
```



---

Completion time: 2024/10/11

---

