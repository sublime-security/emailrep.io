# emailrep.io Alpha API

## Summary:

EmailRep is a system of crawlers, scanners and enrichment services that collects data on email addresses, domains, and internet personas. 

EmailRep uses hundreds of data points from social media profiles, professional networking sites, dark web credential leaks, data breaches, phishing kits, phishing emails, spam lists, open mail relays, domain age and reputation, deliverability, and more to predict and answer these types of questions: 
* Is this email risky?
* Is this a throwaway account?
* Is there potential for this signup to commit fraud or abuse?
* What kind of online presence does this email have?
* Is this a trustworthy sender?

## URL: 

```
https://emailrep.io
```

## Authentication:

"key: [key]" header

## Endpoints:

* GET /[email] - query an email
* POST /report - report an email

### /[email]
Method: `GET`

Required Parameters:

* email - Email address to query

Optional Parameters:

* key - A valid API key enables you to have custom rate limits

Example:
```
$ curl -s emailrep.io/bill@microsoft.com
{
  "email": "bill@microsoft.com",
  "reputation": "high",
  "suspicious": false,
  "references": 61,
  "details": {
    "blacklisted": false,
    "malicious_activity": false,
    "malicious_activity_recent": false,
    "credentials_leaked": true,
    "credentials_leaked_recent": false,
    "data_breach": true,
    "last_seen": "03/01/2015",
    "domain_exists": true,
    "domain_reputation": "high",
    "new_domain": false,
    "days_since_domain_creation": 10232,
    "suspicious_tld": false,
    "spam": false,
    "free_provider": false,
    "disposable": false,
    "deliverable": true,
    "accept_all": true,
    "valid_mx": true,
    "spoofable": false,
    "spf_strict": true,
    "dmarc_enforced": true,
    "profiles": [
      "twitter",
      "linkedin",
      "instagram",
      "spotify",
      "vimeo",
      "angellist",
      "tumblr",
      "flickr",
      "myspace",
      "pinterest"
    ]
  }
}
```

## Response Details:

* `reputation`: high/medium/low/none
* `suspicious`: true/false (our best assessment based off all the information we have)
* `references`: total number of positive and negative sources of reputation. note that these may not all be direct references to the email address, but can include reputation sources for the domain or other related information
* `blacklisted`: the email is believed to be malicious or spammy
* `malicious_activity`: the email has exhibited malicious behavior (e.g. phishing or fraud)
* `malicious_activity_recent`: malicious behavior in the last 90 days (e.g. in the case of temporal account takeovers)
* `credentials_leaked`: credentials were leaked at some point in time (e.g. a data breach (source: haveibeenpwned.com), pastebin, dark web, etc.)
* `credentials_leaked_recent`: credentials were leaked in the last 90 days
* `data_breach`: the email was in a data breach at some point in time (source: haveibeenpwned.com)
* `last_seen`: the last date the email was observed in a breach, credential leak, or exhibiting malicious or spammy behavior ('never' if never seen)
* `domain_exists`: valid domain
* `domain_reputation`: high/medium/low/n/a (n/a if the domain is a free_provider, disposable, or doesn't exist)
* `new_domain`: the domain was created within the last year
* `days_since_domain_creation`: days since the domain was created
* `suspicious_tld`: suspicious tld
* `spam`: the email has exhibited spammy behavior (e.g. spam traps, login form abuse)
* `free_provider`: the email uses a free email provider
* `disposable`: the email uses a temporary/disposable service
* `deliverable`: deliverable
* `accept_all`: whether the mail server has a default accept all policy. some mail servers return inconsistent responses, so we may default to an accept_all for those to be safe
* `valid_mx`: has an MX record
* `spoofable`: email address can be spoofed (e.g. not a strict SPF policy or DMARC is not enforced)
* `spf_strict`: sufficiently strict SPF record to prevent spoofing
* `dmarc_enforced`: DMARC is configured correctly and enforced
* `profiles`: online profiles used by the email

### /report
Method: `POST`

Required Parameters:

* key - A valid API key is required
* email - The email address to report
* tags - The tags to be applied
  * `spam` - Unsolicited spam or spammy behavior (e.g. forum submissions)
  * `malicious` - Malicious intent of any kind. Superset of the below tags
  * `credential_phishing` - Credential phishing, brand impersonation
  * `bec` - Business email compromise, whaling, contact impersonation/display name spoofing
  * `brand_impersonation` - Impersonating a well-known brand (e.g. Paypal, Microsoft, Google, etc.)
  * `maldoc` - Malicious documents, either directly (attachemnts) or indirectly (free file hosting sites)
  * `romance_scam` - Romance scam, sextortion
  * `account_takeover` - Legitimate email has been taken over by a malicious actor
  * `threat_actor` - Threat actor/owner of phishing kit
  * `browser_exploit` - The hosted website serves an exploit
  * `generic_phishing` - Generic phishing, should only be used if others don't apply or a more specific determination can't be made or would be too difficult

Optional Parameters:

* description - Additional information and context
* timestamp - When the activity occurred. Defaults to now()
* expires - Number of h ours the email should be blacklisted. Defaults to no expiration unless account_takeover tag is specified, in which case the default is 14 days

Example:
```
curl -s 'https://emailrep.io/report' -H 'key: [api_key]' -d '{"email": "foo@bar.com", "tags": ["malicious", "credential_phishing"], "description": "Phishing email sent to accounting dept", "timestamp": 1562171178, "expires": 24 }'
{
  "status": "success"
}
```

## Use cases

Defensive:
* Detect targeted phishing attacks.
* Detect and prevent fraud.
* Detect throwaway accounts.
* Require additional layers of verification (MFA) during your signup flow to prevent abuse.
* Contextualize netflow and other products that analyze email addresses or related data.

Offensive (ethical):
* Conduct recon on a target email address for credential brute forcing.
* Construct targeted phishing attacks based off of target's social media profiles.
* Inform reputation of social engineering campaigns (higher reputation can help avoid the spam folder).
