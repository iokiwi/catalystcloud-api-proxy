# Catalyst Cloud API Jumphost / Proxy

## The Problem

Catalyst Cloud API's are not open to the internet.

> As an additional security measure, the Catalyst Cloud APIs only accept requests from whitelisted IP addresses. If you have provided an IP address during sign up, you should be able to reach the APIs from that IP. Otherwise, you can open a support request via the dashboard at any time to request a change to the white-listed IPs.
>
> -- <cite>Source: https://docs.catalystcloud.nz/getting-started/access.html#api-access</cite>

While, as per the documentation, you can open a support request to get your IP Address whitelisted this is a pain in the ass for many reasons and causes unecessary friction in some of the following situations:

 * You move around often or are simply away from your whitelisted network.
 * You don't have a static IP address. Most residential internet connections don't have static IP's unless you specifically request one any pay an aditional montly fee.
 * You want to use a popular cloud based CI/CD platform to automate with Catalyst Cloud

There are many ways to work around the restriction but the final situation outlined above is, in my opinion, the easiest and most compelling reason to go down the route of the solution proposed here.

If you take [Github Actions](https://github.com/features/actions) for example, there are more than 2000 subnets that your CI/CD job could potentially be running on all of which would need to be whitelisted.

```
$ curl -s  https://api.github.com/meta | jq '.actions | length'
2076
```

I've tried several approaches to work around the limitations of the whitelist including:

 * Working remotely on a server in the cloud with whitelisted access to the API's
 * Setting up a bastion and ssh tunneling through them https://github.com/iokiwi/cloud_tunnel

Both of these solutions were annoying and cumbersome and fell short as soon as I wanted to
start doing CI/CD using github actions.

What I really wanted was something lightweight, simple, easy to setup and something that I could set and forget that would give me the same experience as working from a whitelisted network wherever I go.

This solution comes the closest to that.

## Quick Start / Proof of Concept

This works for creating a jumphost in the `nz-hlz-1` region

1. Go to https://dashboard.cloud.catalyst.net.nz/project/stacks/
2. Click Launch Stack
3. Use the [heat-teamplate-nz-hlz-1.yml](heat-template-nz-hlz-1.yml) file provided in this repo as the template
4. Navigate to the stack in the console and find `floating_ip` in the outputs section
5. Put the value of `floating_ip` into your machines hosts file. (See [How to Edit Your Hosts File on Windows, Mac, or Linux](https://www.howtogeek.com/howto/27350/beginner-geek-how-to-edit-your-hosts-file/))
    ```config
    <your_floating_ip> api.nz-hlz-1.catalystcloud.io object-storage.nz-hlz-1.catalystcloud.io api.cloud.catalyst.net.nz
    ```
6. Where previously the following command would have hung and timed out, you should now be able to reach the API's from your local machine

    ```bash
    $ curl https://api.nz-hlz-1.catalystcloud.io:5000
    {"versions": {"values": [{"status": "stable", "updated": "2016-04-04T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.6", "links": [{"href": "https://api.nz-hlz-1.catalystcloud.io:5000/v3/", "rel": "self"}]}, {"status": "stable", "updated": "2014-04-17T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v2.0+json"}], "id": "v2.0", "links": [{"href": "https://api.nz-hlz-1.catalystcloud.io:5000/v2.0/", "rel": "self"}, {"href": "http://docs.openstack.org/", "type": "text/html", "rel": "describedby"}]}]}}
    ```
