# CentOS Server Template

[1] Use the Minimal ISO as the basis for this template. You can download it from https://www.centos.org/download/.

[2] Use any virtualization tool to instantiate an OS instance from the iso. Accept all defaults.

[3] Apply system updates:

```shell
su -c 'yum update'
```

[4] Configure ```ssh``` support as needed.

[5] Install any special software needed to install additional components.

[6] Define any special userids or other configuration settings you want to be present on all deployable instances that will be based on this template.
