# Installation

```
pip install splunk-hec-handler
```

# Features
1. Log messages to Splunk via HTTP Event Collector (HEC).
See [Splunk HEC Documentation](http://docs.splunk.com/Documentation/Splunk/latest/Data/AboutHEC)
2. All messages are logged as '_json' sourcetype by default.
3. A dictionary with 'log_level' and 'message' keys are constructed for logging records of type string.

![String log record representation in Splunk](https://github.com/vavarachen/splunk_http_handler/blob/master/resources/str_record.png)

4. Dictionary objects are preserved as JSON.

![Dictionary log record representation in Splunk](https://github.com/vavarachen/splunk_http_handler/blob/master/resources/dict_record.png)

5. If log record (dict) does not contains a 'time' field,  one is added with the value set to current time.

# Examples

## Basic
```python
import logging
from splunk_hec_handler import SplunkHecHandler
logger = logging.getLogger('SplunkHecHandlerExample')
logger.setLevel(logging.DEBUG)

# If using self-signed certificate, set ssl_verify to False
# If using http, set proto to http
splunk_handler = SplunkHecHandler('splunkfw.domain.tld',
                    'EA33046C-6FEC-4DC0-AC66-4326E58B54C3',
                    port=8888, proto='https', ssl_verify=True,
                    source="HEC_example")
logger.addHandler(splunk_handler)
```

Following should result in a Splunk entry with _time set to current timestamp.

```python
logger.info("Testing Splunk HEC Info message")
```

![Basic Example](https://github.com/vavarachen/splunk_http_handler/blob/master/resources/basic_example.png)

Following should result in a Splunk entry of Monday, 08/06/2018 4:33:43 AM, and contain two
custom fields (color, api_endpoint).  Custom fields can be seen in verbose mode. 

```python
dict_obj = {'time': 1533530023, 'fields': {'color': 'yellow', 'api_endpoint': '/results'},
                    'user': 'foobar', 'app': 'my demo', 'severity': 'low', 'error codes': [1, 23, 34, 456]}
logger.error(dict_obj)
```

![Fields Example](https://github.com/vavarachen/splunk_http_handler/blob/master/resources/fields_example.png)

:warning: In order to use custom fields, 'sourcetype' property must be specified in the event 
and sourcetype definition must enable *indexed field extractions*.


See http://dev.splunk.com/view/event-collector/SP-CAAAE6P for 'fields'

## Advanced
Using 'fields', many of the metadata fields associated with an event can be changed from the default.  Additionally, new
fields, which are not part of the event, can be also added.

In the following example, we are sending events to two different indexes (see "Select Allowed Indexes (optional)" setting)
and overriding 'host', 'source', 'sourcetype' fields, while adding some new fields ('color', 'api_endpoint').

```python
import logging
from splunk_hec_handler import SplunkHecHandler

logger = logging.getLogger('SplunkHecHandlerExample')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.level = logging.DEBUG
logger.addHandler(stream_handler)

token = "EA33046C-6FEC-4DC0-AC66-4326E58B54C3'
splunk_handler = SplunkHecHandler('splunkfw.domain.tld',
                                 token, index="hec",
                                 port=8080, proto='https', ssl_verify=False
                                 source="evtx2json", sourcetype='xxxxxxxx_json')
logger.addHandler(splunk_handler)


dict_obj = {'fields': {'color': 'yellow', 'api_endpoint': '/results', 'host': 'app01', 'index':'hec'},
            'user': 'foobar', 'app': 'my demo', 'severity': 'low', 'error codes': [1, 23, 34, 456]}
logger.info(dict_obj)

log_summary_evt = {'fields': {'index': 'adhoc', 'sourcetype': '_json', 'source': 'adv_example'}, 'exit code': 0, 'events logged': 100}
logger.debug(log_summary_evt)
```

![Advanced Fields Example](https://github.com/vavarachen/splunk_http_handler/blob/master/resources/advanced_example.png)


# Todo
1. Event acknowledgement support

