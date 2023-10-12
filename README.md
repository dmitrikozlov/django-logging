# django-logging
Setting up Google Cloud logging in a Django project

$ ./manage.py shell
>>> import logging
>>> l=logging.getLogger('mysite')
>>> l.notice('notice with no additional data')
>>> l.notice('notice with data', x=1, y='foo', z={'k1':10,'k2':20})

`2023-10-12 01:35:36,731 4462732800@31686 NOTICE  mysite notice with no additional data {<console>:1 <module>}`
`2023-10-12 01:37:08,069 4462732800@31686 NOTICE  mysite notice with data {'x': 1, 'y': 'foo', 'z': {'k1': 10, 'k2': 20}} {<console>:1 <module>}`

`{
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
}`


