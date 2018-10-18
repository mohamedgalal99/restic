# restic
restic backup solution

Restic backup

Restic backup based on take copy from all data first time and after that it take only changes which marked as snapshots every time, and we can roll back to one of this snapshots

- On backup server we create dir for backup

        => mkdir /opt/bc-incibaid

- On original server where we want to take backup from
- download binary package from https://github.com/restic/restic/releases/tag/v0.9.3

        => curl -LO https://github.com/restic/restic/releases/download/v0.9.2/restic_0.9.2_linux_amd64.bz 
           bunzip2 restic_0.9.2_linux_amd64.bz2
           mv restic_0.9.2_linux_amd64 /usr/local/bin/restic
           chmod a+x /usr/local/bin/restic
        
        => restic generate --bash-completion /etc/bash_completion.d/restic     # To make auto complete for restic command
           source /etc/bash_completion.d/restic
        
        => cat >> ~/.ssh/config << END
        HostName backup-server
                HostName 185.15.201.75
                User root
                Port 2022
        END     
        
        => restic -r sftp:backup-server:/opt/bc-incubaid/ init  # This will initialize repo over (sftp) backup server, passwd u created should be remembered else data will be loss
        
        => echo "dc106401468b6b" > pass.txt
        
        =>  restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid list snapshots
        
        => restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid --verbose backup /mnt/tttttt   # take backup from dir
        => restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid --verbose backup --stdin --stdin-filename ${db}.sql
        
        => for db in $(su postgres -c 'psql -U postgres -c "\l"' 2> /dev/null | grep odoo | awk '{print $1}')
           do
               su postgres -c "pg_dump -U postgres -b -E UTF-8 ${db}" | restic --password-file /root/pass.txt -r sftp:backup-server:/opt/bc-incubaid --verbose backup --stdin --stdin-filename ${db}.sql
           done

        => restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid dump ${snapshot_id} ${db}.sql | su postgres -c psql     # Restor database back
           
        => restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid forget ccdae593  # to delete specific snapshot
           restic --password-file pass.txt -r sftp:backup-server:/opt/bc-incubaid prune

