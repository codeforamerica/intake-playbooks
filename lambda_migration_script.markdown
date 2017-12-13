Before Day 0
- [x] Research Sentry for Lambda/AWS
- [x] Set ttl for clearmyrecord.codeforamerica.org to 60sec
- [ ] Increase Limit of Elastic IPs
- [x] Create ACM certificates for Prod
- [x] Setup alias from clearmyrecord.org to the redirect bucket
- [ ] Create Inventories in Tower for Prod
- [x] Create tower AMI user Update Credentials on Tower
- [ ] Search for old url in site and on social media and update where possible.
- [ ] Add Run arbitrary zappa command on Ansible Tower

Day 0

- Run snapshot on prod
- Run create_storage playbooks to update DB and S3 configurations
- Turn on bucket encryption for the media bucket

Day 1

- Update to new sync the new s3 bucket and update Heroku and delete on old
- Confirm Heroku site is working and Lambda branch up to date
- Run snapshot on prod
- Run Deploy Zappa playbook
- It should take <40min for the site to appear at www.clearmyrecord.org, we can test on the temporary domain if we set allowed host.
- After UAT on the new site and the domain is up and working.
- New site is live

- Update Twilio and Mailgun end points to new URLs
- Turn off scheduled tasks on Heroku
- UAT Twilio Voicemail

- Make really sure
- Add redirect on NameCheap from old new to new.
- Add pingdom on new Domain at /health/

Day 2

- Check DNS has completely propogated and check Heroku traffic
- Turn off heroku app and old site is gone
- Change database user password(requires temporary downtime <1min)
- Make database not public(by clicking in AWS)
- Run close postgres ports playbooks

Rollback

If rollback is  before we turn on the redirect we can just not launch since the old domain is not effected.  After the launch before Day 2, rollback is just a matter of changing the Namecheap DNS back to heroku and the AWS dns to a bucket that redirects to the OLD url.

DNS Rollback

- Revert Namecheaps DNS back
- Point Twilio and Mailgun back to Heroku
- Set Route53 to redirect back to old domain
- Add scheduled tasks back to prod.

If we are doing a rollback after Day 2 but the database and s3 buckets are ok.

- We need to update the Database and s3 credentials
  - Potentially recreate IAM users
- Rollback to a pre zappa codebase if zappa changes have been merged in
- Open Postgres ports by running create_storage environments again
- Make RDS public by clicking in the console
- Turn on heroku app
- Quick UAT of Heroku app
- Follow DNS rollback steps

If the db and s3 are corrupted

This is the nightmare scenario and is likely something that will cause unavoidable downtime and most likely have lost applications

- Rollback database to known clean state
- The s3 buckets don't have versioning for security reasons so we will likely need rebuild any pdfs generated from broken data.  If we have new files that are no longer in the db they shouldn't cause any harm but could be used to(by hand) redo a submission
- Followed post Day 2 rollback steps
- We might need to rerun scheduled tasks and do other steps depending on the issue.

Stuff to Reseach

- Will replicas effect changing PG subnets around?
- Since Lambda rolls keys when updating Heroku will loss S3 access each deploy/update unless it is also updated.
  - Make a 2nd user for heroku ahead of time and delete it later?

