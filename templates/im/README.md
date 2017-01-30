
Configuration for scaling up/down Nodenames and Datanodes separatelly.

Supported by CLUES (automatically and manually) and by IM (manually)

Infrastructures deployed directly by IM
Manually deploying the three components (https://github.com/eubr-bigsea/ec3client/tree/master/templates/im  – generated and edited from the dryrun option of ec3 CLI)

$ im_cli.py create hadoop_deployMaster.radl add_hadoop_wn.radl add_hadoop_nodemanager.radl -a auth.dat

It will return an infrastructure Identifier (infid).
Adding resources (Datanodes or NodeManagers) 

$ im_cli.py addresource infid add_hadoop_wn.radl -a auth.dat
$ im_cli.py addresource infid add_hadoop_nodemanager.radl -a auth.dat

Removing resources (specific VM obtained from getinfo)

$ im_cli.py getinfo infId –a auth.dat
$ im_cli.py removeresource infid vmId -a auth.dat

