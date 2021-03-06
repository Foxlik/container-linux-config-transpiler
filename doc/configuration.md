# Configuration Specification #

The configuration is a YAML document conforming to the following specification, with **_italicized_** entries being optional:

* **_ignition_** (object): metadata about the configuration itself.
  * **_config_** (objects): options related to the configuration.
    * **_append_** (list of objects): a list of the configs to be appended to the current config.
      * **source** (string): the URL of the config. Supported schemes are http and https. Note: When using http, it is advisable to use the verification option to ensure the contents haven't been modified.
      * **_verification_** (object): options related to the verification of the config.
        * **_hash_** (object): the hash of the config
          * **_function_** (string): the function used to hash the config. Supported functions are sha512.
          * **_sum_** (string): the resulting sum of the hash applied to the contents.
    * **_replace_** (object): the config that will replace the current.
      * **source** (string): the URL of the config. Supported schemes are http and https. Note: When using http, it is advisable to use the verification option to ensure the contents haven't been modified.
      * **_verification_** (object): options related to the verification of the config.
        * **_hash_** (object): the hash of the config
          * **_function_** (string): the function used to hash the config. Supported functions are sha512.
          * **_sum_** (string): the resulting sum of the hash applied to the contents.
* **_storage_** (object): describes the desired state of the system's storage devices.
  * **_disks_** (list of objects): the list of disks to be configured and their options.
    * **device** (string): the absolute path to the device. Devices are typically referenced by the `/dev/disk/by-*` symlinks.
    * **_wipe_table_** (boolean): whether or not the partition tables shall be wiped. When true, the partition tables are erased before any further manipulation. Otherwise, the existing entries are left intact.
    * **_partitions_** (list of objects): the list of partitions and their configuration for this particular disk.
      * **_label_** (string): the PARTLABEL for the partition.
      * **_number_** (integer): the partition number, which dictates it's position in the partition table (one-indexed). If zero, use the next available partition slot.
      * **_size_** (integer): the size of the partition (in sectors). If zero, the partition will fill the remainder of the disk.
      * **_start_** (integer): the start of the partition (in sectors). If zero, the partition will be positioned at the earliest available part of the disk.
      * **_type_guid_** (string): the GPT [partition type GUID][part-types]. If omitted, the default will be 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem data).
  * **_raid_** (list of objects): the list of RAID arrays to be configured.
    * **name** (string): the name to use for the resulting md device.
    * **level** (string): the redundancy level of the array (e.g. linear, raid1, raid5, etc.).
    * **devices** (list of strings): the list of devices (referenced by their absolute path) in the array.
    * **_spares_** (integer): the number of spares (if applicable) in the array.
  * **_filesystems_** (list of objects): the list of filesystems to be configured and/or used in the "files" section. Either "mount" or "path" needs to be specified.
    * **_name_** (string): the identifier for the filesystem, internal to Ignition. This is only required if the filesystem needs to be referenced in the "files" section.
    * **_mount_** (object): contains the set of mount and formatting options for the filesystem. A non-null entry indicates that the filesystem should be mounted before it is used by Ignition.
      * **device** (string): the absolute path to the device. Devices are typically referenced by the `/dev/disk/by-*` symlinks.
      * **format** (string): the filesystem format (ext4, btrfs, or xfs).
      * **_create_** (object): contains the set of options to be used when creating the filesystem. A non-null entry indicates that the filesystem shall be created.
        * **_force_** (boolean): whether or not the create operation shall overwrite an existing filesystem.
        * **_options_** (list of strings): any additional options to be passed to the format-specific mkfs utility.
    * **_path_** (string): the mount-point of the filesystem. A non-null entry indicates that the filesystem has already been mounted by the system at the specified path. This is really only useful for "/sysroot".
  * **_files_** (list of objects): the list of files, rooted in this particular filesystem, to be written.
    * **filesystem** (string): the internal identifier of the filesystem. This matches the last filesystem with the given identifier.
    * **path** (string): the absolute path to the file.
    * **_contents_** (object): options related to the contents of the file.
      * **_inline_** (string): the contents of the file.
      * **_remote_** (object): options related to the fetching of remote file contents.
        * **_compression_** (string): the type of compression used on the contents (null or gzip)
        * **_url_** (string): the URL of the file contents. Supported schemes are http and [data][rfc2397]. Note: When using http, it is advisable to use the verification option to ensure the contents haven't been modified.
        * **_verification_** (object): options related to the verification of the file contents.
          * **_hash_** (object): the hash of the config
            * **_function_** (string): the function used to hash the config. Supported functions are sha512.
            * **_sum_** (string): the resulting sum of the hash applied to the contents.
    * **_mode_** (integer): the file's permission mode.
    * **_user_** (object): specifies the file's owner.
      * **_id_** (integer): the user ID of the owner.
    * **_group_** (object): specifies the group of the owner.
      * **_id_** (integer): the group ID of the owner.
* **_systemd_** (object): describes the desired state of the systemd units.
  * **_units_** (list of objects): the list of systemd units.
    * **name** (string): the name of the unit. This must be suffixed with a valid unit type (e.g. "thing.service").
    * **_enable_** (boolean): whether or not the service shall be enabled. When true, the service is enabled. In order for this to have any effect, the unit must have an install section.
    * **_mask_** (boolean): whether or not the service shall be masked. When true, the service is masked by symlinking it to `/dev/null`.
    * **_contents_** (string): the contents of the unit.
    * **_dropins_** (list of objects): the list of drop-ins for the unit.
      * **name** (string): the name of the drop-in. This must be suffixed with ".conf".
      * **_contents_** (string): the contents of the drop-in.
* **_networkd_** (object): describes the desired state of the networkd files.
  * **_units_** (list of objects): the list of networkd files.
    * **name** (string): the name of the file. This must be suffixed with a valid unit type (e.g. "00-eth0.network").
    * **_contents_** (string): the contents of the networkd file.
* **_passwd_** (object): describes the desired additions to the passwd database.
  * **_users_** (list of objects): the list of accounts to be added.
    * **name** (string): the username for the account.
    * **_password_hash_** (string): the encrypted password for the account.
    * **_ssh_authorized_keys_** (list of strings): a list of SSH keys to be added to the user's authorized_keys.
    * **_create_** (object): contains the set of options to be used when creating the user. A non-null entry indicates that the user account shall be created.
      * **_uid_** (integer): the user ID of the new account.
      * **_gecos_** (string): the GECOS field of the new account.
      * **_home_dir_** (string): the home directory of the new account.
      * **_no_create_home_** (boolean): whether or not to create the user's home directory.
      * **_primary_group_** (string): the name or ID of the primary group of the new account.
      * **_groups_** (list of strings): the list of supplementary groups of the new account.
      * **_no_user_group_** (boolean): whether or not to create a group with the same name as the user.
      * **_no_log_init_** (boolean): whether or not to add the user to the lastlog and faillog databases.
      * **_shell_** (string): the login shell of the new account.
  * **_groups_** (list of objects): the list of groups to be added.
    * **name** (string): the name of the group.
    * **_gid_** (integer): the group ID of the new group.
    * **_password_hash_** (string): the encrypted password of the new group.

[part-types]: http://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs
[rfc2397]: https://tools.ietf.org/html/rfc2397
