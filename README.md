# Openshift Metric using GlusterFS Persistent Volume
- I am using this case Openshift Enterprise 3.5 and GlusterFS 3.2 Container Native Storage.
- My glusterfs services running on pods.In this enable Cluster Metric existing Openshift Cluster.


##### GlusterFS Preperation For Cluster Metric
-  Before you have deploy cluster metric on existing Openshift Cluster you are prepare glusterfs Persistent backend.Because Openshift Enterprise 3.5 ansible playbook only install Metric Cluster on existing Persistent volume.If you deploy first Metric cluster there will a a few issue creating cassandra persistent database.Whereas glusterfs volume security at first cassandra pod won't running state.
###### Start with deploy Cluster metric using ansible playbook.

If you have not openshift-ansible playbook. you can get it from Github from version 3.5

`# git clone https://github.com/openshift/openshift-ansible.git`


`# ansible-playbook /path/to/openshift-ansible/playbooks/byo/openshift-cluster/openshift-metrics.yml -e openshift_metrics_install_metrics=True -e openshift_metrics_cassandra_storage_type=pv  -e openshift_metrics_install_hawkular_agent=True`

You can also look at exstra variable from Openshift Documentation.
`<link>`: https://docs.openshift.com/container-platform/3.5/install_config/cluster_metrics.html

After deployment complete cassandra pod will not running state.
At this point we have to prepare glusterfs persistent backend.
-  Create gluster end point

`oc create -f sample-gluster-endpoints-hawkular.yaml`
- After create ep you have to create services

`oc create -f sample-gluster-service-hawkular.yaml`

After It's done we have to get fsgroup from hawkular pod as below.Because hawkular application has to required permssion to write to the glusterfs persistent Volume

`# export GID=$(oc get po --selector="hawkular-cassandra-1-txzks=openshift-infra" -o go-template --template='{{printf "%.0f" ((index .items 0).spec.securityContext.fsGroup)}}')`

###### NOTE: if you can not get fsGroup as above command you can manualy find it `oc get -o pod hawkular-cassandra-1-txzks -o yaml | grep fsGroup`

- Create Glusterfs volume for casssandra pod

`# heketi-cli volume create --size=10 --name=gluster-hawkular-volume --gid=${GID}`

- After create gluster volume persistent volume as below.

###### NOTE: You have to replace labels, endpoints and path

`# oc create -f pvhawkular.json`

- After it you have to create PVC and likewise you have to replace labels

`oc create -f hawkular-pvc.yaml`

Finaly your cassandra pod running if it is not you can delete it and replication controlller recreate it another node
