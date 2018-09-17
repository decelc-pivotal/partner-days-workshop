---
pageTitle: Exercise #2 - BOSH Lab
---

# Introduction

In this lab, you will create and upload a BOSH release.

# Part 1: Create a BOSH Release

Requirements:
BOSH cli: https://bosh.io/docs/cli-v2.html#install
Check version with:
    ```bash
    bosh -v
    ```
Make sure you have the latest (version 3.0.1+)


1. Initialize a new release
    ```bash
    mkdir my-postgres-release
    cd my-postgres-release
    bosh init-release --git
    ```

Best Practice: Use - (dash) in the release name, and _ (underscores) in all the files in the release

Review what happened by viewing the directory tree
	tree


2. Create a new BOSH job
    ```bash
    bosh generate-job my_pg_server
    ```

Review new job
	tree

Check the job skeletons
    ```bash
    cat jobs/my_pg_server/spec
    cat jobs/my_pg_server/monit
    ```


3. Create a startup script
    ```bash
    vim jobs/my_pg_server/templates/start.erb
    ```

    Script contents:
    -------------------------- cut and paste --------------------------
    ```bash
    #!/bin/bash -e
    LOG_DIR=/var/vcap/sys/log/my_pg_server
    DATA_DIR=/var/vcap/jobs/my_pg_server/data
    BASE_DIR=/var/vcap/packages/my_pg_pkg
    BINARY_DIR=$BASE_DIR/bin
    if ! test -d $LOG_DIR; then
      sudo mkdir $LOG_DIR
      sudo chown -R vcap:vcap $LOG_DIR
    fi

    if ! test -d $DATA_DIR; then
      mkdir $DATA_DIR
      sudo chown -R vcap:vcap $DATA_DIR
      sudo chmod 700 $DATA_DIR
      sudo -u vcap $BINARY_DIR/pg_ctl initdb -D $DATA_DIR -o "--auth=trust"
      # enable TCP/IP connections
      sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/g" $DATA_DIR/postgresql.conf
      sed -i 's/#port = 5432/port = 5432/g' $DATA_DIR/postgresql.conf
      echo 'host		all	all	0.0.0.0/0	trust' >>$DATA_DIR/pg_hba.conf
    fi
    sudo -u vcap $BINARY_DIR/pg_ctl start -l $LOG_DIR/server.log -D $DATA_DIR
    ```

    Fun fact: vcap stands for "VMWare Cloud Application Platform"


4. Create a stop script
    ```bash
    vim jobs/my_pg_server/templates/stop.erb
    ```

    Script contents:
    -------------------------- cut and paste --------------------------
    ```bash
    #!/bin/bash -e
    LOG_DIR=/var/vcap/sys/log/my_pg_server
    DATA_DIR=/var/vcap/jobs/my_pg_server/data
    BASE_DIR=/var/vcap/packages/my_pg_pkg
    BINARY_DIR=$BASE_DIR/bin
    PIDFILE=$DATA_DIR/postmaster.pid
    killall postgres
    ```

  5. Edit the monit script
      ```bash
      vim jobs/my_pg_server/monit
      ```

      Script contents:
      -------------------------- cut and paste --------------------------
      ```bash
      check process psql
        with pidfile /var/vcap/jobs/my_pg_server/data/postmaster.pid
        start program "/var/vcap/jobs/my_pg_server/bin/start" with timeout 600 seconds
        stop program "/var/vcap/jobs/my_pg_server/bin/stop"
        group vcap
      ```

6. Update the job spec
    ```bash
    vim jobs/my_pg_server/spec
    ```

    Script contents:
    -------------------------- cut and paste --------------------------
    ```bash
    ---
    name: my_pg_server
    templates:
      start.erb: bin/start
      stop.erb: bin/stop
    packages:
    - my_pg_pkg
    properties: {}
    ```

7. Get postgres distribution

    Download Postgres
	   # From my-postgres-release
    ```bash
    wget https://ftp.postgresql.org/pub/source/v9.3.5/postgresql-9.3.5.tar.gz -O src/postgresql-9.3.5.tar.gz
    ```

    Note: if you don't have wget, you can install on a Mac with:
    ```bash
    brew install wget
    ```

    Note: real world use case would typically use a blobstore for a large compressed file like this. See https://bosh.io/docs/create-release/#config-blobstore. Definitely consider a blobstore if the file size is large.

8. Create a package
    ```bash
	  bosh generate-package my_pg_pkg
    ```

    Review new package
	   tree

     Update the package spec
      ```bash
      vim packages/my_pg_pkg/spec
      ```

      Script contents:
      -------------------------- cut and paste --------------------------
      ```bash
      ---
      name: my_pg_pkg
      dependencies: []
      files:
      - postgresql-9.3.5.tar.gz
      ```

    Edit the packaging script to compile and install the software
    ```bash
    vim packages/my_pg_pkg/packaging
    ```


    Script contents:
    -------------------------- cut and paste --------------------------
    ```bash
    #!/bin/bash -e

    tar zxvf postgresql-9.3.5.tar.gz
    pushd postgresql-9.3.5
      # need to run as root?
      # sudo su -
      ./configure --prefix=${BOSH_INSTALL_TARGET}

      make
      make install
    popd

    # post-install procedures
    LD_LIBRARY_PATH=/usr/local/pgsql/lib
    export LD_LIBRARY_PATH
    ```

9. Create a deployment manifest

	 From my-postgres-release
   ```bash
   mkdir manifests
   vim manifests/postgres.yml
   ```

   -------------------------- cut and paste --------------------------
    ```bash
    name: my_postgres
    releases:
      - {name: my-postgres, version: latest}
    stemcells:
      - alias: bosh-google-kvm-ubuntu-trusty-go_agent
        os: ubuntu-trusty
        version: latest
    update:
      canaries: 1
      max_in_flight: 10
      canary_watch_time: 1000-30000
      update_watch_time: 1000-30000
    instance_groups:
      - name: postgresql_server_node
        instances: 1
        azs: [z1]
        jobs:  
          - name: my_pg_server
            release: my-postgres
        vm_type: n1-standard-1
        cloud_properties:
          tags:
            - allow-ssh
        stemcell: bosh-google-kvm-ubuntu-trusty-go_agent
        persistent_disk_type: 50GB
        networks:
          - name: private
    ```

# Part 2: Upload and deploy your BOSH release

10. Target the BOSH environment via the credentials we shared with you, copy to your workspace and unarchive
    ```bash
  	cd <directory containing bbl environment>
  	source env.sh
  	# Verify your environment setup
  	bosh environment
    ```

11. Create and upload a BOSH release
    ```bash
  	cd my-postgres-release
  	bosh create-release --force
  	bosh upload-release
    ```


12. Deploy
    ```bash
  	bosh upload-stemcell https://s3.amazonaws.com/bosh-gce-light-stemcells/light-bosh-stemcell-3445.7-google-kvm-ubuntu-trusty-go_agent.tgz
  	bosh -d my_postgres deploy manifests/postgres.yml
    ```


13. Verify
    ```bash
  	# ssh to the postgres VM.
  	bosh -d my_postgres ssh
  	# on that VM, start a postres session
  	/var/vcap/packages/my_pg_pkg/bin/psql -h 127.0.0.1 --username=vcap -d template1
  	# inside the postres session:
  	\list
  	# when you're ready to exit the postgres session
  	\q
  	# when you're ready to end the ssh session
  	exit
    ```


14. Cleanup: this will delete the BOSH postgres deployment.
    ```bash
    bosh -d my_postgres delete-deployment --force
    ```
