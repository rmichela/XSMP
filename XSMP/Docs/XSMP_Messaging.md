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
+ *Message Frame* - the social history message container that establishes message linkage, identity, signature, and confidentiality.
+ *Content Message* - A message in a social history containing information intended for user presentation.
+ *Control Message* - A message in a social history containing metadata about a user.

## Social history structure
### Message Format
A user's social history is made up a directed acyclic graph of digitally signed messages. Messages come in two types: control and content messages. Control messages are used to exchange metadata about the user, such as friends, identity, and demographics. Content messages make up the portion of the social history intended for consumption by others.

All messages are wrapped in a message frame, which is used to establish message linkage, identity, signature, and confidentiality. Message structures in this document are specified in Abstract Syntax Notation One basic notation [ASN.1]. The message frame and all message bodies MUST be serialized using ASN.1 Distinguished Encoding Rules [DER]. 

The message frame is defined as: 

    Message-Frame ::= SEQUENCE {
        id SHA256,
        parents SEQUENCE OF Parent-Info,
        author Key-Info,
        content OCTET STRING, --content of the message--
        signature Digital-Signature,
        encryption Optional-Encryption
    }

    SHA256 ::= OCTET STRING(SIZE(32))

    Parent-Info ::= SEQUENCE {
        parent-type Parent-Type,
        parent-id SHA256
    }

    Parent-Type ::= ENUMERATED {
        last-message(0),
        last-control-message(1),
        last-same-message(2),
        reply-to(3),
        other(4)
    }

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
        key-hash SHA256,
        encrypted-message-key OCTET STRING
    }

### Message IDs
Message IDs are the [SHA256] hash of the message content. They MUST be computed as follows and stored in the 'id' field of the message frame:

    SHA256(parents + author + body)

### Message Linkage
Messages in a social history link together into a graph using the `parents` field in the message frame. The first message in the social history graph MUST have exactly one parent made of twenty zeroed bytes. For all other messages, the `parents` field MUST point to the SHA256 id of at least one prior message. Multiple parent messages MAY be used to ease traversal of the social history graph. All parent ids MUST be unique.

Content messages can be either original content posted by the owner of a social history, or a reply to an existing content message. Replies to an existing content message MUST include the message id of the message being replied to in the parents field. Original content MUST include the message id of the most recent original content message or control message posted by the social history owner in the parents field.

Control messages MUST include the message id of the most recent original content message or control message posted by the social history owner in the `parents` field. Additionally, control messages MUST include the message id of the most recent control message posted by the social history owner in the parents field. Control messages may also include additional references to prior control messages of the same type to ease aggregation.

All parent links MUST also include a valid Parent-Type enumeration value in the `parent-type` field to indicate the meaning of the parent link. Parent-type enumeration values have the following meaning:

+ `last-message` - Points to the most recent non-reply message in the social history. Used to establish a total ordering over the social history.
+ `last-control-message` - Points to the most recent control message. Used to link control messages together.
+ `last-same-message` - Points to the most recent message of the same message type. Used to link messages of the same type together for convenience.
+ `reply-to` - Points to a content message for which the message is a reply to. Used to establish reply chains.
+ `other` - Points to some other message for a purpose defined by the message type.

### Message Signing
All messages in the social history MUST be signed by the [X509] private RSA keys of the users that created them. Each message MUST also include either the entire X.509 public key certificate of its author, or SHA256 fingerprint of the author's X.509 public key certificate stored in the `author` field of the message frame. As an optimization, the entire X509 certificate only needs to be included once in the social history.

When signing a message frame, the name of the algorithm used to compute the signature must be included along with the digital signature itself. The digital signature of the message frame MUST be computed as

    Sign(id + parents + author + body)

using the RSA-OAEP signature algorithm defined in [PKCS#1].

### Message Content
The `content` field of the message frame stores the body of the message frame. A content or control message is first serialized into DER, and then the resulting binary blob is inserted into `content` field. The structure of the `content` field is as follows:

    Content ::= SEQUENCE {
        media-type PRINTABLE STRING,
        content-data OCTET STRING,
        is-amendment BOOLEAN
    }

The message frame's `content` field is an open-ended sequence of bytes of any valid [RFC2046] media type, including `multipart`. Any data associated with the message is stored in the `content-data` field, and the associated content type is stored in the `media-type` field. User agents MUST implement all control messages specified in this document, along with `multipart/mixed`, `multipart/apternative`, and `multipart/related` [RFC2387] messages. 

If the `is-amendment` flag is true, the message represents an edit to a previous message instead of a stand-alone message. User agents SHOULD show the amended content to the user instead of the original content, but note that an edit was made. Replies to either the original or amended content should be presented as the same thread of discussion.

Different user agents may choose which other types of message media types to display to the user. For example, a photo oriented program could show only the photos in a social history while a social bookmarking program might show only hyperlinks. Regardless of the content of each message, all user agents MUST replicate every message in the social history - even the ones they cannot decode.

### Message Encryption
All content messages and some control messages can be encrypted. Encrypting a message is used to restrict which parties can read the message content. The message frame itself remains unencrypted to preserve the integrity of the social history and allow third parties to replicate a social history without giving them insight into its content. To encrypt a message, the message is serialized into DER and the resulting blob is encrypted with a unique symmetric message key. The resulting ciphertext data is then inserted into the message frame's content field in the place of the plaintext DER serialized data.

Encrypted messages can be shared with multiple individuals or groups. When sharing with an individual, the unique symmetric message key is encrypted with the individual's public key. When sharing with a group, the unique symmetric message key is encrypted with a pre-shared symmetric group key.

#### Message Encryption Algorithm
1. Serialize the message into a blob using DER.
2. Generate a 32-byte random message key unique to this message. Implementations SHOULD use a cryptogrphcally secure random number generator to generate the message key.
3. Apply the [AES-256] encryption algorithm to the serialized message content using the unique message key to generate the message ciphertext.
4. Insert the message ciphertext into the `content` field of the message frame data structure.
5. For each party that can decrypt the message, add an entry in the `message-keys` dictionary in the message frame so that each party can decrypt the message key. 
    1. For individuals, encrypt the unique message key using that individual's RSA public key extracted from their X.509 certificate using the OAEP padding scheme [PKCS#1]. Store the encrypted message key in the `encrypted-message-key` field. Hash the individual's X.509 certificate using SHA256 and store the resulting hash in the `key-hash` field.
    2. For groups, apply the AES-256 symmetric encryption algorithm to the unique message key, using the group's 32-byte pre-shared symmetric group key as the block cipher key. Store the encrypted message key in the `encrypted-message-key` field. Hash the pre-shared symmetric group key using SHA256 and store the resulting hash in the `key-hash` field.

#### Message Decryption Algorithm
1. For each group the user has been added, hash the pre-shared symmetric group key using SHA256. Also hash the user's X.509 public key certificate using SHA256.
2. Search the `message-keys` dictionary in the message frame for a matching `key-hash`. If a matching `key-hash` is found, the user has been identified as a recipient of the message.
3. Extract the matching `encrypted-message-key` field from the `message-keys` dictionary and decrypt it using either the identified pre-shared symmetric group key or the user's X.509 RSA private key using the OAEP padding scheme.
4. Use the decrypted message key to decrypt the message from the `content` field of the message frame.
5. Decode the resulting plaintext using DER encoding to get the content of the message.

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
[AES-256] Advanced Encryption Standard (AES). http://csrc.nist.gov/publications/fips/fips197/fips-197.pdf

[ASN.1] Abstract Syntax Notation One (ASN.1): Specification of Basic Notation. http://www.itu.int/rec/T-REC-X.680-200811-I/en

[DER] ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER). http://www.itu.int/rec/T-REC-X.690/en

[PKCS#1] Public-Key Cryptography Standards (PKCS) #1: RSA Cryptography Specifications Version 2.1. https://www.ietf.org/rfc/rfc3447.txt

[RFC2046] Multipurpose Internet Mail Extensions (MIME) Part Two: Media Types. http://tools.ietf.org/html/rfc2046

[RFC2387] The MIME Multipart/Related Content-type. https://tools.ietf.org/html/rfc2387

[SHA256] FIPS PUB 180-4 Secure Hash Standard. http://csrc.nist.gov/publications/fips/fips180-4/fips-180-4.pdf

[RFC2119] Key words for use in RFCs to indicate Requirement Levels. https://tools.ietf.org/html/rfc2119

[X509] Internet X.509 Public Key Infrastructure Certificate and CRL Profile. https://www.ietf.org/rfc/rfc2459
