# Google Cloud Platform Logging in a Django project

Django projects use the standard Python Logging module to write logs to various destinations, including the Google Cloud Platform Logging. However, Google Logging provides more features than the Python Logging module. For example, Google Logging introduced structural logs and additional logging levels. You may use the Google Logging API client library for Python directly, but that’s not ideal in many cases.
The code presented here provides an interface to Google Logging via the Python Logging module API with additional functionality. It allows writing structural logs and emitting records with 'NOTICE', 'ALERT' and 'EMERGENCY' levels. It requires only minimal changes to a Django project and supports all other features of the Python logger.

For example, the call

```
logger.notice('A notice with a data payload', x=1, y='foo', z={'k1':10,'k2':20})
```
emits an entry in the Google Logging, such as
```
{
  "insertId": "1qpskcifhsqcri",
  "jsonPayload": {
    "x": 1,
    "y": "foo",
    "z": {
      "k1": 10,
      "k2": 20
    },
    "message": "A notice with a data payload"
  },
  ...
```
Additionally, you may redirect the output to a file or any other destination supported by the Python logger:
```
... A notice with a data payload {'x': 1, 'y': 'foo', 'z': {'k1': 10, 'k2': 20}} ...
```
Calls `logger.error('an error message'), logger.info('FYI'), etc.` work as expected.

The implementation is in the utils/logging.py file. That is the only file you need.
The rest of the files here are for a quick setup of a demo.

## How to set up
1. Set Up Google Logging for Python. See [Google Docs.](https://cloud.google.com/logging/docs/setup/python)
2. Add the file utils/logging.py to your project
3. Modify the settings.py file.

   You will need:
   
    `LOGGING_CONFIG = 'utils.logging.logging_config'`, where
    `logging_config` is a helper function for additional configuration steps
     
    `gcplog 'class: 'utils.logging.CloudLoggingHandlerX'`
    to map logger levels to Google Cloud severity levels
    
    `gcplog 'client': google_cloud_client()` to instantiate Google Logging client
          
    `file 'class': 'utils.logging.JsonFieldsFileHandler'` if you want to write JSON fields in the log file as shown in examples here

   Use custom labels `'NOTICE', 'ALERT', 'EMERGENCY'` if you want to specify Google Logging levels 

Here is an example of a configuration enabling logging to the Google Cloud and a file.

```
import os
from utils.logging import google_cloud_client
LOGGING_CONFIG = 'utils.logging.logging_config'
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format':'%(asctime)s %(thread)d@%(process)d %(levelname)-7s '\
                '%(name)s %(message)s {%(filename)s:%(lineno)d %(funcName)s}'
        },
    },
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'utils.logging.JsonFieldsFileHandler',
            'filename': os.path.join(BASE_DIR, 'example.log'),
            'formatter': 'verbose',
            'encoding': 'utf8'
            },
        'gcplog': {
            'class': 'utils.logging.CloudLoggingHandlerX',
            'client': google_cloud_client(),
            'level': 'NOTICE'},
        },
    'loggers': {
        'mysite': {'handlers': ['gcplog', 'file', ],
            'level': 'DEBUG',
            'propagate': True
            },
        }
    }
```
### Note on authentication with GCP
If you use a credential JSON file for authentication, you may pass the location of the file to the `google_cloud_client` function. The function will set `GOOGLE_APPLICATION_CREDENTIALS` environment variable.
Other [authentication methods](https://cloud.google.com/docs/authentication/client-libraries) are available too.

# How to test

Check out this project and set up a virtual environment. E.g.
```
$mkdir django-logging
$cd django-logging
$git clone https://github.com/dmitrikozlov/django-logging.git
$python3 -m venv env
$source env/bin/activate
$cd django-logging
$pip install -r requirements.txt
```
Run tests:
```
$ ./manage.py shell
>>> import logging
>>> l=logging.getLogger('mysite')
>>> l.notice('notice with no additional data')
>>> l.notice('notice with data', x=1, y='foo', z={'k1':10,'k2':20})
```
That produces two entries in the ecample.log file and two Google Logging entries:

```
2023-10-12 01:35:36,731 4462732800@31686 NOTICE  mysite notice with no additional data {<console>:1 <module>}
2023-10-12 01:37:08,069 4462732800@31686 NOTICE  mysite notice with data {'x': 1, 'y': 'foo', 'z': {'k1': 10, 'k2': 20}} {<console>:1 <module>}
```

```
{
  "textPayload": "notice with no additional data",
  "insertId": "vn5iebfpd1ktn",
  "resource": {
    "type": "global",
    "labels": {
      "project_id": "experiments"
    }
  },
  "timestamp": "2023-10-12T01:35:36.731815Z",
  "severity": "NOTICE",
  "labels": {
    "python_logger": "mysite"
  },
  "logName": "projects/experiments/logs/python",
  "sourceLocation": {
    "file": "<console>",
    "line": "1",
    "function": "<module>"
  },
  "receiveTimestamp": "2023-10-12T01:35:37.021579896Z"
}

{
  "insertId": "1qpskcifhsqcri",
  "jsonPayload": {
    "x": 1,
    "z": {
      "k1": 10,
      "k2": 20
    },
    "y": "foo",
    "message": "notice with data"
  },
  "resource": {
    "type": "global",
    "labels": {
      "project_id": "experiments"
    }
  },
  "timestamp": "2023-10-12T01:37:08.069294Z",
  "severity": "NOTICE",
  "labels": {
    "python_logger": "mysite"
  },
  "logName": "projects/experiments/logs/python",
  "sourceLocation": {
    "file": "<console>",
    "line": "1",
    "function": "<module>"
  },
  "receiveTimestamp": "2023-10-12T01:37:08.230030431Z"
}
```
Note that a log entry with no additional (structured) data is saved as the `textPayload` field. If you supply additional data, the output is saved as the `jsonPayload` property.

I hope you find it useful.

