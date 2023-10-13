# Google Cloud logging in a Django project

I implemented a simple extension of the Python logger to access Google Cloud structured logging and to use additional log levels 'NOTICE', 'ALERT' and 'EMERGENCY' with minimal changes to a Django project.

For example, the following call

```
logger.notice('notice with data', x=1, y='foo', z={'k1':10,'k2':20})
```
produces the following log entry in the Google Cloud Logging
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
    "message": "notice with data"
  },
  ...
```
Additionally, you may direct the output to a file:
```
... notice with data {'x': 1, 'y': 'foo', 'z': {'k1': 10, 'k2': 20}} ...
```
Calls `logger.error('an error message'), logger.info('FYI'), etc.` work as expected.

The implementation is in utils/logging.py file. That's the only file you need.
The rest of the files are for demonstration purposes in the Django environment.

## How to set up
1. Set Up Google Cloud Logging for Python. See [Google Docs.](https://cloud.google.com/logging/docs/setup/python)
2. Add utils/logging.py file to your project
3. Modify settings.py file. Below is an example config enabling logging to Google Cloud and a file.

   You will need to specify:
   
    `LOGGING_CONFIG = 'utils.logging.logging_config'`, where
    `logging_config` is a helper function for additional configuration steps
     
    `gcplog 'class: 'utils.logging.CloudLoggingHandlerX'`
    to map logger levels to Google Cloud severity levels
    
    `gcplog 'client': google_cloud_client()` to instantiate Google Cloud logger client
          
    `file 'class': 'utils.logging.JsonFieldsFileHandler'` you may want to use this handler to add json_fields in the file entries, shown in examples here

     You can specify logger levels with custom labels: `'NOTICE', 'ALERT', 'EMERGENCY'`.

Here is a complete example of an addition to your configuration:

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
That produces two entries in the ecample.log file and two Google Cloud Logging entries:

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
Note that a log entry with no additional (structured) data is saved in `textPayload` field. If you supply additional data, the output is saved in the `jsonPayload` property.

I hope you find it useful.

