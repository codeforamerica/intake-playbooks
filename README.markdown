# Working within Tower

Tower can be used from the web interface found [here](tower.clearmyrecord.org) and as well as using the Tower cli documented [here](http://tower-cli.readthedocs.io/en/latest/)

## Creating and Updating Environments

After being given login credentials anyone with proper permissions can update the production to the most recent master branch environment by going [here](https://tower.clearmyrecord.org/#/templates/job_template/24) and the staging environment by going [here](https://tower.clearmyrecord.org/#/templates/job_template/10).

The [Job Templates](https://tower.clearmyrecord.org/#/templates) page lists all of the jobs currently configured.  

The [Deploy New Zappa Environment](https://tower.clearmyrecord.org/#/templates/job_template/16) to create a new environment. First you will now create a new inventory(see creating inventory section).  Then simply click the corresponding rocket ship icon in the actions menu and follow the instructions to select the newly created inventory and dev aws credentials from the wizard.


You can then use [Update Lambda Environment](https://tower.clearmyrecord.org/#/templates/job_template/18) to update that environment in the future.  Then simply click the corresponding rocket ship icon in the actions menu and follow the instructions to select the proper inventory and aws credentials from the wizard.

[Run Arbitrary Command on Lambda](https://tower.clearmyrecord.org/#/templates/job_template/22) is a Job which wraps the Zappa cli command like so `zappa [run__command_pre] [environment] [run_command_post]`.  So for example following the instructions below will run the run_debug_task management command.

1. Click on the rocketship icon on Run Arbitrary Command on Lambda row.
1. Select the development inventory and credentials.
1. Set the extra inventory variables to yaml below.  

```yaml
---
run_command_pre: "manage"
run_command_post: "run_debug_task"
```

You can use the Zappa CLI in this way to run any maintence code necessary for instance data requests or to manually correct something in the database in cases where a migration wouldn't make sense.


## Creating Inventories
CMR's tower instance has one schema for inventories.  All of the environments have the same keys but some different values.  The easiest way to make a new one is to copy the Dev inventory and change these values to new values: project,project_slug,environment_name,domain.

You can do that by going to [Inventories](https://tower.clearmyrecord.org/#/inventories) section and copying the variables.  Then add a new inventory and paste in the old variables, update the variables, and save them.

## Anatomy of our inventories in Tower and Locally

Typically inventories are broken our into vars and vault files but Tower just has a single textarea for both.  So for clarity the top half are safe to share but the second half are private and should not be made public.  To make local development easier the production/staging/demo/dev inventories are kept in the repository as well in inventories/stage_name/group_vars/all/vars.  The vault files are kept out of git and encrypted using ansible-vault when not in use.


## Troubleshooting

There are a small number of known issues that you might run into while creating, removing, or updating machines.

-  While deploying code if an error occurs its possible that S3 user credentials might have been created but the old one not deleted.  This wil cause an error prevent the next code update fro mworking.  When this happens go into the IAM panel in AWS and remove the most recently created access key for the cmr-[stage name]-s3-user, which should never have been used.  If both have been used check and see which is in use by Lambda.
- When undeploying zappa it fails trying to remove a security gruop.  This is because Zappa's undeploy doesn't remove everything it created when its inside a VPC.  You will need to remove the network interface in the related lambda-security-group.  This means if you want to remove everything associated with Zappa instance you will have to manually remove that.  Zappa also leaves behind the DNS settings, custom domains, which will need to be removed if you want to be able to deploy to the same domain later.
- Its possible if Zappa has only partially been removed and you try to deploy it again it might cause errors which make it hard to deploy environments with the same name.  If this happens try removing the CloudFormation group and see what errors and then remove those things until you can remove the CloudFormation.



# Developing Ansible Playbooks for CMR

## Getting Started Developing Locally


```bash
mkdir intake-ops
cd intake-ops
git clone git@github.com:codeforamerica/intake-playbooks.git
cd intake-playbooks
virtualenv -p /path/to/python2.7 venv
source venv/bin/activate
pip install -r requirements.txt
```

Go get your IAM credentials

Make cmr-dev-envs with export statements and then source it

```bash
export AWS_ACCESS_KEY_ID=''
export AWS_SECRET_ACCESS_KEY=''
```

## To create a new project

```bash
cp -r inventories/development-r1 inventories/project-name
# edit inventories/project-name/vars/all/vars to make a unique project slug and other variables
# edit inventories/project-name/vars/all/vault to include new secrets
#   pg_password: "secret"
#   pg_app_password: "secret"
#   pg_clips_password: "secret"
```


Running this command will create a new environment
```bash
ansible-playbook -i inventories/project-name create_storage_environents.yml
```

To remove everything run.

```bash
ansible-playbook -i inventories/project-name remove_storage_environents.yml
```

If you want to deploy a new instance of Zappa you can run, which will also create a new db and s3 buckets if you haven't already

* This only works on a Linux machine.  OSX currently doesn't work with django-Levenstein

```bash
ansible-playbook -i inventories/project-name zappa_deploy.yml
```

