---
title: "The Taming of the GPG Agent"
layout: post
date: 2015-08-26 22:01:08
comments: true
tags: [gpg-agent,tmux,tty,screen,ssh-agent,yubikey]
share: true
---

Recently I decided to start looking at my PGP/GPG setup again, in an effort to
make it more useful. Initially, I set out to satisfy my desire to be capable of
signing my git commits from any workstation I was on in a secure fashion.

On doing a bit more research, I realized I was selling myself short and added on
the requirement to implement token-based two factor authentication for SSH.
However, instead of using OTP functionality I elected to look into using a PGP
key designated for authentication purposes, with the secret key stored on a
SmartCard in order to accomplish this. The two factors being my physical
SmartCard, and the pin to unlock it.

Eventually (this took me a few months to settle on) I decided that I could
live with a [2048-bit Keypair](https://www.yubico.com/2015/02/big-debate-2048-4096-yubicos-stand/),
and elected to use a Yubikey Neo for my efforts.
