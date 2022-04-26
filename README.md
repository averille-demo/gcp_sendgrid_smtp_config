## gcp_sendgrid_smtp_config

Google Cloud Composer (Airflow v2.1.1) SMTP Email Configuration

<img src="./img/gcp_airflow.png">

[Option 1](https://console.cloud.google.com/marketplace/product/sendgrid-app/sendgrid-email): setup SendGrid within a single project on GCP (free for 12,000 emails/mo)

[Option 2](https://signup.sendgrid.com): setup a dedicated SendGrid account outside of GCP (free for 100 emails/day = ~3000/mo)
then add config to Airflow environements. 

Both options require adding DNS entries to your domain provider, the benefit of Option 2 is only we only need to set this up once across more than one cloud project. 


### SendGrid Setup

<img src="./img/sendgrid.png" width=50% height=50%>


```
# create account and enable 2FA
https://app.sendgrid.com/settings/auth

# setup sender authentication: 
https://app.sendgrid.com/settings/sender_auth

# authenticate custom domain name
https://app.sendgrid.com/settings/sender_auth/domain/create

# verify a single sender email address
https://app.sendgrid.com/settings/sender_auth/senders/new

# setup new API key: 
https://app.sendgrid.com/settings/api_keys

# stmp relay: 
https://app.sendgrid.com/guide/integrate/langs/smtp
```

### Standard SMTP ports: 
```
25      (unencrypted)
587     (TLS connections)
465     (SSL connections)
```

### Google Cloud Platform (GCP) Setup

```
Enable: 'Secret Manager API' in cloud project
https://console.cloud.google.com/apis/library/secretmanager.googleapis.com

GCP: Security -> Secret Manager -> CREATE SECRET
https://console.cloud.google.com/security/secret-manager

# create secret:
name: 'airflow-config-smtp-password'
value: SMTP_API_KEY (value from: https://app.sendgrid.com/settings/api_keys)

Assign role/permissions to service account to access secret: 
[SA_NAME]@[GCP_PROJECT]iam.gserviceaccount.com (used to create composer environment)
'Secret Manager Admin' role

# update Airflow configuation override values: (requires ~20 minute rebuilding Cloud Composer environment after Edit -> Save)
Section     Key                     Value
webserver   dag_orientation         TB
...
email       email_backend           airflow.utils.email.send_email_smtp
...
smtp        smtp_host               smtp.sendgrid.net
smtp        smtp_port               587
smtp        smtp_user               apikey (not actual API key, username='apikey')
smtp        smtp_password_secret    smtp-password
smtp        smtp_starttls           True
smtp        smtp_ssl                False
smtp        smtp_mail_from          no-reply-composer@your_custom_domain.com (needs to be validated)
...
secrets     backend                 airflow.providers.google.cloud.secrets.secret_manager.CloudSecretManagerBackend
etc.
```

### Resources:
* [SendGrid](https://sendgrid.com/)
* [Configure Email Notifications](https://cloud.google.com/composer/docs/configure-email)
* [Secrets Manager Docs](https://cloud.google.com/composer/docs/secret-manager)
* [Create a Secret](https://console.cloud.google.com/security/secret-manager)
* [Override airflow.cfg](https://cloud.google.com/composer/docs/overriding-airflow-configurations)
* [Override secrets manager](https://cloud.google.com/composer/docs/secret-manager)
