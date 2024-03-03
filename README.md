## Custom Docker iptables Handling

Docker is an indispensable tool for container orchestration, but its default behavior can be overly invasive, especially regarding how it interacts with the host system's iptables. By default, Docker assumes priority over the host's network stack, adding numerous iptables rules and using `iptables -I` (insert) to enforce this. This behavior can disrupt carefully configured network policies by placing Docker's rules at the top of iptables chains, potentially overriding existing rules set by the system administrator.

To address this and ensure that custom iptables rules always take precedence, I have modified Docker's libnetwork source code. Specifically, I've replaced all instances of `iptables -I` with `iptables -A` within the source. This alteration changes Docker's approach from inserting rules at the beginning of iptables chains to appending them at the end. This ensures that Docker's rules do not override the administrator's configurations, providing a more respectful coexistence with the host system's network stack.

This version of Docker is maintained in my private OneDev repository and will be mirrored here, tailored for environments where precise control over network policies is paramount.

Usage:
```
git clone
cd MutinyOnTheDocker
make
mv /usr/bin/dockerd /usr/bin/xdocker (backup)
cp bundles/binary/dockerd /usr/bin/dockerd
iptables -F && iptables -X && iptables -t nat -F && ./yourfirewallscript.sh && systemctl restart docker
confirm behavior with: iptables -L -nv && iptables -t nat -L -nv
```

## Before
```
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0            
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0            
    0     0 ACCEPT     all  --  *      docker_gwbridge  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker_gwbridge  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker_gwbridge !docker_gwbridge  0.0.0.0/0            0.0.0.0/0           
  822 70029 ACCEPT     all  --  *      br-16b2a696f08f  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    5   300 DOCKER     all  --  *      br-16b2a696f08f  0.0.0.0/0            0.0.0.0/0           
  833  709K ACCEPT     all  --  br-16b2a696f08f !br-16b2a696f08f  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  br-16b2a696f08f br-16b2a696f08f  0.0.0.0/0            0.0.0.0/0           
61323  135M ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
49401 2978K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0                    
    0     0 DROP       all  --  docker_gwbridge docker_gwbridge  0.0.0.0/0            0.0.0.0/0           
    0     0 PORT_FWD   all  --  ens3   *       0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
```

## After
```
Chain FORWARD (policy ACCEPT 1 packets, 176 bytes)
 pkts bytes target     prot opt in     out     source               destination         
 7117 1093K PORT_FWD   all  --  ens3   *       0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
 4576  671K ACCEPT     all  --  virbr0 ens3    0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
 7117 1093K ACCEPT     all  --  ens3   virbr0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED /* I_D_TS */
  148 21238 ACCEPT     all  --  virbr0 virbr0  0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
    0     0 log_kvm_DEBUG  all  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
    0     0 log_kvm_DEBUG  all  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            /* I_D_TS */
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

TODO: Change Docker iptables rule ordering for due dilligence since rule ordering is important. Should be as easy as adding a numerical value after -I.
Although I haven't noticed any issues with this configuation (yet).



The Moby Project
================

![Moby Project logo](docs/static_files/moby-project-logo.png "The Moby Project")

Moby is an open-source project created by Docker to enable and accelerate software containerization.

It provides a "Lego set" of toolkit components, the framework for assembling them into custom container-based systems, and a place for all container enthusiasts and professionals to experiment and exchange ideas.
Components include container build tools, a container registry, orchestration tools, a runtime and more, and these can be used as building blocks in conjunction with other tools and projects.

## Principles

Moby is an open project guided by strong principles, aiming to be modular, flexible and without too strong an opinion on user experience.
It is open to the community to help set its direction.

- Modular: the project includes lots of components that have well-defined functions and APIs that work together.
- Batteries included but swappable: Moby includes enough components to build fully featured container systems, but its modular architecture ensures that most of the components can be swapped by different implementations.
- Usable security: Moby provides secure defaults without compromising usability.
- Developer focused: The APIs are intended to be functional and useful to build powerful tools.
They are not necessarily intended as end user tools but as components aimed at developers.
Documentation and UX is aimed at developers not end users.

## Audience

The Moby Project is intended for engineers, integrators and enthusiasts looking to modify, hack, fix, experiment, invent and build systems based on containers.
It is not for people looking for a commercially supported system, but for people who want to work and learn with open source code.

## Relationship with Docker

The components and tools in the Moby Project are initially the open source components that Docker and the community have built for the Docker Project.
New projects can be added if they fit with the community goals. Docker is committed to using Moby as the upstream for the Docker Product.
However, other projects are also encouraged to use Moby as an upstream, and to reuse the components in diverse ways, and all these uses will be treated in the same way. External maintainers and contributors are welcomed.

The Moby project is not intended as a location for support or feature requests for Docker products, but as a place for contributors to work on open source code, fix bugs, and make the code more useful.
The releases are supported by the maintainers, community and users, on a best efforts basis only, and are not intended for customers who want enterprise or commercial support; Docker EE is the appropriate product for these use cases.

-----

Legal
=====

*Brought to you courtesy of our legal counsel. For more context,
please see the [NOTICE](https://github.com/moby/moby/blob/master/NOTICE) document in this repo.*

Use and transfer of Moby may be subject to certain restrictions by the
United States and other governments.

It is your responsibility to ensure that your use and/or transfer does not
violate applicable laws.

For more information, please see https://www.bis.doc.gov

Licensing
=========
Moby is licensed under the Apache License, Version 2.0. See
[LICENSE](https://github.com/moby/moby/blob/master/LICENSE) for the full
license text.
