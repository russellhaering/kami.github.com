# One tiny daemon to harvest your server statistics (and more)

---

<!--

Doing on host monitoring is hard. In a given organization there are multiple OS
platforms, multiple machine types with different CPU/memory availability and
you need to monitor them all regardless.

Ideally you want these machines to be running similar software which means you
turn to a monitoring daemon written in a high level language like
Ruby/Python/etc. But you don’t want your small cloud machines using a large
percentage of memory or CPU doing monitoring either. It is a difficult
trade-off of simplicity vs portability.

Virgo was designed with this in mind.

At its core virgo is written in C. But that is just for the basic platform.
Everything that can be is written in Lua on top of Luvit. This means it is
light on CPU and memory usage.

To target multiple platforms virgo is compiled into one standalone binary that
requires the most minimal OS platform beneath it. Oh, and it builds against OS
X, Linux and Windows.

It is a great bit of technology, built on some great emerging projects and it
is open source. Learn about the cool tech, why it is being built, how it is
being used and how you can start hacking on it too. 

-->

---

### Who am I?

<div class="notes">

- Worked on Linux Kernels and robots. Lots of C in my past
- Contracted with Cloudkick working on the Cloudkick agent
- We will discuss how that shaped some of the design decisions of this agent. 

</div>

- Love working on systems software
- Used to do Linux Kernel stuff
- I work on Cloud Monitoring
    - API Driven, [Monitoring as a Service][cm].
    - Distributed across all Rackspace data centers.
    - Async events trigger support issues.
    - When things break, we can be called in.

[maas]: http://www.rackspace.com/cloud/cloud_hosting_products/monitoring/

---

## What is monitoring?

<div class="notes">
- Define monitoring
- External monitoring
- Internal monitoring
</div>

---

### What is monitoring?

<div class="notes">

- Monitoring is all about gaining insight into a system.
- Concept of blackbox testing and white box testing in all systems
- Products that exist today including: Wormly, Circonus, ServerDensity, Pingdom, Copperegg, Cloudkick

Cloudkick is now Rackspace

</div>

- Awareness of the system

1. External monitoring
1. Internal monitoring

Our goals:
- Move past top
- Move past ping

---

### What is external monitoring?

<div class="notes">

- Black box testing
- We have all done adhoc monitoring with ping, nmap or curl
- Also pollers like nagios, gangalia, cacti

</div>

![](./blackbox.png)

- ad-hoc monitoring
    - `ping`, `nmap`, `curl`
- pollers
    - nagios, ganglia, cacti, zenoss, noitd

---

### What is internal monitoring?

<div class="notes">

- White box testing
- Essentially run scripts, gather data, push up to a central server
- NRPE and NSClient are arguably best known
- Proprietary ones mostly written in ruby, python, etc to skirt around platform issues
- Cloudkick agent was C + Lua
- Others are just a php script you run on cron

</div>

![](./whitebox.png)

- run scripts
- gather statistics
- push to monitoring server

- examples of internal monitors
    - NRPE, NSClient++
    - Cloudkick agent
    - Several proprietary ones

---

## Alerting and Historical Data

<div class="notes">

- So you are monitoring, great! Now what?
- There are essentially two things:
    - Urgent problems: Alert
    - Figuring out slow moving problems: Historical data

**punchline: very bipolar problems solved with monitoring either panic mode or
pretty graphs**

</div>

---

### Using monitored data: historical

- Trending data
    - Immediate, Hourly, Daily
    - Etc
- Generate pretty graphs

---

### Using monitored data: alerting

- Pipeline looking at data
- Language to define alert states
- Notifications sent on alert
    - SMS
    - Webhooks
    - Email

---

## What does a monitoring agent do?

<div class="notes">

- Externally your machine serves up services over tcp and udp. 
- Internally your machine has a ton of metrics and information that it can expose
- May allow the user to write their own plugins to ship metrics back

</div>

- Enables white box monitoring
- Report CPU/Memory/Disks/Network/Database/Files
- User provided plugins or scripts
- Daemonized or ran on cron

---

## Virgo

<div class="notes">

- Virgo is an open source project
- Infrastructure for the Rackspace Cloud Monitoring agent

</div>

---

### Design constraints

<div class="notes">

- Cloud machines come in small sizes, need low memory
- Updating a binary is hard
- Statically link to reduce support load and problems
- Support all of the platforms

</div>

- Low memory usage (< 5 Mb)
- Simple secure "proxyable" protocol
- High level scripting language
- Statically linked requirements
- Windows, Linux, and OSX

---

### Design decisions

<div class="notes">

- C++ has a ABI that changes a lot
- We use luajit **210K for a jit and VM**
- statically link everything except libc
    - sigar, openssl, minizip

</div>

- Avoid C++ & use Lua
- Use Lua
- Only depend on libc
- SSL + JSON newline + JSONRPC
- libuv

---

### Memory Usage

<div class="notes">

- 15 - 20 Mb for a ruby agent
- 8 - 12 Mb for a python agent
- 20+ for a C++ agent

</div>

![](./memory.png)

---

### Protocol Overview

<div class="notes">

- 15 - 20 Mb for a ruby agent
- 8 - 12 Mb for a python agent
- 20+ for a C++ agent

</div>

![](./agent-overview.png)

---

### Protocol Overview: Hello Request

<div class="notes">

- Here is an example of an agent saying hello

</div>

    {
      target = "endpoint",
      source = "agentA",
      id = 0,
      params = {
        agent_id = "agentA",
        process_version = "f451d7097edb197a9e08fa05cf5b0556ed15d7c7",
        token = "0000000000000000000000000000000000000000000000000000000000000000.7777",
        bundle_version = "0.1-75-gf451d70"
      },
      v = "1",
      method = "handshake.hello"
    }

---

### Protocol Overview: Hello Response

<div class="notes">

- And a response from the agent endpoint

</div>

    {
      target = "agentA",
      source = "endpoint",
      id = 0,
      result = { 
        heartbeat_interval = 1000
      },
      v = "1"
    }

---

### Protocol Overview: Check Schedule

    {
      "v": "1",
      "id": 2,
      "source": "endpoint",
      "target": "agentA",
      "result": {
        "checks": {
          "id": "ch1234",
          "type": "agent.cpu",
          "details": { "foo": "foo" },
          "period": 30,
          "timeout": 30,
          "disabled": false
        }
      },
      "error": null
    }

---

### Protocol Overview: Proxyable

<div class="notes">

- the target and source is for proxyability
- customer might want to proxy all agent connections through one host

</div>

![](./agent-overview-proxy.png)

---

## How is it built?

---

## Luvit

![](./ziggy.png)

---

### Untechnical Overview

<div class="notes">

luvit is scrawny like Mr. Stardust and uses very little memory.  luvit is a
young project and still growing, expect awkwardness. lua is Portuguese for moon
so it is space themed just like Ziggy. There is a great community with a good
sense of humor (luv_handles are a great data structure name)

</div>

Luvit is a platform for building your app

- Scrawny
- Awkward
- Space Themed (lua)
- &lt;3 community
- Familiar node APIs


---

### Technical Overview

<div class="notes">

luajit is a really tiny jit vm for lua, super fast. The event loop is I/O
driven like nodejs. Unlike node.js lua has a really simple C APIc; this makes
native modules quite a bit nicer to build. luvit has the protocols and crypto
you would expect. Also, runs on all major platforms. Essentially a cross
platform platform to build your application.

</div>

- lua using luajit
- low memory footprint
- I/O driven event loop
- Small simple C API
- crypto, ssl, zlib, json bindings
- tcp, http, dns protocol support
- Windows, Linux, FreeBSD and OSX

---

### HTTP Server Example


<div class="notes">

This code works today. It serves up an HTTP 1.1. server on 8080
that tells you Hello!

</div>

    local http = require("http")

    http.createServer(function (req, res)
      local body = "Hello world\n"
      res:writeHead(200, {
        ["Content-Type"] = "text/plain",
        ["Content-Length"] = #body
      })
      res:finish(body)
    end):listen(8080)

    print("Server listening at http://localhost:8080/")


---

## lua

---

### Lua - Javascript's Long Lost Brazilian Cousin

<div class="notes">

Lua shares a lot of features with javascript like using floating
point numbers only, being dynamic and having first class functions. It
is like node's long lost Brazilian cousin

</div>

- Dynamic language
- Floating point numbers only
- First class functions
- Lexical closures
- Metatables
- Embeddable

---

### Example code

<div class="notes">

Using meta tables in lua you can implement an object system.  Essentially the
meta table `__index` field tells lua "if you can't find the requested element
in this table try this table". In that way you can implement Objects. One thing
to note about lua is that the a:heard() syntax passes in a as "self" to the
heard function. Calling a.heard() will error as self will be nil.

</div>


    GroundControl = {}

    function GroundControl.new()
      obj = {}
      obj.heard_major_tom = false
      setmetatable(obj, { __index = GroundControl })
      return obj
    end

    function GroundControl:heard()
      print(self.heard_major_tom)
    end

    a = GroundControl.new()

    a:heard()
    a.heard() -- this will error

---

## libuv

---

### Basic idea

<div class="notes">

Essentially libuv is just a big loop (see the next section) that
runs poll on a bunch of file descriptors with the timeout of the poll
set to the next timer that needs to run. When the poll complete a
callback is made so the user can handle the event. I have [talk on
libuv][libuv-osb] that covers all the details too

</div>

- Two types of events in the loop:
    - I/O on file descriptors
    - Timers for future events
- Callbacks are attached to these events
- epoll()/completion ports/kqueue() wait
- callback is called on the correct event


[libuv-osb]: http://ifup.org/slides/libuv-osb/#1

---

### Event loop pseudo code

    while (1) {
      nfds = poll(fds, next_timer());

      if (nfds == 0)
         timer_callback();

      for(n = 0; n &lt; nfds; ++n) {
         if (fds[n] == READY)
            callbacks[n]();
      }
    }

---

## Zip

---

## Zip

- Check upgrades don't require package and binary upgrade

- Lua code lives in a zip file
- One file upgrades over API
- Small number of files on disk

--

## Sigar

---

## Thats Virgo.

<div class="notes">

- Its an agent
- We want to take over the world
- Join us

</div>

---

### Demo: Fixture server just for OSCON &lt;3

<div class="notes">

- Live demo, danger zone

</div>

![](./demo.gif)
