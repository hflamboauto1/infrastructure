# Infrastructure

This is the configuration for the Brandfolder infrastructure. Built and managed
with [Terraform](github.com/hashicorp/terraform).

## Prerequisites

* Install Terraform
* Sign up for [Atlas](https://atlas.hashicorp.com/) and get invited to the Brandfolder organization.
* Find your Atlas API access key, you will need it next.
* Run the following command:
  `terraform remote config -backend-config="access_token={{Atlas Access Key}}" -backend-config="name=brandfolder/infrastructure"`
* (Optional) Set your environment variables to include
  ```
  export AWS_ACCESS_KEY_ID={{Your AWS Access Key}}
  export AWS_SECRET_ACCESS_KEY={{Your AWS Secret Access Key}}
  ```

## Usage

### Viewing changes

`make plan`

### Applying changes

`make apply`

## Workflow

1. Make changes to terraform files
2. Run `make plan` to view changes and check for errors
3. Commit changes
4. Run `make apply`

## Infrastructure Map

```
___                  _  __     _    _           ___       __             _               _
| _ )_ _ __ _ _ _  __| |/ _|___| |__| |___ _ _  |_ _|_ _  / _|_ _ __ _ __| |_ _ _ _  _ __| |_ _  _ _ _ ___                                               Jason Waldrip
| _ \ '_/ _` | ' \/ _` |  _/ _ \ / _` / -_) '_|  | || ' \|  _| '_/ _` (_-<  _| '_| || / _|  _| || | '_/ -_)                                       CTO, Brandfolder.com
|___/_| \__,_|_||_\__,_|_| \___/_\__,_\___|_|   |___|_||_|_| |_| \__,_/__/\__|_|  \_,_\__|\__|\_,_|_| \___|                              jason.waldrip@brandfolder.com

╔═Legend══════════════════════════════════════════════════════════╗
║ ┌─────────────┐                       ┌──────────────────────┐  ║
║ ▥Talks to ETCD│  ◀═══Networking═══▶   │//// Server Group ////│  ║           bastion.brandfolder.host                   *.brandfolder.com  *.brandfolder.ninja
║ └─────────────┘                       └──────────────────────┘  ║                      ║                                     ║      ║        ║    ║     ║
╚═════════════════════════════════════════════════════════════════╝                     SSH                                  HTTP   HTTPS    HTTP HTTPS  GIT
                                                                          ╔═══════════════▼════════════════╗                ╔═══▼══════▼═══╗  ╔═▼════▼═════▼═╗         
┌─────────────────────────────────────────────────────────────────────────╣            Bastion             ╠────────────────╣   Site ELB   ╠──╣   Dev ELB    ╠────────┐
│Amazon VPC        ┌─────────┐ ┌─────────────────────────┐                ╚════════════════════════════════╝                ╚══════════════╝  ╚══════════════╝        │
│                  │   RDS   │ │  ElasticCache (Redis)   │                                ║                                        ║              ║      ║            │
│                  └─────────┘ └─────────────────────────┘                        ╔══════SSH═══════╗                             HTTP           HTTP    GIT           │
│                                                                        ╔════════╝       ║        ╚════════╗                      ║              ║      ║            │
│                  ┌─────────────────────────────────────┐      ╔════════╝┌───────────────▼────────────────┐╚════════╗      ┌──────▼──────────────▼──────▼───┐        │
│                  │//////////////   Core   ////////////◀╬══════╝         │/////////// Workers  ///////////│         ╚══════╬▶////////// Routers  ///////////│        │
│    ┌─────────────┴─────────────────────────────────────┴────────────────┴────────────────────────────────┴────────────────┴────────────────────────────────┴───┐    │
│    │             │///┌────────┐/┌────────┐/┌────────┐//│                │┌────────┐/┌────────┐/┌────────┐│                │┌────────┐/┌────────┐/┌────────┐│   │    │
│    │             │///│        │/│        │/│        │//│                ││ Worker │/│ Worker │/│ Worker ││                ││        │/│        │/│        ││   │    │
│    │             │///│ Core 1 │/│ Core 3 │/│ Core 5 │//│                ││ (prod) │/│ (prod) │/│ (prod) ││                ││ Router │/│ Router │/│ Router ││   │    │
│    │             │///│        │/│        │/│        │//│                ││        │/│        │/│        ││                ││        │/│        │/│        ││   │    │
│    │ Servers     │///└────────┘/└────────┘/└────────┘//◀══════VPC═══════▶└────────┘/└────────┘/└────────┘◀══════VPC═══════▶└────────┘/└────────┘/└────────┘│   │    │
│    │             │////////┌────────┐//┌────────┐///////│   Networking   │┌────────┐//////////////////////│   Networking   │////////////////////////////////│   │    │
│    │             │////////│        │//│        │///////│                ││ Worker │//                  //│                │///////                  ///////│   │    │
│    │             │////////│ Core 2 │//│ Core 4 │///////│                ││ (feat) │//  autoscales...   //│                │///////  autoscales...   ///////│   │    │
│    │             │////////│        │//│        │///////│                ││        │//                  //│                │///////                  ///////│   │    │
│    │             │////////└────────┘//└────────┘///////│                │└────────┘//////////////////////│                │////////////////////////////////│   │    │
│    ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤    │
│    │              ETCD ◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀ ETCD Proxy ◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀◀ ETCD Proxy     │    │
│    ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤    │
│    │             │//////┌──────────┐//┌──────────┐/////│                │//┏━━━━━━━━━━━┓/////////////////│                │////////////////////////////////│   │    │
│    │             │//////│   Deis   │//│   Deis   │/////│                │//┃    App    ┃//┌────────────┐/│                │////////////////////////////////│   │    │
│    │             │//////│ Builder  │//│Controller│/////│                │//┗━━━━━━━━━━━┛//▥ Publisher  │/│                │////////////////////////////////│   │    │
│    │             │//////└──────────┘//└──────────┘/////│                │//┏━━━━━━━━━━━┓//└────────────┘/│                │////////////////////////////////│   │    │
│    │             │//////┌──────────┐//┌──────────┐/////│                │//┃    App    ┃//┌────────────┐/│                │////////┌─────────────┐/////////│   │    │
│    │             │//////▥  SkyDNS  │//│ Elastic  │/////◀════Flannel═════▶//┗━━━━━━━━━━━┛//▥ Registrator│/◀════Flannel═════▶////////▥ deis-router │/////////│   │    │
│    │ Containers  │//////│          │//│  Search  │/////│   Networking   │//┏━━━━━━━━━━━┓//└────────────┘/│   Networking   │////////└─────────────┘/////////│   │    │
│    │             │//////└──────────┘//└──────────┘/////│                │//┃    App    ┃//┌────────────┐/│                │////////////////////////////////│   │    │
│    │             │//////┌──────────┐//┌──────────┐/////│                │//┗━━━━━━━━━━━┛//│  logspout  │/│                │////////////////////////////////│   │    │
│    │             │//////▥   Reg-   │//│ Logspout │/////│                │//┏━━━━━━━━━━━┓//└────────────┘/│                │////////////////////////////////│   │    │
│    │             │//////│ istrator │//│          │/////│                │//┃    App    ┃/////////////////│                │////////////////////////////////│   │    │
│    │             │//////└──────────┘//└──────────┘/////│                │//┗━━━━━━━━━━━┛/////////////////│                │////////////////////////////////│   │    │
│    │             │/////////////////////////////////////│                │////////////////////////////////│                │////////////////////////////////│   │    │
│    └─────────────┬─────────────────────────────────────┬────────────────┬────────────────────────────────┬────────────────┬────────────────────────────────┬───┘    │
│    ┌─────────────┴─────────────────────────────────────┴────────────────┴────────────────────────────────┴────────────────┴────────────────────────────────┴───┐    │
│    │             │/┌──────┐┌───────┐┌───────┐┌───────┐/│                │/┌───────┐/┌───────┐/┌───────┐//│                │/┌───────┐/┌───────┐/┌───────┐//│   │    │
│    │  Services   │/│ Etcd ││Flannel││ Fleet ││Docker │/│                │/│Flannel│/│Docker │/│ Fleet │//│                │/│Flannel│/│Docker │/│ Fleet │//│   │    │
│    │             │/└──────┘└───────┘└───────┘└───────┘/│                │/└───────┘/└───────┘/└───────┘//│                │/└───────┘/└───────┘/└───────┘//│   │    │
│    └─────────────┬─────────────────────────────────────┬────────────────┬────────────────────────────────┬────────────────┬────────────────────────────────┬───┘    │
│                  └─────────────────────────────────────┘                └────────────────────────────────┘                └────────────────────────────────┘        │
│                                                                                                                                                                     │
│                                                                                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
