# Launch Timeline

## Before Day 0
- [x] Research Sentry for Lambda/AWS
- [x] Set ttl for clearmyrecord.codeforamerica.org to 60sec
- [x] Increase Limit of Elastic IPs
- [x] Create ACM certificates for Prod
- [x] Setup alias from clearmyrecord.org to the redirect bucket
- [ ] Create Inventories in Tower for Prod
- [x] Create tower AMI user Update Credentials on Tower
- [ ] Search for old url in site and on social media and update where possible.
- [ ] Add Run arbitrary zappa command on Ansible Tower

## Day 1 Morning

- Run snapshot on prod
- Run create_storage playbooks to update DB and S3 configurations
- Turn on bucket encryption for the media bucket
- Sync new Media bucket with old
- Update heroku to use new Heroku S3 user(config:set from command line)
- Run Sync again just incase on new Media bucket with old

## Day 1 Lunch

- Confirm Heroku site is working and Lambda branch up to date
- Run snapshot on prod
- Turn off scheduled tasks on Heroku
- Run Deploy Zappa playbook
- It should take <40min for the site to appear at www.clearmyrecord.org, we can test on the temporary domain if we set allowed host.
- Take temporary URL and run smoke tests/UAT
  - Hit the 500 page and make sure emails go out and logs happen
  - Run debug task
  - Log into admin
  - View applications list page
  - Explore applications, click around but change nothing
  - View pdf.
- After UAT on the new site and the domain is up and working.
- New site is live
- Update Twilio and Mailgun end points to new URLs
  - In mailgun click the test bounce button and check the logs for the request.
  - Call Twilio Voicemail number and check email went out to the VOICEMAIL email.

- Make really sure
- Add redirect on NameCheap from old url to new.
- Add pingdom on new Domain at /health/
- Tell team migration Lambda is live and tested.

##Day 2

- Check DNS has completely propogated and check Heroku traffic
- Do an audit hour looking atthe past days traffic for errors.
- Turn off heroku app and old site is gone
- Change database user password using change_db_password playbook (requires temporary downtime <1min)
- Make database not public(by clicking in AWS)
- Run close postgres ports playbooks
- Remove temporary user for Heroku to access media bucket.

#Rollback

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

# Monitoring Lambda

## Logs 

Currently the best way to scan the Django logs on Lambda is to go into cloud watch and click on the [log section](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logs:)

Click on the log group called intake-production, on this page you will see one log steam per invocation of zappa.  To make it easier to view click text instead of row  on the top right.  To search through the all click on the top checkbox in the checkbox column and click Search Log Group.

The logs go from oldest(top) to newest(bottom).  You can filter events using a special language decribed [here](https://docs.aws.amazon.com/console/cloudwatch/logs/patternSyntax).

- To see just the info logs type `"[INFO]"`
- To see the any 500s and other error logs search  `Error`
- to get just the gets to `"[INFO]" GET` or for post `"[INFO]" POST`
- All requests to url `"'path': '/'"`

## Lambda Metrics

To view metric(graphs) using the Web API metrics is going to give the closest experience to typical web server metrics. In the [metric section ](https://c  onsole.aws.amazon.com/cloudwatch/home?region=us-east-1#metricsV2:graph=~();namespace=AWS/ApiGateway;dimensions=ApiName,Stage). From there its simple to make a graph of total requests, 4XX, and 5XX responses.  You can also make a latency graph.

From the Lamba function itself in the monitoring tab there is a premade dashboard that shows metrics and allows us to click through to the logs in that area.  One thing to be aware of is that invoccations include scheduled tasks, command run via Zappa/Ansible, and 4min keep warm task.


## Lambda Alarms

- You can also set alarms here bases on metrics in CloudWatch.
- Tower has and can be configured to send emails on failed jobs.
- Pingdom can be pointed at the Health domain
- Django Emails on 500s(as long as sendgrid is working)

## Postgres

Postgres isn't really changing on this migrations.  We have advanced metrics turned on but AWS doesn't do slow query logs out of the box.  Here are the production postgres [logs](https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstance:id=cmr-production-r1-database;view=logs)

Do view the build in RDS dashboard click show monitoring in the RDS instance section.

