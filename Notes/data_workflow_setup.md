# Nanopore data workflow setup

In some cases, there are separate HPC nodes for storage and compute (GPU), so and so these tasks will be fulfilled by different nodes. Set up two user(s) with the same UID on the SDN and the compute node. One for _data_ and one for _basecalling_. In the case of a two-node layout, you can share the same home directory by mounting and giving the same directory under the ```-m``` flag.

```sudo useradd -m -d /mnt/path_to_home/ -u 8888 basecalling```
```sudo useradd -m -d /mnt/path_to_home/ -u 9999 data```

Ensure users on each node can write to to their counterpart directories on each node. It is important to note that the basecalling user should only be able to read data owned by the _data_ user, and not modify it. The _data_ user will only be accessible to sysadmin and the scripts responsible for syncing dat between SAM and SDN.

Next, we need to make sure the SAM and the SDN can talk to eachother, so we will exchange SSH public keys.

```ssh-copy-id data@SDN ```

Now, we set up the sync to run every hour from the SAM to the _data_ account on the SDN.

```crontab -e```
And add the line:
```0 * * * * /usr/bin/rsync -rv --progress --log-file=/path/to/log_file-$(date +%Y-%m-%d)  /path/to/nanopore_data_SAM/ user@remote-machine:/path/to/nanopore_data_SDN/```

Finally, we need a script to periodically purge the SAM of the sequencing data, only after having confirmed it has been uploaded. Here, I have opted for this script to be executed by the user, however it could be run automatically.

```/usr/bin/rsync --remove-source-files -rv --progress --log-file=/path/to/log_file_purge-$(date +%Y-%m-%d)  /path/to/nanopore_data_SAM/ user@remote-machine:/path/to/nanopore_data_SDN/```



## Fix for MinKNOW read/write problems

Make a new user group where you can add both the minknow user and the current user
```sudo groupadd nanopore_data```
Add both users to the _nanopore_data_ user group
```sudo usermod -aG nanopore_data nanopore_SAM```
```sudo usermod -aG nanopore_data minknow```

Change the mount point to _nanopore_data_ ownership
```sudo chgrp nanopore_data /media/nanopore_data_SAM```
Mount the external drive, specifying the _nanopore_data_ group ID.
```sudo mount -o gid=nanopore_data /dev/sda1 ./nanopore_SAM```

Ensure the drive is always remounted with the correct GID
```sudo nano /etc/fstab```
And add the line:
```/dev/sda1 /media/nanopore exfat umask=gid=nanopore_data```







