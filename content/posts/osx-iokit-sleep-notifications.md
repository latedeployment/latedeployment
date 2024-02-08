---
date: 2015-02-19T19:50:21+02:00
draft: true
title: OSX IOKit sleep notification 
tags: ['IOKit', 'kernel', 'macOS']
---

## Get sleep notification in your IOKit kext

{{< highlight cpp >}}

IOReturn yourIOKitDriverClass::powerStateHandler(void *target, void *refCon, UInt32 messageType, IOService *service, void *messageArgs, vm_size_t argSize)
{
    IOPMSystemCapabilityChangeParameters * params;
    if ( messageType != kIOMessageSystemCapabilityChange )
    {
        // We are not interested in anything other than cap change.
        return kIOReturnSuccess;
    }
    params = (IOPMSystemCapabilityChangeParameters *) messageArgs;
    
    if ((params->changeFlags & kIOPMSystemCapabilityWillChange) &&
    (params->fromCapabilities & kIOPMSystemCapabilityGraphics) &&
    (params->toCapabilities & kIOPMSystemCapabilityGraphics) == 0)
    {
        // caught a sleep 
    }
    return kIOReturnSuccess;
}
    
bool yourIOKitDriverClass::start(IOService *provider)
{
    IOService *rootDomain = (IOService *)getPMRootDomain();
    if (rootDomain)
       rootDomain->registerInterestedDriver(this);
    registerPrioritySleepWakeInterest(powerStateHandler, this, 0);
}

{{< / highlight >}}

Actually it's not a sleep notification exactly but it's good enough if you don't need to handle power in your kext (and you probably don't want to handle power in your kext)
