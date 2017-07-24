..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Nova disc quota tracking
========================

https://blueprints.launchpad.net/nova/+spec/nova-disk-quota-tracking

Nova has lots of resource quotas, but not for allocated storage to VMs.

Problem description
===================

Now, it is not possible to limit user with some specific amount of root and
ephemeral disc storage allocated to VM during boot based on flavor's values.
Because of absence of such quota we are forced to rely on Cinder volumes
to be able to bill for storage and restrict storage in flavors.

Use Cases
---------

In case we care only about allocated storage in general and not on allocation
means, whether it is Cinder or root disc for VM, we would need a storage quotas
in nova. So, cloud admins will have more flexibility for storage tracking.

Proposed change
===============

It is proposed to create additional quota resource called, for example,
'local_gb' that will consist of root disc and ephemeral storage, that nova
has been tracking for long time. It will require small update of API, where
we show new quota resource in addition to existing ones and allow to update
it with quota and quota class sets. Also, we will need to create new
configuration option with default value, in the same way it is done for all
other existing quotas.

Alternatives
------------

The only alternative is to continue doing things as is, not tracking storage
we allocate for nova VMs based on its flavor.

Data model impact
-----------------

No DB models changes are expected, because nova already tracks such kind of
a storage, just doesn't use it for quotas.

REST API impact
---------------

Bump of microversion will be required, because we will extend existing
'quota-sets', 'quota-class-sets' and 'limits' APIs.

Following API changes are expected to be implemented:

* 1) quota show API

  * URL: /v2.1/os-quota-sets/<scope>
  * Method: GET
  * Proposed update to JSON response::

        {
            "quota_set": {
                "ram": 51200,
                ...
                "local_gb": 1024,
                ...
                "cores": 20

            }

        }

* 2) quota update API

  * URL: /v2.1/os-quota-sets/<scope>
  * Method: PUT
  * Proposed update to JSON request::

        {
            "quota_set": {
                "local_gb": 1024

            }

        }

  * Proposed update to JSON response::

        {
            "quota_set": {
                "local_gb": 1024

            }

        }

* 3) quota class show API

  * URL: /v2.1/os-quota-class-sets/<scope>
  * Method: GET
  * Proposed update to JSON response::

        {
            "quota_class_set": {
                "ram": 51200,
                ...
                "local_gb": 1024,
                ...
                "cores": 20

            }

        }

* 4) quota class update API

  * URL: /v2.1/os-quota-class-sets/<scope>
  * Method: PUT
  * Proposed update to JSON request::

        {
            "quota_class_set": {
                "local_gb": 1024

            }

        }

  * Proposed update to JSON response::

        {
            "quota_class_set": {
                "local_gb": 1024

            }

        }

* 5) View existing limits and usages API

  * URL: /v2.1/limits
  * Method: GET
  * Proposed update to JSON response::

        {
            "limits": {
                "rate": [],
                "absolute": {
                    "maxTotalRAMSize": 51200,
                    ...
                    "maxLocalGBSize": 1024,
                    ...
                    "maxTotalCores": 20

                }

            }

        }

Security impact
---------------

None

Notifications impact
--------------------

No changes to notification mechanism or its usage is expected, because
we will reuse existing nova logic for one more resource quota.

Other end user impact
---------------------

- Users will not be able anymore to abuse of storage resources provided by
  flavors.

- We will need to update python-novaclient to be able to update new quota.

Performance Impact
------------------

In theory, we will have some additional logic for taking decision about whether
a user is allowed to boot VM with specific flavor based on its storage or not.
In practise, this logic is very cheap operation.

Other deployer impact
---------------------

Deployer will need to define one more configuration option called,
for example, "quota_local_gb" in case default value is not suitable for his
deployment.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Artem Tiumentcev ("darland-maik" at lauchpad.net)

Work Items
----------

- Update quotas and limits API bumping API microversion.
- Update python-novaclient to support this new server feature.
- Update docs with new quota resource description.

Dependencies
============

None

Testing
=======

Several kinds of tests can be added for covering this feature:

- unit tests in nova
- functional tests in nova
- API and scenario tests in tempest

Documentation Impact
====================

Small changes to docs will be required, where we will describe one new
quota resource.

References
==========

None

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Queens
     - Introduced
