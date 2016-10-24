- Feature Name: Tracer Support
- Start Date: 24/10/16
- RFC PR:
- Issue: 

# Summary
[summary]: #summary

As Katello provides the package update infrastructure to update EL7 it would be really good to be able to tell users what services / processes need to be restarted after an update. [Tracer](https://github.com/FrostyX/tracer) can do this, Katello could use Tracer to report this in a similar way that it reports packages installed on client systems. Users could then use foreman_remote_execution to restart services that are required.

# Motivation
[motivation]: #motivation

As someone that has helped countless organisations install Katello something they often ask for is a way to do this, the current best way is `needs-restarting` which it part of the yum-utils package, but that just prints out a list of processes.
Tracer has the benefit that it can display what service needs to be restarted & the command that is used to restart them if it knows, which is a lot more user friendly than a list of process ID's.

# Detailed design
[design]: #detailed-design

Katello-agent could add another yum plugin similar to [package_upload](https://github.com/Katello/katello-agent/blob/master/src/yum-plugins/package_upload.py) but can use the [tracer API](http://docs.tracer-package.com/en/latest/api/) and then make a API call to the Katello Server/Capsule/Proxy and report any services that Tracer reports requires a restart, how to restart and the app type (daemon, static, session).  

On the Content Hosts page we could add another tab to list these services and a option to restart them on the client. (this would be dependant on remote execution being enabled)

# Drawbacks
[drawbacks]: #drawbacks

- Tracer currently doesn't report Kernel updates https://github.com/FrostyX/tracer/issues/45
- We would need to package tracer for el7 (I don't think it is already packaged)
- Tracer doesn't support < el7, I'm not sure how easy it would be add el6 support

# Alternatives
[alternatives]: #alternatives

I guess we could use `needs-restarting` instead of `tracer` but a list of process ID's isn't that helpful and needs-restarting is slow compared.
