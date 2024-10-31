## Starting AICM


### Install Dependencies

AICM files are located in:
```
/opt/qti-aic/tools/aic-manager
```

AICM requires several Python packages to run.<br>
We recommend installing the dependencies in a virtual environment:
```bash
python -m venv <path-to-the-virtual-environment>
source <path-to-the-virtual-environment>/bin/activate
pip install -r requirements.txt
```

#### Start Agent

After activating the virtual environment, AICM can be started using the following command:
```
python aicm_agent.py

optional arguments:
  -h, --help                      Show this help message and exit
  --config_file                   Path to the Config file.
  --dump-default-config,          Dump the default config to the specified folder
    --dump_default_config_users   and exit.
  --dump-default-users,           Dump the default users file to the specified
    --dump_default_users          folder and exit.
  --ip                            IP address to bind to
  --port                          Port number to listen on
  --log                           Path where to store logs
  --max-log-size, --max_log_size  Max size of logs in bytes
  --ssl-key, --ssl_key            Path to the SSL key file. Needs to be provided
                                  for HTTPS.
  --ssl-cert, --ssl_cert          Path to the SSL certificate file. Needs to be
                                  provided for HTTPS
  --qmonitor-ip,                  IP of QMonitor GRPC Server
    --qmonitor_ip                 provided for HTTPS.
  --qmonitor-port,                Port of QMonitor GRPC Server
    --qmonitor_port
  -u, --users                     Full path to the users credentials file (.yaml)
  -v, --verbose                   Increase output verbosity
```

These settings can also be supplied via a configuration file.<br>
If both are found, then the command line arguments will take priority.<br>
The configuration file can be supplied via the `--config_file` option.<br>

```ini
# AICM Configuration

# IP to bind AICM
ip = 127.0.0.1

# Port to bind AICM
port = 9000

# Full path to users credentials file
# users =

# SSL Key path
# ssl_key =

# SSL Certificate path
# ssl_cert =

# Path to directory where to store logs
# log =

# IP of QMonitor GRPC Server
qmonitor_ip = localhost

# Port of QMonitor GRPC Server
qmonitor_port = 62472

# Max sizes of logs in bytes
max_log_size = 100000000

# Verbosity of AICM agent
verbose = 0
```

AICM can also be run in a service-like manner :
```
sudo bash scripts/start_aicm_agent.sh
```

Once running, the APIs can be tested at ```<ip>:<port>/docs``` through the SwaggerUI.

#### Stop Agent
The following command will stop the agent when `scripts/start_aicm_agent.sh` was used to start it:
```bash
sudo bash scripts/stop_aicm_agent.sh
```
Alternatively, pressing `Ctrl+C` will stop AICM if it's running using the `python aicm_agent.py` command.

### Setup Basic Auth

Since [Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication) is used as the authentication method, users will need to authenticate all requests to our API.

**The accepted credentials are stored in the ```.users.yaml``` file.**

You can add/modify the credentials using the following syntax:

```yaml
credentials:
  - username: admin
    hash: $2b$12$rEQTKF4IVHKPyeX6miseJ.xOjhmI5OFqlLuwE2OB4CuEIvHC2IFP6
    note: "Example of credential"
```
These credentials will be needed in every request made to the HTTP Rest Endpoints.
For security purposes the password is hashed using `bcrypt`.
A script used to get the hash is provided at `/scripts/hash_password.py`<br>
Replace `<password>` and run this command:
```bash
python ./scripts/hash_password.py <password>
```

### HTTPS
Basic Auth is just a simple mechanism for authentication.
For added security, running HTTPS is recommended, which requires users to provide a certificate and key upon startup.
This can be done by passing the following args:

```bash
  --ssl-key SSL_KEY    Path to the SSL key file. Needs to be provided for
                       HTTPS
  --ssl-cert SSL_CERT  Path to the SSL certificate file. Needs to be provided
                       for HTTPS
```

## Metrics
These are currently the metrics served by AICM (stored in `docs/metrics.csv`):
In addition to a set of core metrics, AICM provides Reliability, Availability, Serviceability (RAS) error statuses.

{{ read_csv('./metrics.csv') }}