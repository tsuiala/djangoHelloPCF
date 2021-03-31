# Development
Refer to _manifest.yml_
```bash
python3 manage.py collectstatic --noinput && gunicorn djangoHelloPCF.wsgi:application
```

# Pre tasks <a name="pre_tasks"></a>
1. Create MySQL service for the App djangoHelloPCF `cf create-service p.mysql db-small mysql`
1. Create service key `cf create-service-key mysql mysql-service-key`
1. Update variable `DATABASES['default']` in _djangoHelloPCF/settings.py_ with the `uri` information from `mysql-service-key` created above
1. Generate `SECRET_KEY` with `python3 -c 'from django.core.management import utils; print(utils.get_random_secret_key())'` command
1. Fill in `SECRET_KEY` in _djangoHelloPCF/settings.py_ with key generated from the above

# Getting start
```bash
CF_HOME=${HOME}
cf login
cf target $ORG $SPACE

# Install prerequisite libraries
pip3 download -d vendor \
  --index-url https://artifactory.platform.manulife.io/artifactory/api/pypi/pypi/simple \
  --trusted-host artifactory.platform.manulife.io \
  -r requirements.txt --no-binary :all:

# Push App to PCF
cf set-env djangoHelloPCF DISABLE_COLLECTSTATIC 1
cf push -b python_buildpack

# Bind database service
cf bind-service djangoHelloPCF mysql
cf restage djangoHelloPCF
cf logs djangoHelloPCF --recent
```

# Post tasks
```bash
cf run-task djangoHelloPCF "python manage.py migrate" --name migrate
cf logs djangoHelloPCF --recent

cf run-task djangoHelloPCF "python manage.py test" --name test
cf logs djangoHelloPCF --recent
```

# Troubleshooting
## Symptom
```
ImproperlyConfigured at /lists/new
settings.DATABASES is improperly configured. Please supply the ENGINE value. Check settings documentation for more details.
```

## Solution
Refer to [Pre tasks](#pre_tasks)

## Symptom
```python
ProgrammingError at /lists/new
(1146, "Table 'service_instance_db.lists_list' doesn't exist")
```
## Solution
```python
cf run-task djangoHelloPCF "python manage.py migrate" --name migrate
```

# Reference
https://www.ianhuston.net/2017/10/bringing-django-to-cloud-foundry/
