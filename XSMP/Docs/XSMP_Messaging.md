# Extensible Social Media Protocol Messaging

## Introduction
### Purpose
The XSMP Messaging protocol defines a user's social history as a directed acyclic graph of social history nodes. As interacts with a social system, new nodes are appended to the end of her social history graph. As the user and her friends interact with a particular post, new nodes are added to the graph as children of the initial post. The structure and linking of nodes was inspired by the Git version control system.
### Requirements
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 [34].

An implementation is not compliant if it fails to satisfy one or more
of the MUST or REQUIRED level requirements for the protocols it
implements. An implementation that satisfies all the MUST or REQUIRED
level and all the SHOULD level requirements for its protocols is said
to be "unconditionally compliant"; one that satisfies all the MUST
level requirements but not all the SHOULD level requirements for its
protocols is said to be "conditionally compliant."
### Terminology
+ *Social History* - the immutable directed acyclic graph of all content and control messages making up the totality of a user's social interactions.

## Social history structure
### Message Format
### Message Linkage

## Identity Control Messages
### ANNOUNCE
### IDENTITY

## Friend Control Messages
### FRIEND_REQUEST
### FRIEND_ADD
#### Friend Keys
### FREQUENCY_HOP
### DELEGATE

## Content Messages
### CONTENT