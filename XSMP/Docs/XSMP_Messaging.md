# Extensible Social Media Protocol Messaging

## Introduction
### Purpose
The XSMP Messaging protocol defines a user's social history as a directed acyclic graph of social history nodes. As interacts with a social system, new nodes are appended to the end of her social history graph. As the user and her friends interact with a particular post, new nodes are added to the graph as children of the initial post. The structure and linking of nodes was inspired by the Git version control system.
### Requirements
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [RFC2119].
### Terminology
+ *Social History* - the immutable directed acyclic graph of all content and control messages making up the totality of a user's social interactions.

## Social history structure
### Message Format
A user's social history is made up a directed acyclic graph of digitally signed messages. Messages come in two types: control and content messages. Control messages are used to exchange metadata about the user, such as friends, identity, and demographics. Content messages make up the portion of the social history intended for consumption by others.

All messages are wrapped in a message frame, which is used to establish message linkage, identity, and signature. Message structures in this document are specified in Abstract Syntax Notation One basic notation [ASN.1]. The message frame is defined as: 

    Message-Frame ::= SEQUENCE {
        id SHA256,
        parents SEQUENCE OF SHA256,
        author Key-Info,
        body OCTET STRING, --content of the message--
        signature Digital-Signature,
        encryption Optional-Encryption
    }

    SHA256 ::= OCTET STRING(SIZE(32))

    Key-Info ::= CHOICE {
        x509-certificate OCTET STRING,
        x509-fingerprint SHA256
    }

    Digital-Signature ::= SEQUENCE {
        digest-method PRINTABLE STRING, --Name of the algorithm used to hash--
        signature OCTET STRING    --Encrypted message digest--
    }

    Optional-Encryption ::= CHOICE {
        message-keys SEQUENCE OF Message-Key,
        none NULL
    }

    Message-Key ::= SEQUENCE {
        friend-key-hash SHA256,
        encrypted-message-key OCTET STRING
    }

### Message IDs
Message IDs are the [SHA256] hash of the message content. They MUST be computed as follows:

    SHA256(parents + author + body)

### Message Linkage
Messages in a social history link together into a graph using the parents field in the message frame. The first message in the social history graph MUST have exactly one parent made of twenty zeroed bytes. For all other messages, the parents field MUST point to the SHA256 id of at least one prior message. Multiple parent messages MAY be used to ease traversal of the social history graph. All parent ids MUST be unique.

Content messages can be either original content posted by the owner of a social history, or a reply to an existing content message. Replies to an existing content message MUST include the message id of the message being replied to in the parents field. Original content MUST include the message id of the most recent original content message or control message posted by the social history owner in the parents field.

Control messages MUST include the message id of the most recent original content message or control message posted by the social history owner in the parents field. Additionally, control messages MUST include the message id of the most recent control message posted by the social history owner in the parents field. Control messages may also include additional references to prior control messages of the same type to ease aggregation.

### Message Signing
All messages in the social history MUST be signed by the [X509] private keys of the users that created them. Each message MUST also include either the entire X509 public key certificate of its author, or SHA256 fingerprint of the author's X509 public key certificate. As an optimization, the entire X509 certificate only needs to be included once in the social history.

When signing a message frame, the name of the algorithm used to compute the signature must be included along with the digital signature itself. The digital signature of the message frame MUST be computed as

    Encrypt(SHA256(id + parents + author + body))

### Message Encryption

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

## References
[ASN.1] Abstract Syntax Notation One (ASN.1): Specification of Basic Notation. http://www.itu.int/rec/T-REC-X.680-200811-I/en

[SHA256] FIPS PUB 180-4 Secure Hash Standard. http://csrc.nist.gov/publications/fips/fips180-4/fips-180-4.pdf

[RFC2119] Key words for use in RFCs to indicate Requirement Levels. https://tools.ietf.org/html/rfc2119

[X509] Internet X.509 Public Key Infrastructure Certificate and CRL Profile. https://www.ietf.org/rfc/rfc2459
