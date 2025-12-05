# CREATE A PUREDISK VOLUME POOL on NetBackup

To create a PureDisk storage server and volume pool (disk pool) in Veritas NetBackup, follow these steps using the command line:

## Create the PureDisk Storage Server
Use the nbdevconfig command to create the storage server. Specify the storage server name, the type (PureDisk), and the media server that will host it.
```bash
/usr/openv/netbackup/bin/admincmd/nbdevconfig -creatests -storage_server <storage_server_name> -stype PureDisk -media_server <media_server_name>
```

## Add Credentials for the Storage Server
Configure the credentials (username and password) for the storage server using the tpconfig command.
```bash
/usr/openv/volmgr/bin/tpconfig -add -storage_server <storage_server_name> -stype PureDisk -sts_user_id <username> -password <password>
```

## Initialize the PureDisk Volume (Optional - for local storage)
If using local disk, create a configuration file (e.g., /tmp/purediskVolume.conf) with the necessary parameters like storagepath, spapasswd, and dbpath.
Then, apply this configuration using nbdevconfig:
```bash
/usr/openv/netbackup/bin/admincmd/nbdevconfig -setconfig -stype PureDisk -storage_server <storage_server_name> -configlist /tmp/purediskVolume.conf
```

## Create the Disk Pool (Volume Pool)
First, generate a list of the volumes (disk volumes) that will be part of the disk pool using the nbdevconfig -previewdv command. This command lists the volumes available for the specified storage server.
```bash
/usr/openv/netbackup/bin/admincmd/nbdevconfig -previewdv -storage_servers <storage_server_name> -stype PureDisk | grep <lsu_name> > /tmp/dvlist
```

Replace <storage_server_name> and <lsu_name> with the appropriate names. The output is saved to a file (/tmp/dvlist). Next, use the nbdevconfig -createdp command to create the disk pool, referencing the list of volumes.
```bash
/usr/openv/netbackup/bin/admincmd/nbdevconfig -createdp -dp <disk_pool_name> -stype PureDisk -dvlist /tmp/dvlist -storage_servers <storage_server_name>
```

Replace <disk_pool_name> and <storage_server_name> with your actual names.

## Create a Storage Unit (STU)
Finally, create a storage unit that uses the newly created disk pool. Use the bpstuadd command.
```bash
/usr/openv/netbackup/bin/admincmd/bpstuadd -label <storage_unit_label> -dp <disk_pool_name> -host <media_server_name> -cj <max_jobs> -mfs <max_fragment_size_KB>
```

Replace <storage_unit_label>, <disk_pool_name>, <media_server_name>, <max_jobs>, and <max_fragment_size_KB> with your specific values.
