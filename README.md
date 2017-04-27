# Gaia
This is a placeholder repository for Gaia, the off-chain encrypted storage layer in Blockstack.  It will not hold code, but will be the community's point of contact for issues about the design and implementation of this subsystem. The code itself will be merged to [Blockstack Core](https://github.com/blockstack/blockstack-core).

Gaia is mostly complete.  The only missing features are encryption and multi-device support.  We are looking for a fast, correct ECIES library.  Let us know if you know of one :)

We expect to have this working as a PoC by the end of spring 2017.

# Design Notes

Gaia implements datastores on top of one or more existing commodity storage services.  A user gets a datastore for each application; the single-page Blockstack application running in the user's browser uses the Gaia datastore to host its user-specific state.

Datastores follow a filesystem-like layout, with files and directories.  A datastore's data is hosted on one or more commodity storage services of the user's choosing.  Gaia uses commodity storage services to host encrypted blocks and inodes, as if the storage service was just a dumb hard drive.

Gaia ships with drivers for many different storage services.  These include cloud storage (e.g. Dropbox, S3), decentralized storage (e.g. BitTorrent), and personal servers.  Gaia replicates data across all of them by default, so that as long as at least one is online, the users can access their data.

Gaia datastores are writable only by the user that owns them.  However, other users can read them as long as they have the right drivers.  For example, a photo-sharing app that uses Gaia to host data will let each user store their photos and photo albums in their own datastores, and let them view other users' photos and albums by looking up that user's app-specific datastore and downloading the files.  This lets Gaia support access patterns found in many widely-used applications today, like blogging, PIM, file-sharing, and social media.

When the application stores a file or directory in a Gaia datastore, it will be signed by the user's private key to prevent tampering.  Other users can find the public key by looking up the user's name with Chronos, getting the root data public key from the user's Atlas zone file, and then using the application domain name to derive an application-specific public key for the application's Gaia datastore.  Each file and directory is versioned in Gaia, and readers keep track of the last versions of each file they read in order to avoid consuming stale data.

For confidentiality, files in Gaia will be padded and encrypted with an application-given set of public keys.  Gaia would get the public keys for specific users by looking them up with Chronos, and then finding their datastore-specific keys via Atlas.

## Multi-device Datastores

A critical requirement is that users should be able to read and write to their Gaia datastores from multiple devices.  Each of a user's devices should have a separate data public key listed in the user's Atlas zone file (i.e. derived from a per-device keyring).

It's not clear to me yet how this should work.  In a naive attempt, this would translate to datastores being specific to a user, application, *and* device.  Looking up an inode would mean looking up a user's device public keys, and then looking for the inode in each device's app-specific datastore.  The problem is that with multiple devices, this turns each inode lookup into N requests against M storage services (N == number of devices).  This could get costly, since M is going to be greater than 1 already.

Another (naive) approach would be to simply give each device the same keys.  This reduces the number of network requests to M, but reduces security (i.e. a single device compromise would compromise all application data for that user--the status quo today).

What I'd like is a way to look up an inode using just one network request in the common case, even if the datastore is writable by multiple devices.

# Deliverables

* Storage service drivers
* Standardized data formatting schemas
* Code for loading and storing client-signed datastore information via an untrusted Blockstack Core node
* API endpoints for creating, reading from, writing to, and deleting datastores
* Cross-device datastore support--an application running on a user's devices sees the same view of the data regardless of which device it's on.

# Point of Contact

* Jude Nelson (@jude on blockstack.slack.com)

# How can I Help?

Glad you asked!

* Help us work out cross-device datastores
* Point us to a fast ECIES implementation (preferably using `cryptography`).
* Ask questions as Github issues
* PRs are welcome :)
