# Extensible Social Media Protocol (XSMP)

## Overview
XSMP is a protocol for building peer-to-peer, federated social media systems. 

The XSMP protocol is divided into four main areas:
1. _XSMP Messaging_ - Specifies the message structures that make up a user's immutable social history.
2. _XSMP Identity_ - Specifies how X.509 certificates work with XSMP.
3. _XSMP Transport_ - Specifies the gossip protocol used to exchange XSMP messages between nodes.
4. _XSMP Discovery_ - Specifies how XSMP nodes find each other.

## Goals
XSMP exists to fulfill the following goals:
1. _Nobody but you owns your social history_ - Your social history should not belong to a corporation. Your social history should be portable and you should be able to manage it directly if you wish from your own computer.
2. _Social systems should not be all or nothing_ - Social systems should work together to share your social profile, not carve it up and lock it away in walled gardens and data silos. Social systems should be federated and forced to compete on quality of service, not quantity of users.
3. _Users should only be as public as they wish to be_ - How much you share and to whom you share should be completely in your control. Users should have fine-grained control over where their information is used and over who can see their content.
4. _Users should be as anonymous as they wish to be_ - Users should have complete control over how much or how little of their identity is available to others. A user should be able to actively participate in a social setting without revealing anything at all about themselves.