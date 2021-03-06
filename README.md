# CI-RPM-Building #

## Why we need ci-rpm-building?

Is [Jenkins](http://jenkins-ci.org/) not enough to build RPMs automatically?
No, it isn't.

-  If you need build RPMs on a koji that jenkins cannot login in.
-  If you need build yum repos for these packages.

## Install and Configure [CI-RPM-Building](https://github.com/xning/ci-rpm-building)

### What do we need?

You should think about carefully about the following questions.

1. The [koji](https://fedorahosted.org/koji/) command need source RPMs/SCM URLs to
build RPMs. So you need first make a descion how to prepare source RPMs or SCM URLs,
and how [koji](https://fedorahosted.org/koji/) can get the source RPMs and SCM URLs.

2. To login in the [koji](https://fedorahosted.org/koji/) building server need
authentication and authorization.

3. Where, when and who issue an event that triger RPMs building process.

Our solutions

1. We use [fedpkg](https://fedorahosted.org/koji/) and rpmbuild tools to build SCMs
and source RPMs.

2. We use a [koji](https://fedorahosted.org/koji/) system that uses
[Kerberos](http://web.mit.edu/kerberos/) authentication. It's OK if we just exec
koji commands locally, just kinit is OK. If we need delegate koji job to a remote
server, we use ssh to forward a forwardable ticket to the remote server. Then
use sftp to upload our scripts to the remote server, and use ssh to exec the
script. Surely the remote server runs ssh daemon and the ssh daemon uses
[Kerberos](http://web.mit.edu/kerberos/) authentication.

3. We could use cron to run scheduled jobs or git hooks to issue the events.

### How setup [CI-RPM-Building](https://github.com/xning/ci-rpm-building)

1. Setting up ssh daemon that use [Kerberos](http://web.mit.edu/kerberos/) authentication.

2. Installing and configuring koji on the server.

3. Packages that we need

    yum -y install krb5-workstation perl-Authen-Krb5

### How to use [CI-RPM-Building](https://github.com/xning/ci-rpm-building)?

1. Prepare source RPM or scm url

2. Issue the event that will triger the building job.

## Functions in functions

### Notice

Now in the following functions, the argurment valid_time all could be replaced by
time literals that GNU date could accept.

For example, you can check whether the tgt is valid in future hours like this

     is_tgt_valid_after '+2 hour'

Or just simply

     is_tgt_valid_after '-2 hour'


Now these functions only work on [Bash](http://www.gnu.org/software/bash/).

### Functions to control [Kerberos](http://web.mit.edu/kerberos/) authentication

    tgt_config [valid_time [user [krb5_realm]]]]

This should be the first functions that you should exec before the following functions

    is_krb5_full_principal [string]
    get_krb5_default_realm
    get_krb5_cache_file_name [user_name or kerb principal]
    is_tgt_forwardable [user_name or kerb principal]
    is_tgt_valid_after [valid_time [user_name or kerb principal]]
    is_tgt_ok [valid_time [user_name or kerb principal]]
    
### Functions to work with [ssh/sftp](http://www.openssh.com/)

    ssh_config [[host] [[user] [worddir [hash_known_host]]]]
    
This should be the first functions that you should exec before the following functions.

    is_ssh_ok
    ssh_upload [file_path1 [file_path2 [...]]]
    ssh_download [file_path1 [file_path2 [...]]]
    ssh_exec [command1 [command2 [...]]]

### Functions for koji

    koji_config [build_tag [result_tag [get_raw_data [how_new_RPMs_we_want]]]]
    
You should exec koji_config first before the following functions

    have_koji_to_login
    get_pkgs_owned_by [user_name [pkg]]
    get_rpm_from_pkg pkg
    get_rpm_info_idx rpm
    get_pkgs_info_idx pkg1 [pkg2 [...]]


## About [GNU date](http://www.gnu.org/s/coreutils/)

### Some Exaples

We need UTC time in koji functions.

     # Get current UTC time
     date +%s

     # Covert UTC time to local time
     date -d @1403083279
     # A funny one
     date -d @$(date +%s)

     # second, minute, day, week, month and year.
     date -d '-1 day' +%s
     date -d '-1 week' +%s
     date -d '-1 month' +%s
