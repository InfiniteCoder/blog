---
date: "2018-05-28T00:00:00Z"
published: true
aliases:
    - /2018-05-28-finding-domains/
subtitle: Automate boring task of trial and error while going through hundreds of
  domain names.
title: Finding short, good domain names using AWS API ( Free of Cost )
---

Recently, I was on a lookout for a domain name for personal use. This domain name will probably never be seen by more than 2 eyes, so I didn't care exactly what TLD it has, or even what it's FQDN is. 

That said, who doesn't like a short, memorable and meaningful domain name? foo.bar is always more favourable than say, verylongname.engineering (And yes, both are valid domain names). However, trying out each possible domain by typing it into some registrars search bar was too boring and time consuming. More often than not, you would come across unavailable domains. Hence I decided to automate the process by using some API.

## Introduction

### The Task
The task was to go through thousands of possible domain names, and list out those which are available, so I could choose one of them. Once that is done, I would select whichever I like most.

### Finding an API
There aren't many APIs out there to find if a domain is available or not, and the one's that are available, are way too costly for me to try out thousands of combinations. Eventually, I came across Amazon's Route 53 Domain API. This API provides endpoints for tasks such as registering, renewing, transfering domains. One of the operations is [CheckDomainAvailability](https://docs.aws.amazon.com/Route53/latest/APIReference/API_domains_CheckDomainAvailability.html). In their own words,

> This operation checks the availability of one domain name.

Exactly what I wanted. As a bonus, I found out this API is free. Although the "free"dom did came at a cost, which I have mentioned below.

## The Code
Amazon provides SDKs to use the API in various languages. As this project was going to be a one off script, I decided to go with Python. The boto3 package provides an easy to use abstraction for the API. It also comes with a quite good [documentation](https://boto3.readthedocs.io/en/latest/reference/services/route53domains.html#Route53Domains.Client.check_domain_availability).

### Requirements

First, I needed to create an [IAM user](https://console.aws.amazon.com/iam) in AWS. For this API, we need to enable the [AmazonRoute53DomainsReadOnlyAccess](https://console.aws.amazon.com/iam/home?#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonRoute53DomainsReadOnlyAccess) permission for the user.

I used [AWS Command Line Interface(CLI)](https://aws.amazon.com/cli/) to easily set up the credentials for the IAM user. Boto3 uses these details when making API calls. `awscli` and `boto3` can be installed using pip.

    pip install awscli boto3

Next task was to configure aws cli. This is pretty straight forward, simply run,

    aws cli configure

and enter access key, secret key, default region name and default output format.

### Script
The script is quite simple. It reads words from the dict/cracklib file that is available on most Linux distros. Then it gets only those words whose length matches the requirement(say 3), and then checks the availability for all given tld's. If reply is "AVAILABLE", it prints out the domain name.

``` python
import boto3
import sys

# number of characters
size = int(sys.argv[1]) + 1

# setup client
client = boto3.client('route53domains')

# open dict file
words = open("/usr/share/dict/cracklib-small", "r")
for word in words:
    if len(word) == size:
        for domain in [word.strip() + ".trade", word.strip() + ".loan"]:
            try:
                response = client.check_domain_availability(
                    DomainName=domain,
                )
            except Exception as ex:
                print("error occured for: " + domain)
                print(ex)
            else:
                status = response["Availability"]
                if status == "AVAILABLE":
                    print(domain)

```

## Issues

### Rate Throttling
As mentioned earlier, the API is free to use. But this freedom comes at a cost. The API applies rate throttling. Frequently, I would see this error,

    An error occurred (ThrottlingException) when calling the CheckDomainAvailability operation (reached max retries: 4): Rate exceeded

Initially, to speed up the script, I had created a multi threaded variant which used thread pools. However, that simply made the script hit rate throttling more often. Basically, rate throttling becomes a function of number of threads, with throttling being directly proportional to number of threads. So I decided to use the single threaded variant.

Secondly, I needed to create a way to retry the failed calls. For this, I added the exception messages as in the above script. Then I would parse the output for this errors, and pass the domains to a slightly modified version of the script( which reads domains instead of words, from a given file).

The commands for retrying are as such

``` sh
python dna.py 4 | tee 4-letter-dn														# get list
cat 4-letter-dn | egrep "^[a-z]{4}.(loan|trade)$" >> available							# gets available
cat 4-letter-dn | grep "error occured"|sed "s/error occured for: //g" |LC_COLLATE=C egrep "[a-z]{4}.(loan|trade)" > retry-these													  # gets errored
python dna-retry.py retry-these | tee 4-letter-dn   									# retry errored
```

### Lack of Support for TLDs

This API doesn't support many TLDs, some of which are quite nice, like `.site` or `.space`. Well, one can't complain too much about free stuff.

## Conclusion

Using this script, I found some 3 letter and hundreds of 4 letter domains for `.loan` and `.trade` TLDs. Manually, this could have taken me longer than it would take Voyager 1 to reach interstellar space. 
