# Ansible Playbooks for CMR

Currently this is a Work in Progress.  The easiest way to work on this project is to create an intake-ops folder on your laptop.

You will need to create a virtualenv with python 3.6 and 2.7

# Create checkout and install playbook requirements

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

Make cmr-dev-envs with export statements with 

```bash
export AWS_ACCESS_KEY_ID=''
export AWS_SECRET_ACCESS_KEY=''
```

Also manually configure aws cli.  This will not be required soon but is also useful later.
````bash
aws configure
```

To create a new project

```bash
cp -r inventories/development-r1 inventories/project-name
# edit inventories/bgolder/vars/all/vars to make a unique public project slug
# edit inventories/bgolder/vars/all/vault to include new secrets
#   pg_password: "secret"
#   pg_app_password: "secret"
#   pg_clips_password: "secret"
```

To test what will happen when if you were to create_storage_environments you can run this command.  create_storage_environments.yml is configured to create the VPC, users, s3 buckets, and databases for the CMR project.

```bash
ansible-playbook -i inventories/project-name create_storage_environents.yml --check
```

If that is a new inventories then check should say created for each line.  If not you've likely not updated your project_slug.

So if you then run the same command without check it should work fine.

```bash
ansible-playbook -i inventories/project-name create_storage_environents.yml
```

To remove everything run.

```bash
ansible-playbook -i inventories/project-name remove_storage_environents.yml
```


# To Install zappa

From witn the intake-ops folder checkout intake and install requirements.

```bash
git clone git@github.com:codeforamerica/intake.git
python3.6 -m intake-venv
source intake-venv/bin/activate
cd intake
git checkout feature/zappa_changes
pip install -r requirements/app.txt
pip install zappa
pip install boto
```


