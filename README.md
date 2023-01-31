This is an export template for netbox to generate a router.db file for rancid.

It uses custom fields that will need to be configured on deivce, device_role,
device_type, and manufacturer.  This enables you to specify the rancid type
somewhere in the hirarchy and you don't need to override it for each
individual device.

For instance, you may have cisco devices and 99% of them might use "ios" as
the device type.  You can set the manufacturer rancid_type to "ios".

But then you get an ASA and need to specify that it uses a different rancid
type.  To do this you can go to the device_type and specify "asa" there.
Every ASA will inherit this rancid_type.

Maybe you have a physical machine that is a supermicro, but it happens to be
acting as a router and running Vyos.   This is where the "Role" override can
come in.  Make a role called "Vyos Router" and specify the rancid_type to be
"vyos".  This will be passed to all devices that run vyos.

Finally, you may have a device that is completely unique and none of the above
fit its situation.  In this case you can override the rancid_type in the
device itself.

# Custom Fields setup

Create a new "Custom Field"

Name: rancid_type

Label: Rancid Type

Type: Selection

Required: no

Assigned Models:
```
    dcim | device
    dcim | device role
    dcim | device type
    dcim | manufacturer
```

Values:

Default Value: None

Choices: airos, asa, casa, edgemax, edgerouter, foundry, ios, juniper, no_rancid, nxos, routeros, vyos, zhone, zynos

# Export template setup

Create a new "Export Template"

Name: gen_routerdb

Description: Generate Rancid router.db file

Attachment: no

Assigned Models: dcim | device

Template:
```
{% for device in queryset %}
      {%- set type = device.cf.rancid_type
                    or device.device_role.cf.rancid_type
                    or device.device_type.cf.rancid_type
                    or device.device_type.manufacturer.cf.rancid_type
       -%}
      {% if type and type != 'no_rancid' and device.status == 'active' and device.primary_ip4 -%}
{{ device }};{{ type }};up
{% endif %}
{%- endfor %}
```

# Setup a cron job for the update_routerdb script on your rancid host

```
0 */4   * * *   rancid /usr/local/bin/update_routerdb && /rancid/bin/rancid-run
```



