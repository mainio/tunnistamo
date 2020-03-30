# Tunnistamo

## Prerequisites

Tunnistamo runs on postresql. Install the server on Debian based systems with:
```
apt install postgresql
```

Then create a postgres user and db as root:
```
createuser <your username>
createdb -O <your username> tunnistamo
```


## Installing
Clone the repo:
```
git clone https://github.com/City-of-Helsinki/tunnistamo.git
cd tunnistamo
```

Initiate a virtualenv and install the Python requirements:
```
pyenv virtualenv 3.6.2 tunnistamo-env
pyenv local tunnistamo-env
pip install -r requirements.txt
```

You may choose some other Python version to install but currently Tunnistamo
requires Python 3.

Create `local_settings.py` in the repo base dir containing the following line:
```
DEBUG = True
```

In case you want to modify the default database configurations, you may also
modify them in the same file by adding these lines:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'custom_database',
        'USER': 'custom_user',
        'PASSWORD': 'your_password',
        'HOST': '127.0.0.1',
    }
}
```

Run migrations:
```
python manage.py migrate
```

Create admin user:
```
python manage.py createsuperuser
```

Run dev server:
```
python manage.py runserver
```
and login to http://127.0.0.1:8000/ using the admin user credentials.

To access the themed views you also need to install
[npm](https://docs.npmjs.com/getting-started/installing-node) and run
`npm install` at the project root.


## Setting up

To setup the OpenID connect authentication flow, complete the following steps:

1. Setup authentication providers that you want to use (e.g. social media
   accounts). Tunnistamo hooks up with
   [django-allauth](http://django-allauth.readthedocs.io/en/latest/index.html),
   so take a look at
   [their documentation](http://django-allauth.readthedocs.io/en/latest/providers.html)
   on how to setup the different providers. The available providers can be set
   directly from the admin panel.
1. Setup login methods under "Users" that are connected with the providers.
1. Setup an application under "Users" that you want to authenticate the users
   to. Map the applicable login methods to the application.
1. Setup at least one RSA key under "OpenID Connect Providers".
1. Add a new client under "OpenID Connect Providers". This is the configuration
   for the application that needs to authenticate through Tunnistamo.
   * The `code` response type is suitable for most "normal" web applications,
     some other application may need other response type. See
     [RFC Section 3](http://openid.net/specs/openid-connect-core-1_0.html#rfc.section.3)
     for more information. Note that your client library may also have
     limitations on which response type you can use.

At the client side, please use the following configurations for the `code`
response type (change the URLs accordingly to your instance):

- Scope: `['openid', 'email', 'profile']` (add more scopes when needed)
- Response type: `code` (or whatever you defined in the client configurations on
  the server)
- Client ID: the ID you defined for the "OpenID Connect" client at the
  Tunnistamo server
- Secret: the secret that was generated for the "OpenID Connect" client by the
  Tunnistamo server
- Server URL: `http://127.0.0.1:8000` (depending on the client library, this
  may also be split to multiple configs: scheme, host and port).
- Authority or issuer domain: `http://127.0.0.1:8000/openid`
  * This is the domain which is used to fetch the configuration from. The
    configuration is served from
    `http://127.0.0.1:8000/openid/.well-known/openid-configuration`. The current
    implementation does not support configuration
    [autodiscovery](http://openid.net/specs/openid-connect-discovery-1_0-21.html)
    which is why this needs to be explicitly defined.
  * If you need to manually configure the OpenID Connect client (without machine
    reading the configuration definitions), you can read the configuration from
    that endpoint on the server. Most implementations should support automatic
    configuration.
- Redirect URL: This is the URL where the user is returned to in your
  application after the authentication process completes


## Developing

### Outdated Python dependencies
Tunnistamo uses [prequ](https://github.com/suutari/prequ) – a fork of pip-tools –
to manage the Python dependencies.
prequ can handle `-e` style dependencies (git URLs) in the requirements files.

Update the requirements with:
```
pip install prequ
rm requirements.txt
prequ update
```

## License
This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.
