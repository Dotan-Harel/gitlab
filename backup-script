This restores user accoutns and 2 factor authentication without touching the girlab.rb file
so that you can have 2 live production sites OR a hot backup.


-------------------------
[gitlab restore.sh]


gitlab-ctl stop
touch /etc/gitlab/skip-auto-migrations
yum update -y
gitlab-ctl restart
gitlab-ctl reconfigure
gitlab-ctl restart
mkdir -p /backups/gitlab_bak
cp -rf /restore /backups/gitlab/restore
mv /backups/gitlab /backups/gitlab_bak/"gitlab_bak_$(date +%F)"
mkdir -p /backups/gitlab/rails/
rsync -avrx -e "ssh -c arcfour -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --progress ec2-user@**OMITTED**:/backups/ /restore/

mv /etc/gitlab/gitlab-secrets.json /etc/gitlab/"gitlab-secrets.json_bak_$(date +%F)"
cp -f /restore/gitlab/gitlab-secrets.json /etc/gitlab/gitlab-secrets.json

cd /restore/gitlab/rails/
file=`ls -t *.tar | awk '{printf("%s",$0);exit}'`
cp /restore/gitlab/rails/$file /backups/gitlab/rails/

gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

name=`echo $file | cut -c1-21`

gitlab-rake gitlab:backup:restore BACKUP=$name force=yes

rm /etc/gitlab/skip-auto-migrations
gitlab-ctl start
gitlab-rake gitlab:check SANITIZE=true
gitlab-ctl reconfigure
echo GitlabRestored and Online

-------------------------
[gitlab backup.sh]

#backing up before new version installed
umask 0077; tar cfz /backups/gitlab/$(date "+etc-gitlab-\%s.tgz") -C / etc/gitlab

yum update -y
gitlab-ctl reconfigure

#below replaces crontab so that the backup is made with the latest version
umask 0077; tar cfz /backups/gitlab/$(date "+etc-gitlab-\%s.tgz") -C / etc/gitlab
sleep 30s
gitlab-rake gitlab:backup:create

mkdir -p /backups/gitlab/gitlab-secrets_bak/
mv  /backups/gitlab/gitlab-secrets.json /backups/gitlab/gitlab-secrets_bak/"gitlab-secrets.json_bak_$(date +%F)"
cp /etc/gitlab/gitlab-secrets.json /backups/gitlab/gitlab-secrets.json
echo Done Gitlab Backup
