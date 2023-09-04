# Cling in java

## Two important concepts in cling
Before you start using the Cling framework, you must understand two concepts
in Cling. One is `Service`, and the other is `Device`.

1. **Service**: Just like a function in common programming language, service also accepts
arguments and return result after processing.
2. **Device**: A device can have many services, just like a class can hava multiple
functions in Object-Oriented Language.

## Creating a service
Next, I will customize a `VolumeControlService` service, that is same to
[official example](http://4thline.org/projects/cling/core/manual/cling-core-manual.xhtml#section.SwitchPower) in Cling

1. First, we need to create a maven project, then add some dependencies and `nexus-4thline`
repository (Cling and Jetty related package all downloaded from this repository) 
2. Second, Modify maven version then sync maven project(Please use `apache-maven-3.6.3`, the higher
versions don't allow direct use HTTP repository url )
3. Create VolumeControlService, the detail code is as follows
```java
package org.example;

import org.fourthline.cling.binding.annotations.*;

@UpnpService(
        serviceId = @UpnpServiceId("VolumeControlService"),
        serviceType = @UpnpServiceType(value = "VolumeControlService")
)
public class VolumeControlService {
    @UpnpStateVariable(defaultValue = "50")
    private int volume = 50;

    @UpnpAction
    public void setVolume(@UpnpInputArgument(name = "ExpiredVolumeValue") String expiredVolumeValue) {
        int volumeValue = Integer.parseInt(expiredVolumeValue);
        if (volumeValue < 0 || volumeValue > 100) {
            throw new IllegalArgumentException(String.format("VolumeValue range between [0, 100], you input value is %s", volumeValue));
        }
        volume = volumeValue;
    }

    @UpnpAction(out = @UpnpOutputArgument(name = "RealVolumeValue"))
    public int getVolume() {
        return volume;
    }
}
```

## Binding service to device
After the service is successfully created, wo need to create a device to bind the service. In the next,
we will create PlayDevice which bind VolumeControlService.
```java
package org.example;

import org.fourthline.cling.binding.annotations.AnnotationLocalServiceBinder;
import org.fourthline.cling.model.DefaultServiceManager;
import org.fourthline.cling.model.ValidationException;
import org.fourthline.cling.model.meta.*;
import org.fourthline.cling.model.types.DeviceType;
import org.fourthline.cling.model.types.UDN;

public class PlayDevice {
    public static LocalDevice getInstance() throws ValidationException {
        DeviceIdentity deviceIdentity = new DeviceIdentity(UDN.uniqueSystemIdentifier("PlayDevice"));
        DeviceType deviceType = new DeviceType("org.example", "PlayDevice");
        DeviceDetails deviceDetails = new DeviceDetails("PlayDevice");
        LocalService volumeControlService = new AnnotationLocalServiceBinder().read(VolumeControlService.class);
        volumeControlService.setManager(new DefaultServiceManager(volumeControlService, VolumeControlService.class));
        return new LocalDevice(deviceIdentity, deviceType, deviceDetails, volumeControlService);
    }
}

```

## Service remote call
Assuming we want to invoke `VolumeControlService` in `PlayDevice`, the simplest way is to get `deviceInstance` first,
and because we emphasized a device can bind many services in previous section, it is obviously that
we can't get the corresponding service through operations like (`deviceInstance.xxxService`).
So in the device inner must keep a service id to service map, so that wo can get correct service by unique service id. 

[//]: # (In order to demonstrate how to make remote service calls, we need to crate a `ServerSideThread` and `ClientSideThread` )

[//]: # (1. Init UpnpService)

[//]: # (2. Choosing device instance &#40;usually obtained through LAN search&#41;)

[//]: # (3. Find the service you want form the device instance)

[//]: # (4. Invoke service by start special action, the result of action in the action callback)

[//]: # ()
[//]: # (---)

[//]: # ()
[//]: # (When you want to use the UPNP/DLNA protocol to achieve cross device screen projection function, the CLING framework seems to be your best choice. Compared to encapsulating HTTP request packets for device control on your own, CLING provides a simple and efficient method. You only need to obtain the specified services of the remote device and set relevant parameters to easily complete the functions you want.)

[//]: # (# Cling in java)

[//]: # ()
[//]: # (## In this cahpter)

