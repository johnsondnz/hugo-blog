---
title: "Troubleshooting Planned vs Unplanned"
date: 2017-11-29T13:38:19+13:00
draft: false
categories: [ cisco, troubleshooting ]
tags: [ study-notes, tshoot, methodologies ]
---

## Structured vs Reactive
* __Structured__: Pre-defined tasks are carried out periodically to ensure the health of the network.
  * Hours a efficiently allocated to tasks to ensure the health and security of the network.
* __Reactive__: Will always exist, however with a structured approach with peridoic maintenance events that cause a reactive repsonse are kept to a minimum.  That's the theory anyway.
  * Staff are required to drop what they are doing and attend to the issue at hand.  Other work schedules suffer as a result.

### Maintenance tasks
* Hardware/software install and configuration.
* Troubleshooting problems.
* Monitoring and tuning of the network.
* Planned.
* Documentation.
* Compliance with legal or business policies.
* Security from internal and external threats.

### Benefits of scheduled tasks.
* Predefined maintenance widow resulting in better managed downtime.
* Schedule backups and validation means quicker restoration of service.
* Predefined peridicity of patches, minimising the impact to users and network services.

## Documentation
* __Physical topology__: Takes in to account device location.
  * City.
  * Building.
  * Floor.
  * Rack.
  * Power connections.
  * Interconnections with port-information.
* __Logical topology__: Not typically concerned with device locations.
  * VLANs.
  * Spanning-tree.
* __Interconnections__: Describes all interconnects between devices and circuit IDs for ISPs.
* __Inventory__: Equipment listing:
  * Manufacturer.
  * Model #.
  * Software version.
  * License information.
* __IP Addressing__: Device/interface assignments.
  * Routing domains: BGP, EIGRP, OSPF areas etc.
* __Configuration__: Information describing:
  * Nightly changes.
  * Archives of configurations are up-to-date.
* __Original designs__: Documents are available and easy to find.

*__Note__*: It is also very important to standardise the documents.  Keeping everything roughly the same makes it so that no matter the network segment the documentation format is well understood.
