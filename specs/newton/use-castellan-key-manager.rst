..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Integrate Castellan for Key Management
======================================

https://blueprints.launchpad.net/cinder/+spec/use-castellan-key-manager

Castellan is a key manager interface library that is intended to be usable
with multiple back ends, including Barbican. The Castellan code is based on
the basic key manager interface that resides in Nova and Cinder. Now that the
key manager interface lives in a separate library, the key manager code can be
removed from Nova and Cinder, and Castellan can be used as the key manager
interface instead.

Problem description
===================

As encryption features in OpenStack projects are becoming more common, the
projects typically need a way to interface with a key manager. Different
deployers may have different requirements for key managers, so the key
manager interface must also be configurable to have different back ends. The
Castellan key manager interface was based off the key manager interfaces found
in Cinder and Nova. Now that the shared key manager interface lives in a
separate library, the original key manager interface embedded in Cinder can be
removed and Castellan used instead.

Use Cases
=========

Castellan supports existing features such as volume encryption.

Proposed change
===============

Castellan must be added to the required libraries list.

Castellan by default pulls configuration options from a Castellan-specific
configuration file in /etc/castellan, but can also take in configuration
options if passed in directly. The configuration options for the key manager
can still be specified in cinder.conf, and passed along to Castellan.

The old key manager interface code and back end implementations in
cinder/keymgr and tests in cinder/tests/unit/keymgr can be removed. Any place
in the Cinder code where the key manager interface was called will be replaced
by calls to Castellan instead. Castellan does not include ConfKeyManager, an
insecure fixed-key key manager that reads the key from the configuration file.
The implementation for ConfKeyManager will remain in Cinder but will be
converted to a Castellan plugin to exercise the relevant code paths.

Alternatives
------------

The alternative is to leave the key manager interface as it is, but this means
that Cinder's key manager will not benefit from the updates, new features, and
future additional back ends available in Castellan.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

Castellan behaves very similarly to the current Cinder key manager. Castellan
has added improvements and bug fixes beyond what is currently in the Nova and
Cinder key managers, making it more secure. The fixed-key key manager found in
Nova and Cinder is insecure for deployments, but is useful for testing.
Castellan doesn't include the fixed-key key manager, so the ConfKeyManager
will be converted to a Castellan plugin and will remain in the Cinder code.

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

The deployer should be made aware of a change in the default key manager back
end. The current default back end in Cinder is a fixed key, but Castellan uses
Barbican as the default. This means that the deployer should ensure Barbican is
running and the fixed key added to Barbican so it can continue to be used.

The options in the Cinder configuration file for volume encryption will
change. The option 'keymgr' will be spelled out to 'key_manager'. The key
manager option group will still have an option 'api_class' to specify the
desired back end. In the 'barbican' option group, a few new options will be
available to increase the robustness of the back end, such as the number of
times to check if a key has been successfully created.

To maintain backwards compatibility, the old options will still be listed as
deprecated options. Standard deprecation policy will be followed, and these
old options should be removed in the next release cycle.

Developer impact
----------------

Cinder developers should not be impacted by this change. If developers find
more uses for a key manager, Castellan should be just as easy to use as the
current Cinder key manager interface.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Kaitlin Farr <kaitlin.farr@jhuapl.edu> kfarr on IRC

Other contributors:
  None

Work Items
----------

 * Add Castellan to Cinder requirements.
 * Replace calls to Cinder's key manager with calls to Castellan.
 * Remove Cinder key manager code.
 * Update documentation.

Dependencies
============

This change depends on Castellan, version >= 0.2.0. Castellan is already in
OpenStack's global requirements.

Testing
=======

This change can be unit tested using a simple in-memory back end. As actual
deployments should be using Barbican, this feature should be tested using a
Barbican back end, too.

Documentation Impact
====================

These changes will be documented. Cinder documentation for volume encryption
will be updated to reference Castellan [4].

References
==========

[1] Castellan source code:
  https://github.com/openstack/castellan

[2] Castellan in OpenStack's global requirements:
  https://github.com/openstack/requirements/blob/master/global-requirements.txt

[3] Current Cinder key manager implementation
  https://github.com/openstack/cinder/tree/master/cinder/keymgr

[4] Volume encryption configuration reference
  http://docs.openstack.org/liberty/config-reference/content/section_volume-encryption.html
