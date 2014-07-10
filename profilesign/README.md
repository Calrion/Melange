Signing and Encrypting iOS Configuration Profiles
=================================================

`profilesign` is a shell script that simplifies signing and encrypting, then 
signing, iOS configuration profiles. The script assists with signing and 
encrypting only, not the creation of the XML configuration profile itself. 
The script employs OpenSSL for all the heavy lifting, and also requires 
`PlistBuddy` (an OS X-only utility, sorry) for working with Apple's property 
list format used by configuration profiles.

## Requirements
You'll need to have a recent version of OpenSSL, and you'll also need Apple's 
`PlistBuddy` utility, which is (as far as I know) only available on OS X.

You'll also need a key and certificate for signing. Creation of these is 
beyond the scope of this document, however certificates with the "client 
authentication", "server authentication", and "code signing" extended key 
usage purposes are known to work. The certificate you use _should be trusted_ 
by the devices you'll deploy to. The key and certificate file both need to be 
in **PEM format**.


## Usage
Before using the script, you need to configure the key and signing 
certificate. You can either save them as `~/.profilesign/signing-key.pem` 
and `~/.profilesign/signing-cert.pem` or specify their location with the 
`SIGNING_KEY` and `SIGNING_CERT` environment variables. 

The command-line syntax is:

    profilesign [-f] [--no-sign] profile.mobileconfig [device-cert.pem]

* `-f` - Optional. Forces operations to proceed, even if that results in 
  overwriting or deleting data.

* `--no-sign` - Optional. Provides for encrypting a profile and then _not_ 
  signing it. This is not recommended.

* `profile.mobileconfig` - Required. The path to the configuration profile 
  to use.

* `device-cert.pem` - Optional. The path to the certificate to use for 
encryption; the device will need to have the private key for this 
certificate. Omitting this value results in a profile that is signed but 
not encrypted.

The output file is placed in the current folder and cannot be specified. The 
output filename is based on the input filename, prefixed with either `signed-` 
or `signed-keyfilename-` based on whether the profile is signed or signed-and 
encrypted. E.g. if the script is run as `profilesign my_config.mobileconfig 
iPhone.pem` then the output file will be named 
`signed-iPhone-my_config.mobileconfig` and placed in the current folder.

## Encrypted key files
Encrypting the signing key file is supported, and will result in being 
prompted for the key password during the script's execution.

## Roadmap
I'd really like to update this to use the OS X keychain for storing 
the signing key and certificate, which would be more secure than simply 
storing them on disk. I'd also like to reduce reliance on the proprietary 
`PlistBuddy` command, so this script could run on other *NIX 
operating systems.

It might also be nice if `profilesign` could use a `.deviceinfo` file 
directly, rather than needing the certificate exported.

## FAQ

### Can I use a self-signed certificate for signing?
Yes, but it likely won't be trusted by the devices you deploy to, and 
you'll get a big red "unverified" when installing. A better idea is to 
obtain a certificate from a CA that has its [root certificates included in 
iOS](http://support.apple.com/kb/HT5012).

### What's the difference between 'verified' and 'unverified'?
When an iOS device receives a signed profile, it checks to see if the 
certificate is trusted in a similar manner to the way your web browser 
checks a web server's certificate when you connect to a secure (`https://...`) 
website. If the certificate is trusted (issued by a trusted CA and not 
expired), then iOS displays "Verified" in green; if not, "Unverified" is 
displayed in red.

What does this mean? The short of it is "verified" means you can be reasonably 
sure (to the level of the certificate's assurance) of who created the profile, 
and that it wasn't changed since it was created. With "unverified", you can be 
sure it hasn't changed since it was created (or iOS would reject it outright) 
but you can't be sure who sent it.

### Where can I get a device certificate from?
Ahh, yes. This is the tricky part. I use Apple's iPhone Configuration Utility 
(iPCU) for this, which has unfortunately been deprecated; it still works, for 
now; I also use iPCU to create the configuration profiles. I've had... issues 
with iPCU and certificates, so I prefer to fire-up a virtual machine that has 
a pristine copy of OS X and iPCU for this, but it's not usually necessary.

What you need to do is run iPCU, connect the device (via a cable), select 
that device from the list, and then export the _device record_. It's crucial 
that you export the device record rather than just the UDID as only the 
device record includes the certificate we're after.

This creates a property list (`.plist`) file from which you can extract 
the certificate using `PlistBuddy` and convert it to the PEM format we need 
using OpenSSL:

    PlistBuddy -c "Print CertificateData" DeviceName.deviceinfo | openssl x509 -inform der -outform pem -out device-cert.pem

In the above command, `DeviceName.deviceinfo` is the exported device record 
file, and `device-cert.pem` is the certificate to use for encryption. Note 
that you'll need to redo this process each time you "Erase all content and 
settings" on the device.

### Can I use Apple Configurator instead of iPCU?
No. Apple Configurator is a different tool and is not supported at this time.
