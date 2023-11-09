# Work flow types

## Summary

The omotes framework allows for different work flow types (calculation types) to be available to the user.
The omotes architecture design determines how the work flow types are configured and how they can be added.  
To decide which of the design option described below is best suited, the following questions play a role:

- Will all deployments use the same work flow types? Are any 'custom' work flow types expected: work flow types that are
  not part of the open source omotes repository?
- How often it is expected to add new work flow types?
- Is downtime acceptable when adding or removing work flow types?

TODO: update conclusion below  
The chosen design is the 'static work flow types, configurable during deployment': added as it gives the option of
custom work flow types with limited additional work or adding many complexities.

## Architecture options

Below are several options in which the work flow types could be declared and made available

- Hard-coded in the python sdk (NWN POC implementation)
- Static work flow types, configurable during omotes deployment
- Dynamic work flow types, configurable during runtime

These three options are described below with a description of what is needed to add a work flow type,
what the requirements are and pro's and con's.

### Hard-Coded in the python sdk

This is the setup used in the NWN POC version.
For custom work flow types a specific sdk version will have to be created, which will not be usable by others.
Adding a new work flow type means redeploying the framework.

#### Steps to add a work flow type

1. Update python sdk code to create specific pip package version
2. Update sdk dependency for components that use the python sdk (front-end, workers)
3. Add new work flow type worker(s) to docker-compose
4. Redeploy omotes

#### Requirements

As this is the setup of the NWN POC version, no further requirements are needed.

#### Considerations

The pro of this option is that it is already available, no further work necessary.  
The cons are that it forces case-specific 'python sdk' versions for custom work flow types, several components need to
be updated when adding a work flow type, and a redeployment is necessary when adding a new work flow type.

#### Conclusion

This design is a straightforward option if we don't expect many new work flow types.
Custom work flow types are not really workable with this design.

### Static work flow types, configurable during installation

Work flow types specified in configuration file used during the installation of omotes.
Adding a new work flow type means redeploying the framework.

#### Steps to add a work flow type

1. Update configuration file
2. Add new work flow type worker(s) to docker-compose
3. Redeploy omotes

#### Requirements

1. The python sdk needs to be updated to set the work flow types via a configuration file.

#### Considerations

The pro of this design is that it streamlines adding a new work flow type and custom work flows are possible.
The con is that a redeployment of omotes is required when adding a work flow type.

#### Conclusion

This design is an efficient option which allows for custom work flow types.
Adding a work flow type will result in some downtime during redeployment.

### Dynamic work flow types, configurable during runtime

A service registry allows for dynamic adding (and removing) of work flow types.

#### Steps to add a work flow type

1. Deploy new work flow type worker(s) which will then automatically register in the service registry and become
   available

#### Requirements

1. Create a service registry component
2. Implement service registration in the workers
3. Update the python sdk to check for available work flow types (to be used by the front-end)

#### Considerations

The pro is that work flow types can be added and removed during runtime.
The con are that another component needs to be created, and it adds complexity to the system.

#### Conclusion

This design offers most flexibility, including adding and removing of work flow types without downtime, at the cost of
the work required to set it up and added complexity to the system. 