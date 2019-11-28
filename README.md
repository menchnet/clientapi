# clientapi
Allows for access to the menchnet fpga cluster for computer vision
applications.


Outline of usage:


1. Create a module in python for your computer vision sensor:

Example: sensor1.py

```python
# import libraries here, imports are restricted by a whitelist
# you can not access io or other system operations, the code
# is executed within a docker container running alpine and running
# as an unprivilaged user. 
import torch


"""
ssapi is the server side api for menchnet it allows for the module
to register to events to listen to and to publish analytics based on
those events. You can even listen to events from other modules you
uploaded so that a composition of multiple asynchronously running modules
can be aggregated.
"""

def onStartup(ssapi):
    # configuration given by menchnet.activate
    config = ssapi.getConfig()
    # ...
    # perform global setup here

    # register for events
    ssapi.register(ssapi.evt.SNAPSHOTS, url=config['url'], format="jpg", interval="2.0") 
        

def onEvent(ssapi, evt):
    if evt.name == ssapi.evt.SNAPSHOTS:
        data = evt.getData()
        # jpg image to do some computation on
        # ...
        # send a confidence value 
        ssapi.publish("score", some_value)
```    
    
2. Upload the module to your account at menchnet


```python
import menchnet

# initialization
menchnet.login(username, api-key)
menchnet.upload("/path/to/sensor1.py", "module_example_above.py")
```

3. Use the module you uploaded

```python
# activation/listen/deactivation
menchnet.login(username, api-key)
def callback(some_value):
    # get the result from ssapi.publish("score", some_value)
    pass
menchnet.listen("sensor1.score", callback)
menchnet.activate("sensor1", url="...")

...

menchnet.deactivate("sensor1")
```

4. Remove the module

```python

import menchnet
menchnet.login(username, api-key)
menchnet.remove("sensor1")
```





