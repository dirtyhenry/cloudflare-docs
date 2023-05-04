---
pcx_content_type: troubleshooting
source: https://support.cloudflare.com/hc/en-us/articles/200168876-Email-undeliverable-when-using-Cloudflare
title: Email issues
meta:
    title: Troubleshooting email issues
---

# Troubleshooting email issues

If you have issues sending or receiving mail, follow these troubleshooting steps.

## Are your records correct?

Consult with your mail administrator or mail provider to ensure you have valid DNS record content.

## Are DNS records missing?

Contact your mail administrator to confirm the DNS records for your domain are correct. Refer to our guide on [managing DNS records in Cloudflare](/dns/manage-dns-records/how-to/create-dns-records) if you need assistance to add or edit DNS records.

## Do you have CNAME Flattening enabled?

When set to [Flatten all CNAMEs](/dns/cname-flattening/set-up-cname-flattening/) in your Cloudflare DNS settings, queries to all `CNAME` records will flatten to an `A` record; no `CNAME` records will be returned.

Also, if `CNAME` records are not returned by the queried nameserver (sometimes nameservers will return TXT records), this may result in nothing being returned when **_Flatten all CNAMEs_** is enabled. Changing to _**Flatten at the root**_ should fix any issues with your CNAME records not being returned.

## Is Cloudflare Spectrum enabled on your account?

Cloudflare does not proxy traffic on port 25 (SMTP) unless [Cloudflare Spectrum](/spectrum/reference/configuration-options#smtp) is enabled and configured to proxy email traffic across Cloudflare. If you do not have Spectrum enabled, then no email traffic (SMTP) will actually pass through Cloudflare, and we will simply resolve the DNS. This also means that any DNS record used to send email traffic must be DNS-only to bypass the Cloudflare network. Check [Identifying subdomains compatible with Cloudflare's proxy](/dns/manage-dns-records/reference/proxied-dns-records) for more details.

## Contact your mail provider for assistance

If your email does not work shortly after editing DNS records, contact your mail administrator or mail provider for further assistance in troubleshooting so that data about the issue can be provided to Cloudflare support.

## dc-######### subdomain

The dc-##### subdomain is added to overcome a conflict created when your `SRV` or `MX` record resolves to a domain configured to [proxy](/dns/manage-dns-records/reference/proxied-dns-records) to Cloudflare.

Therefore, Cloudflare will create a `dc-#####` DNS record that resolves to the origin IP address. The `dc-#####` record ensures that traffic for your `MX` or `SRV` record is not proxied (it directly resolves to your origin IP) while the Cloudflare proxy works for all other traffic.

For example, before using Cloudflare, suppose your DNS records for mail are as follows:

`example.com MX example.com` `example.com A 192.0.2.1`

After using Cloudflare and proxying the `A` record, Cloudflare will provide DNS responses with a Cloudflare IP (`203.0.113.1` in the example below):

`example.com MX example.com` `example.com A 203.0.113.1`

Since proxying mail traffic to Cloudflare would break your mail services, Cloudflare detects this situation and creates a `dc-#####` record:

`example.com MX dc-1234abcd.example.com` `dc-1234abcd.example.com A 192.0.2.1` `example.com A 203.0.113.1`

Removing the `dc-######` record is only possible via one of these methods:

-   If no mail is received for the domain, delete the `MX` record.
-   If mail is received for the domain, update the `MX` record to resolve to a separate `A` record for a mail subdomain that is not proxied by Cloudflare:

`example.com MX mail.example.com` `mail.example.com A 192.0.2.1` `example.com A 203.0.113.1`

{{<Aside type="warning">}}
If your mail server resides on the same IP as your web server, your MX
record will expose your origin IP address.
{{</Aside>}}

___

## Best practices for MX records on Cloudflare

{{<render file="_email-record-origin-ip.md" productFolder="learning-paths">}}