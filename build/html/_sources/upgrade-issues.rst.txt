==============
Upgrade issues
==============

This page documents upgrade issues and notes. These may apply to either of the
three upgrade types (charms, OpenStack, series).

The issues are organised by upgrade type.

Charm upgrades
--------------

rabbitmq-server charm
~~~~~~~~~~~~~~~~~~~~~

A timing issue has been observed during the upgrade of the rabbitmq-server
charm (see bug `LP #1912638`_ for tracking). If it occurs the resulting hook
error can be resolved with:

.. code-block:: none

   juju resolved rabbitmq-server/N

manila-ganesha charm: package updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To fix long-standing issues in the manila-ganesha charm related to `Manila
exporting shares after restart`_, the nfs-ganesha Ubuntu package must be
updated on all affected units prior to the upgrading of the manila-ganesha
charm in OpenStack Charms 21.10.

OpenStack upgrades
------------------

Keystone and Fernet tokens: upgrading from Queens to Rocky
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting with OpenStack Rocky only the Fernet format for authentication tokens
is supported. Therefore, prior to upgrading Keystone to Rocky a transition must
be made from the legacy format (of UUID) to Fernet.

Fernet support is available upstream (and in the keystone charm) starting with
Ocata so the transition can be made on either Ocata, Pike, or Queens.

A keystone charm upgrade will not alter the token format. The charm's
``token-provider`` option must be used to make the transition:

.. code-block:: none

   juju config keystone token-provider=fernet

This change may result in a minor control plane outage but any running
instances will remain unaffected.

The ``token-provider`` option has no effect starting with Rocky, where the
charm defaults to Fernet and where upstream removes support for UUID. See
`Keystone Fernet Token Implementation`_ for more information.

FWaaS: upgrading from Ussuri to Victoria
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Firewall-as-a-Service (`FWaaS v2`_) OpenStack project is retired starting
with OpenStack Victoria. Consequently, the neutron-api charm will no longer
make this service available starting with that OpenStack release. See the
`21.10 Release Notes`_ on this topic.

Prior to upgrading to Victoria users of FWaaS should remove any existing
firewall groups to avoid the possibility of orphaning active firewalls (see the
`FWaaS v2 CLI documentation`_).

Series upgrades
---------------

DNS HA: upgrade to focal
~~~~~~~~~~~~~~~~~~~~~~~~

DNS HA has been reported to not work on the focal series. See `LP #1882508`_
for more information.

Upgrading while Vault is sealed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a series upgrade is attempted while Vault is sealed then manual intervention
will be required (see bugs `LP #1886083`_ and `LP #1890106`_). The vault leader
unit (which will be in error) will need to be unsealed and the hook error
resolved. The `vault charm`_ README has unsealing instructions, and the hook
error can be resolved with:

.. code-block:: none

   juju resolved vault/N

.. LINKS
.. _Release Notes: https://docs.openstack.org/charm-guide/latest/release-notes.html
.. _Upgrades: https://docs.openstack.org/operations-guide/ops-upgrades.html
.. _Update services: https://docs.openstack.org/operations-guide/ops-upgrades.html#update-services
.. _Keystone Fernet Token Implementation: https://specs.openstack.org/openstack/charm-specs/specs/rocky/implemented/keystone-fernet-tokens.html
.. _vault charm: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md#unseal-vault
.. _manila exporting shares after restart: https://bugs.launchpad.net/charm-manila-ganesha/+bug/1889287
.. _21.10 Release Notes: https://docs.openstack.org/charm-guide/latest/2110.html
.. _FWaaS v2: https://docs.openstack.org/neutron/ussuri/admin/fwaas.html
.. _FWaaS v2 CLI documentation: https://docs.openstack.org/python-neutronclient/ussuri/cli/osc/v2/firewall-group.html

.. BUGS
.. _LP #1882508: https://bugs.launchpad.net/charm-deployment-guide/+bug/1882508
.. _LP #1886083: https://bugs.launchpad.net/vault-charm/+bug/1886083
.. _LP #1890106: https://bugs.launchpad.net/vault-charm/+bug/1890106
.. _LP #1912638: https://bugs.launchpad.net/charm-rabbitmq-server/+bug/1912638
