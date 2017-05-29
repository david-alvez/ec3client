# ec3client
ec3client automated build

This repository contains the ec3 cluster configuration tool, including all the templates.

You should refer to the 

You can pull eubrabigsea/ec3client Docker image (docker pull eubrabigsea/ec3client), or you can use the templates available in the git source code repository. To run it, we suggest to use the following commands:

    $ docker pull eubrabigsea/ec3client
    $ docker run -d -i -t --name mybigseaclucli eubrabigsea/ec3client /bin/bash
    $ docker exec -i -t mybisgseaclulci /bin/bash

You can use our own IM service instance (default and recommended) or deploy your own instance by using the Docker image of IM Server:

    docker run -d -p 8899:8899 -p 8800:8800 --name im grycap/im

Then you need to add the parameter -u localhost:8899 to the execution of the ec3 client commands.
Customizing it for your own infrastructure

The documentation for the ec3 tool is available in readthedocs.

To customize it to your infrastructure, you must edit the site description file (e.g. ubuntu-one.radl):

    disk.0.image.url = 'Virtual_Image_Identifier' and
    disk.0.os.credentials.username = 'existing_user_in_the_image' and
    disk.0.os.credentials.password = 'valid_passwd_for_the_user'

And you must create an auth file with the following information (refer to D3.3 for more information).

    type = InfrastructureManager ; username = yourIMusername ; password = anypassword
    id = myiaas ; type = IAASTYPE ; host = host:port ; username = youriaasuser ; password = youriaaspass

The number of data nodes and working nodes is defined in the same file

    system wn (
        ec3_max_instances = 3 and # maximum number of data nodes in the cluster
        ...
        ec3_if_fail = 'wnmesos'
    )

    system wnmesos (
        ec3_max_instances = 5 and # maximum number of working nodes in the cluster
        ...
    )

Finally, the name of the nodes from the cluster may need to be edited if you want to deploy multiple clusters in the same cloud IaaS. First, the information about the front end is in the file mesos.radl:

    system front (
    cpu.count>=1 and
    memory.size>=4g and
    net_interface.0.connection = 'private' and
    net_interface.0.dns_name = 'InternalServerName' and
    net_interface.1.connection = 'public' and
    net_interface.1.dns_name = 'PublicServerName' and
    queue_system = 'mesos' and
    ec3_templates contains 'mesos'
    )

And the information about the agents is on the files

    clues2.radl

    configure front (
    @begin
    ...
        roles:
          - { role: 'grycap.clues', auth: '{{AUTH}}',
                                    clues_queue_system: '{{QUEUE_SYSTEM}}',
                                    max_number_of_nodes: '{{ NNODES }}',
                                    vnode_prefix: 'bsagent' }
    @end
    )

    mesos.radl

    system front (
        cpu.count>=1 and
        memory.size>=4g and
        net_interface.0.connection = 'private' and
        net_interface.0.dns_name = 'bigseaserver' and
        net_interface.1.connection = 'public' and
        net_interface.1.dns_name = 'bigseapublic' and
        queue_system = 'mesos' and
        ...
    )

    configure front (
    ... 
        principal: 'myprincipal'
        secret: 'mysecret'
    ...
        - { role: 'grycap.mesos', mesos_type_of_node: 'front',
                                  max_number_of_nodes: '{{NNODES}}',
                                  vnode_prefix: 'bsagent',
                                  principal: 'myprincipal',
                                  secret: 'mysecret' }
        - { role: 'grycap.clues', clues_queue_system: '{{QUEUE_SYSTEM}}',
                                  auth: '{{AUTH}}',
                                  mesos_servername: 'bigseapublic',
                                  extract_proxy_file: '{{EXTRACT_PROXY_FILE}}',
                                  ec3_max_instances: '{{NNODES}}',
                                  vnode_prefix: 'bsagent' }
    )

    configure wnmesos (
    @begin
    ---
      ...
        roles:
          - { role: 'grycap.mesos', mesos_type_of_node: 'wn',
                                    max_number_of_nodes: '{{NNODES}}',
                                    vnode_prefix: 'bigseawn',
                                    principal: 'ubuntu',
                                    secret: 'ubuntusecret'}
        ...
    )

    configure wn (
    @begin
    ---
      ...
        roles:
          - { role: 'grycap.mesos', mesos_type_of_node: 'wn',
                                    max_number_of_nodes: '{{NNODES}}',
                                    vnode_prefix: 'bigseawn',
                                    principal: 'ubuntu',
                                    secret: 'ubuntusecret'}
    @end
    )

    nfs.radl

    configure front (
    @begin
      - roles:
        - { role: 'grycap.nfs', nfs_mode: 'front', nfs_exports: [{path: "/home", export: "bsagent*.localdomain(fsid=0,rw,async,no_root_squash,no_subtree_check,insecure)"}] }
    @end
    )


MONASCA Configuration.

You need to provide the necessary following vars:

  keystone_ip_address: 'xxx.xxx.xxx.xxx'
  mysql_host: '127.0.0.1' (or an equivalent one if different)
  mysql_root_pass: 'My mysql root pass'
  os_username: 'admin'
  os_password: 'password for admin'
  os_project_name: 'admin'
  os_mon_agent_username: 'monasca-agent'
  os_mon_agent_password: 'password for monasca agent'
  os_mon_project_name: 'mini-mon'
  os_mon_username: 'mini-mon'
  os_mon_password: 'mini mon password'
  monasca_system_shell: '/bin/false'
  monasca_system_user_name: 'monasca'
  monasca_system_user_home: '/var/lib/{{ monasca_system_user_name }}'


Launching the cluster

You can boot up the cluster by means of the following command inside the container. You can skuip the creation of users so you will have only the 'ubuntu' user created.

    ./ec3 launch -g -a /root/auth.dat LARGEBIGS ubuntu-ramses mesos marathon chronos spark2 hadoop nfs bigseausers monasca

After a while, a full cluster will be available.



