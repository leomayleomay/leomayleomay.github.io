---
layout: post
title: "open source the BPDUKPT"
date: 2013-05-07 22:49
comments: true
categories: OSS, Objective-C, DUKPT
---

I am happy to announce I just released the v0.1 of [BPDUKPT](https://github.com/leomayleomay/BPDUKPT).

We've been processing credit card payments from the iOS device more and more, and we are facing a common problem, ___security___. The genius invented the [DUKPT](http://en.wikipedia.org/wiki/Derived_unique_key_per_transaction) to ensure the credit card data will never be hijacked by some middleman between our client app and our server, that's really great for the card holders. But for the developer, we've got no luck to have a ready-to-go solution to decrypt the magnetic data captured from the card reader.

Here comes the [BPDUKPT](https://github.com/leomayleomay/BPDUKPT), you can decrypt magnetic strips like a boss.


```objective-c
#include "BPDUKPTParser.h"

NSString *magData = @"foobarbaz"; // this is the magnetic strips data captured from the card reader
BPDUKPTIDTechParser *parser = [[BPDUKPTIDTechParser alloc] initWithHID:magData];
BPDUKPTParsingResult *result = [parser parse];

NSLog(@"the encrypted track 1 is: %@", result.track1);
NSLog(@"the encrypted track 2 is: %@", result.track2);
NSLog(@"the encrypted track 3 is: %@", result.track3);
NSLog(@"the KSN is: %@", result.ksn);
```

Tell [me](mailto:leomayleomay@gmail.com) what you think about it, and please help to make it better.



