# Declarative Infrastructure
*Discalaimer:* This is just my brain dump, it's work in progress.

The infrastructure configuration is implicit, you need to define some
services and their dependencies and the system will implicitly know
about others.

# Definitions
## Service
A Service is a artifact knowing about it's dependencies and configuration.

## Set
A Set is a group of deployed service keeping the same specific state or
configuration

## Instance
A Instance is a deployed service.

## Interface
Each service provides an interface, describing the offerings of that
instance.
This interface is what other services use to connect to a service.

# Dependencies
A Service can either depend on a generic service or on a specific set.

If it depends on a generic service, dedicated instances for this services
get deployed and form a new service set for the service depending on them.

If it depends on a specific service set, the service shares this set with
other services.

# Processing
1. Push to dockland service
2. Resolve dependencies
  1. For generic service dependency, schedule new instance in each zone
  2. For specific set dependency, return set
3. Convergence existing to new state
  1. Find affected instances
  2. Reload them


# Example
## Infrastructure
- three zones, one hw node per zone

## MANIFEST

    {
      "app": {
        "depends": [
          "psql",
          "memcache"
        ]
      },
      "worker": {
        "depends": [
          "psql:app"
        ]
      },
      "blog": {
        "depends": [
          "psql"
          "memcache"
        ]
      }
    }

## Processing of Example
1. Push
2. Resolve dependencies:
  1. app depends on..
    1. `psql`, so schedule `psql:app`
    2. `memcache`, so schedule `memcache:psql`
  2. worker depends on..
    1. `psql:app`, so die if not exists
  3. blog depends on...
    1. `psql`, so schedule `psql:blog`
    2. `memcache`, so schedule `memcache`

Now each service can connect to the services it depends on by requesting
the interface definition from dockerland.

# Limitations
Replication, failover and master election is no concern of dockerland.
Postgress in this example needs master/slave with transparent leader
election and automatic failover, so that dockerland can just spawn up
now containers.

# Random implementation notes
- Scheduling characteristics are service specific
  - memcache needs to be local, not spread across the grid
- So we need to take this into account
