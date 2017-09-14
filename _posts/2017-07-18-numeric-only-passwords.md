---
layout: post
title:  Using numeric only passwords and pins
author: Luca Ferrari
categories:
- security
- password
- development
permalink: /:year/:month/:day/:title.html
---
Are numerical only pins really secure? I don't think so.

## Using Numeric Only Passwords
-----

It is a fact that a lot of bank accounts are protected by a *numeric only* password, usually a quite long one, but a truly numeric set of characters.
To be honest, bank accounts usually use a mixed approach to identify users that need, in turn, to handle a physical device/token that generates a so called *One Time Password* (OTP), this introduces some kind of randomization into the process that helps solving the password steal problem.
Nonetheless, the wrong approach many account providers apply is to use the OTP only for actions, leaving the reading capability to be protected only by the numeric password.
In other words, if you need to check your balance you will need only a numerical password, if you need to do a payment you will need also the OTP.
Of course not all the banks apply the above approach, and for instance my bank requires *always* the OTP for both login and every single operation that can mutate the final balance. Still, the protection is made by only numbers and never letters, symbols or a mix of them.

Talking with another IT professional about the problem, I got hte answer that a numeric password is enough safe in order to gain read access to the bank account, because after all it is denied any possibility to perform any illegal operation.

I tend to become aggressive when such kind of answer come to my mind...

While it is true that disabling any *dangerous* operation trhu an OTP can be a safe approach, leaving the reading capability masked behind a single numeric password is a **huge** mistake. I will not thank my bank because a thief cannot perform any operation even after stealing my password, I blame the bank for **setting me as a potential target once the thief has discovered the balance of my account**!
We are not talking here about a single action, about a guess-and-try approach, we are talking about people organized to steal passwords (or to sniff them, or to retro-engineer them) and able to find out **who** is the target for the next step: stealing the OTP device.

I don't care if I can read my balance in a matter of seconds or if I need a minute to insert three different password to see it, **I do care my personal data is secure** not matter what operation anybody can do on such data.

Therefore the point is that
1. using a single password to protect such data is always a bad idea;
2. worst than using a single password is using a single password with a limited alphabet, such as a numeric-only one.
