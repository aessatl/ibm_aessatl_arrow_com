# v0.01 20210125 from windows to build HCI three node CentOS 8 Streams Cluster with Openshift HA Cluster
# This project is for use by Jeremey Wise and other CDW resources.
#
# Goal:
# 1) Useing USB key and adding Kickstart file as only touch of systems. Deploy OS and add to Ansible Control.  No shell login
# 2) Structure that provides deployment for NSPOF OpenShift Cluster.  VM initially, but nodes can be replaced later on as physical without rebuild.
# 3) Option to deploy OS OS4.x  and OKD to match end user or demo needs related to version and support needs
# 4) Provide two spaces:  Production and Staging (for test and devops )
# 5) Provide storate and networking in a production class standard such that it can scale without rebuild. High performance and modular as core to design for storage needs.
# 6) Design that is portable so site can with minimal changes and site resource prep, deploy: 
# 7) Modular authentication to site systems and storage. To enable enterprise account and security managment. Intially LDAP into Microsoft AD as that is the most prevelent in datcenters.

#########
# Deployment / Site Customization tasks
# If you are cloning this and need to modify to your site host and VM name convention or other required site dependancies to use this playbook
1) Physical System / Resources: Below are details of what it is built on and where you can modifiy to relfect your site details
    a) 3 x Intel servers: Dual Socket E5-5xxx CPU, 128GB RAM, 3 x SSD class drives.  2 x 1Gb NIC, 2 x 10Gb NIC
    b) OS Setup:  Cento OS 8 Streams. USB install with Kickstart with Cockpit + oVirt + Gluster. 1Gb for OVS.  10Gb for Gluster

2) Site Requirements before starting OCP build
    a) Identity Manager: "FreeIPA" on two node cluster for DDNS LDAP Kerberos, DHCP
    b) IPLB node(s) will be needed later on for applications and is expected.
    c) High speed disk LAN. 10Gb or better stronly recommended for disk / dedicated LAN side of the cluster nodes. Today the design uses gluster but in future other options such as CEPH or SpectrumScale will be built into design.

3) Openshift VM based deployment from oVirt.  Cluster Instance based on variable deployment
    a) 1x bastion node (2cpu x 4GB RAM, 50GB Drive)
    a) 3x master nodes(8GB RAM 2CPU, 50GB Drive)
    c) 3x worker nodes(12GB RAM 2CPU,50GB Drive)

############## Folder and File Explaination ##########
README_CLUSTER_DEVOPS -> your hear already :)... get to know what your getting into
/vars -> variable files for global environment
/group_vars -> folder to host site and all system variables not specific to OS and Infrastructure (see host_vars for that). Variables for each "role" grouped
/host_vars -> variables for physical hosts and OCP cluster deployments 
/library -> Not sure.. but per best practices this is for system libraries if and as needed
/module_utils -> folder for storing custom modules or utilities or binaries needed during deployment
/monitoring -> how to setup system and cluster monitoring or configuration related to event and log managment during build
/okd -> Deployment details of deveopment Openshift cluster instance. This will leverage much of base deployment but have OKD specifics
/openshift_4 -> Deployment details of latest stable branch for OCP. Files and details about version 4.x deployment
/openshift_clusters -> Parent folder for each deployment iteration. Ex: okd, os01, os02 each reflect independant clusters. Many sites use names such as "Production" "Staging" "UAT" "DR"
/roles -> Definitions of a physical or virtual system with its setup and configuration with the total cluster. Broken out by common task or resource setting common configuration.



## Deployment Steps:
1)) Using external System download clone of github https://github.com/penguinpages/cluster_devops to some local directory
2) Customize all varible files in /group_vars and /host_vars
    a) /group_vars/var/vars_global.yml -> file for site wide variables. Details on how to do this are in that folder's README
    b) /group_vars/vault/*  -> each file is decrypted.  But you need to set for site and then run encyption. Details on how to do this are in that folder's README
    c) /host_vars/* -> each file represents the cluster types as well as hypervisor node(s) it will deploy against.  Details on how to do this are in that folder's README
3) Setup CentOS 8 Streams on Physical nodes
    a) Download and build USB boot key
    b) Create customize kickstart file for each host and use as deployment made
    c) Connect nodes under ansible managment 
    d) Deploy base configuration roles on ech node for HCI deployment under oVirt
    e) Layout Gluster and OVS
    f) Build Template VM for CentOS 8 Streams
4) Build out Infrastructrure VMs
    a) Deploy two VMs for Cluster role. Ex: ns01, ns02
    b) Install and configure IDM cluster. Ex: FreeIPA on both nodes with DDNS, DHCP, IPLB, LDAP, Kerberos
    c) Configure DNS, DHCP, LDAP, Kerberos and IPLB for NSPOF 
    d) Populate DNS for OCP clsuters.
4) Build Bastion VM
    a) Deploy VM for OCP cluster deployment Ex: ansible
    b) Bring node under ansible managment
    c) Stage deployment account and directories
5) Deploy OCP cluster
    a) Using bastion node ingite cluster with 3 x master 3 x worker inventory_iplb_servers
    b) Bring cluster under ansible managment
    c) Add persistent volume claims for cluster via gluster or NFS hosted by Gluster 
6) Enjoy HA NSPOF Openshift cluster!!


########## Current Issues / To do List ############
# 
# <ya.. too many to itemize as for now it is WIP>

##########  Change control ########## 
# <ya.. too many to itemize as for now it is WIP>

########## Ideas or features for next version ##########
1) CEPH and Spectrum Scale vs gluster
2) Hypervisor VMWare vs KVM
3) IPLB nodes deployed as part of cluster build
4) Document process to replace each virtual node with physical as moving into production would need more ROI from hardware at high speed.
5) Image repository




# Misc Notes and Documentation
##########
# Directory Tree with Annotation done every major release or change if paths change significantly and need notes.
.
├── filter_plugins
│   └── README
├── group_vars
│   ├── vars
│   │   ├── inventory_iplb_servers.yml
│   │   ├── inventory_ldap_servers.yml
│   │   ├── inventory_name_servers.yml
│   │   ├── inventory_syslog_servers.yml
│   │   └── README
│   └── vault
│       ├── README
│       ├── vault_cluster
│       ├── vault_github
│       ├── vault_ldap
│       ├── vault_rhn
│       ├── vault_root
│       └── vault_SAN
├── host_vars
│   ├── inventory
│   ├── inventory_hypervisor_cluster.yml
│   ├── inventory_okd_cluster.yml
│   ├── inventory_os2_cluster.yml
│   ├── inventory_os_cluster.yml
│   ├── inventory_test.yml
│   ├── README
│   └── vars_global.yml
├── library
│   └── README
├── module_utils
│   └── README
├── monitoring
│   └── README
├── okd
│   ├── cluster_configure_playbook.yml
│   ├── cluster_provision_playbook.yml
│   └── README
├── openshift_311
│   ├── cluster_configure_playbook.yml
│   ├── cluster_provision_playbook.yml
│   └── README
├── openshift_4
│   ├── cluster_configure_playbook.yml
│   ├── cluster_provision_playbook.yml
│   └── README
├── production
│   └── README
├── README_CLUSTER_DEVOPS
├── roles
│   ├── common
│   │   ├── defaults
│   │   │   └── README
│   │   ├── files
│   │   │   └── README
│   │   ├── handlers
│   │   │   └── README
│   │   ├── library
│   │   │   └── README
│   │   ├── lookup_plugins
│   │   │   └── README
│   │   ├── meta
│   │   │   └── README
│   │   ├── module_utils
│   │   │   └── README
│   │   ├── README
│   │   ├── role_network.yml
│   │   ├── tasks
│   │   │   └── README
│   │   ├── templates
│   │   │   └── README
│   │   └── vars
│   │       ├── README
│   │       ├── vars_global_site.yml
│   │       ├── vars_production_site.yml
│   │       └── vars_staging_site\ copy.yml
│   ├── README
│   ├── role_docker_disk_setup.yml
│   ├── role_docker_setup.yml
│   ├── role_gluster_psc_disk_setup.yml
│   ├── role_gluster_psc_setup.yml
│   ├── role_gluster_registry_storge.yml
│   ├── role_kvm_docker_disk_add.yml
│   ├── role_kvm_new_vm.yml
│   ├── role_kvm_psc_disk_add.yml
│   ├── role_ldap_binding.yml
│   ├── role_ping_nodes.yml
│   ├── role_psc_gluster.yml
│   ├── role_registeration_centos.yml
│   ├── role_registeration_rhel.yml
│   ├── role_registeration_rhev.yml
│   ├── role_selinux_disable.yml
│   ├── role_set_ip_vm.yml
│   ├── role_smb_sw2_shares.yml
│   ├── role_ssh_passwordless_login.yml
│   ├── role_swap_disable.yml
│   ├── role_update_all_reboot.yml
│   ├── role_user_mgmt.yml
│   └── role_vdo_setup.yml
└── staging
    └── README

# Pull from Openshift best practices for Ansible deployment others have done
##############################################
## Global varibles I should always referance when possible : https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
##############################################
### Directory Tree for this git repo is per recomemndations below.  This is WIP!.

# This section is based on recommended directory layout per site: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
# Updated 20200103
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/               # https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
    vars/                 # Used for group variables
        group1.yml        # here we assign variables to particular groups
        group2.yml
    vaults/               # Used for secure group variables.You should adjust the variables in the vars file to point to the matching vault_ variables using jinja2 syntax, and ensure that the vault file is vault encrypted.

host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
