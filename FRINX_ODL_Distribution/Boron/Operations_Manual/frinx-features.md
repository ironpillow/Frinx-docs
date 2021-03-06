[Documentation main page](https://frinxio.github.io/Frinx-docs/)
[Operations Manual main page](https://frinxio.github.io/Frinx-docs/FRINX_ODL_Distribution/Boron/operations_manual.html)

<!-- TOC -->

- [FRINX ODL Distribution: Features](#frinx-odl-distribution-features)
    - [To install features](#to-install-features)
        - [To install for the current karaf session only](#to-install-for-the-current-karaf-session-only)

<!-- /TOC -->

# FRINX ODL Distribution: Features

The FRINX distribution offers the following features:

     odlparent
     yangtools
     mdsal
     controller (without xsql)
     netconf
     aaa
     dlux
     topoprocessing
     snmp
     openflowjava
     openflowplugin
     neutron
     sfc
     ovsdb
     gbp
     l2switch
     bgpcep
     lispflowmapping
     cli
     daexim
     faas    
     genius
     netvirt
     honeycomb-vbd
     l2vpn
     l3vpn

## To install features
### To install for the current karaf session only
To view a list of available features: Within the karaf console type

    feature:list

To search for a particular feature e.g. restconf you can use grep e.g.

    feature:list |grep restconf

To install a feature:

    feature:install odl-restconf
    feature:install odl-netconf-connector-all

Multiple features can be installed on a single line - use a space to separate e.g.:

    feature:install odl-restconf odl-netconf-connector-all


