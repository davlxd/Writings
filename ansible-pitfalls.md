# Ansible Pitfalls
2016-04-04

Ansible, being concise and decent with everything in YAML and agentless, 
has been a better choice among DevOps tools like Puppet, Chef, and Salt.
I’ve been using Ansible for the past 3 projects I engaged in.
The last one relies heavily on Ansible to achieve whole Infrastructure as Code,
which makes us wrote thousands of lines of YAML scripts and allows us to spot some pitfalls about Ansible.

## Test is Difficult

Usually when we write code, we write and run tests along,
this guarantees a certain level of code correctness and gives us enough confidence to move on.
So generally these kinds of tests should take no more than a few seconds,
make our coding-testing-refactor cycle as smooth as possible.

Ansible, however, doesn't have an efficient test framework.
There is no other way but run Ansible scripts against real machines shall prove correctness,
so I personally use Vagrant + VirtualBox to accomplish that.
The problem is it takes approximately 10 seconds for the VM to boot up (on my MacBook Pro15),
plus several minutes for Ansible to execute if Ansible role is big enough.
If you are not quite familiar with Ansible or if you are making a major code change,
you may have to go over this again and again,
the coding-testing-refactor cycle being dramatically slowed down.

## Write once, Revise everywhere

We've all heard about the joke about Sun's "Write once, Run everywhere" turns out to be "Write once, Debug everywhere".
To Ansible it truly is Write once, Revise everywhere.

The main philosophy behind Ansible and other similar Configuration Management tools is DSC - Desired State Configuration,
which means if we use Ansible to provision a machine from state A to state B,
we do not need to tell Ansible how to do it, but just tell Ansible we want our desired state B,
Ansible will handle the rest, and guarantee idempotency.

In reality, Ansible doesn't know every state transformation you desire,
so you still need to write procedural style Ansible script here and there, even embed shell scripts.
In this way, it would be the author’s responsibility to guarantee idempotency
and handle as many kinds of initial state as possible - as you can see, it’s not easy.

So in my last project, we've spent a whole lot more time more time on Ansible than we expected.
Every time we apply our Ansible repository in a brand new environment
(e.g. CentOS on DigitalOcean, RHEL on AWS),
we need to modify Ansible scripts to adapt new initial state possibilities,
sometimes in our own roles, sometimes in 3rd party roles,
sometimes for Ansible original modules.
Eventually, we got a stable, full-fledged, time-consumed Ansible repository,
which doesn’t deliver any customer value after all.

## That’s Why We should Use Docker

Docker solves these problems beautifully.
It kind of encapsulates an entire OS and application to a docker image - no more messed-up state,
no more dirty machines, everything is in this small image which you can download, boot up, throw away easily.
When a docker image is built and tested on your laptop, its running environment is identical wherever you run it.
Everything Ansible's been trying to do is encapsulated into docker image in a stable and controlled way.
And because docker is kernel level resource isolation - not an actual VM - it's extremely fast to boot up
and thus make it convenient to debug and test locally.
