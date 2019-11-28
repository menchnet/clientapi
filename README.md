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

    # generate stapshot events every 2.0 seconds using the
    # video source specified by url
    ssapi.generator(ssapi.evt.SNAPSHOTS, url=config['url'], format="jpg", interval="2.0") 
        

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
# upload a module, we give it the arbitrary name 
# sensor1 the same as the module name but it can be 
# anything you want.
menchnet.upload("sensor1", "/path/to/sensor1.py")
```

3. Use the module you uploaded

```python
# activation/listen/deactivation
menchnet.login(username, api-key)
def callback(some_value):
    # get the result from ssapi.publish("score", some_value)
    pass
# inside sensor1.py the ssapi.publish will append the event 
# name score to the name of module so 
#   menchnet.upload("sensor1", ...)
#   and ssapi.publish("score", data)
#   results in a data being published to the sensor1.score topic
#   this is pub/sub so any number of clients can listen for the same 
#   event.
menchnet.listen("sensor1.score", callback)

# results in onStartup being called and passing in the keyword arguments
# of this function, which in this case is url="..."
# after calling this a daemon is running on menchnet which will
# connect to the url using and start taking snapshots, the onStartup
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





