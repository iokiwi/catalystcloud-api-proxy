# Catalyst Cloud API Proxy

## The Problem

Catalyst Cloud API's are not open to the internet.

> As an additional security measure, the Catalyst Cloud APIs only accept requests from whitelisted IP addresses. If you have provided an IP address during sign up, you should be able to reach the APIs from that IP. Otherwise, you can open a support request via the dashboard at any time to request a change to the white-listed IPs.
>
> Source: https://docs.catalystcloud.nz/getting-started/access.html#api-access

While, as per the documentation, you can open a support request to get your IP Address whitelisted this is a pain in the ass for many reasons and causes unecessary friction in some of the following situations:

 1. If you don't have a static IP address (most residential internet connections assign dynamic or at best sticky IP's which aren't guaranteed)
 2. You are away from your whitelisted network
 3. You move around often or are simply
 4. You want to call the API from a CI/CD pipeline

There are many ways to work around the restriction but the final situation outlined above is, in my opinion, the most compelling reason to go down the route of the solution proposed here.

If you want to automate using, say github actions then you would need to get Catalyst to whitelist the whole IP range of githubs CI/CD infrastructure, conveniently detailed here: https://api.github.com/meta

```
$ curl -s  https://api.github.com/meta | jq '.actions | length'
2076
```
]
The endpoint return over 1700 IPv4 and 300 IPv6 subnets for a total of more than 2000 subnets which potentially all need to be whitelisted. Now rinse and repeat for every other major and minor CI/CD automation provider out there...yeah.

## Proof of Concept

This works for creating a jumphost in the `nz-hlz-1` region

1. Go to https://dashboard.cloud.catalyst.net.nz/project/stacks/
2. Click Launch Stack
3. Use the [heat-teamplate-nz-hlz-1.yml](heat-template-nz-hlz-1.yml) file provided in this repo as the template
4. Navigate to the stack in the console and find `floating_ip` in the outputs section
5. Put the value of `floating_ip` into your machines hosts file. See [Configuring your client to use the proxy](#configuring-your-client-to-use-the-proxy)
    ```config
    <your_floating_ip> api.nz-hlz-1.catalystcloud.io object-storage.nz-hlz-1.catalystcloud.io api.cloud.catalyst.net.nz
    ```
6. You should now be able to reach the API's from your local machine

    ```bash
    $ curl https://api.nz-hlz-1.catalystcloud.io:5000
    {"versions": {"values": [{"status": "stable", "updated": "2016-04-04T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.6", "links": [{"href": "https://api.nz-hlz-1.catalystcloud.io:5000/v3/", "rel": "self"}]}, {"status": "stable", "updated": "2014-04-17T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v2.0+json"}], "id": "v2.0", "links": [{"href": "https://api.nz-hlz-1.catalystcloud.io:5000/v2.0/", "rel": "self"}, {"href": "http://docs.openstack.org/", "type": "text/html", "rel": "describedby"}]}]}}
    ```

## Configuring your client to use the proxy

You just need to force your computer to resolve catalyst cloud api domains to
your proxy instead. The easiest way of doing this is by adding an entry to your machine's
hosts file.

```config
<your_floating_ip> api.nz-hlz-1.catalystcloud.io object-storage.nz-hlz-1.catalystcloud.io api.cloud.catalyst.net.nz
```

On ubuntu that looks like this

```
$ sudo nano /etc/hosts
```
```conf
127.0.0.1       localhost

######## Add your config here ###########

103.197.60.145 api.nz-hlz-1.catalystcloud.io object-storage.nz-hlz-1.catalystcloud.io api.cloud.catalyst.net.nz

#########################################

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

Your default hosts file might look a little bit different. Thats okay. For more information on how edit your host file on linux mac or windows see: [How to Edit Your Hosts File on Windows, Mac, or Linux](https://www.howtogeek.com/howto/27350/beginner-geek-how-to-edit-your-hosts-file/)

Of course, the real solution is for Catalyst Cloud to make their public API public.

See also: [Security through obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity)
