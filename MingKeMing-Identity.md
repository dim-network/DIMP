# Ming Ke Ming (名可名) -- Identity Module

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/moky/DIMP/blob/master/LICENSE)
[![Version](https://img.shields.io/badge/alpha-0.1.0-red.svg)](https://github.com/moky/DIMP/wiki)

This document introduces a common **Identity Module** for decentralized user identity authentication.

Copyright &copy; 2018 Albert Moky

- [Meta](#meta)
    - [Version](#meta-version)
    - [Seed](#meta-seed)
    - [Key](#meta-key)
    - [Fingerprint](#meta-fingerprint)
- [ID](#id)
    - [Type](#id-type)
    - [Name](#id-name)
    - [Address](#id-address)
    - [Terminal](#id-terminal)
    - [Number](#id-number)
- [Samples](#samples)

## <span id="meta">0. Meta</span>

The **Meta** was generated by your **private key**, it can be used to build a new ID for entity, or verify the ID/PK pair.

It consists of 4 fields:

| Field       | Description                   |
| ----------- | ----------------------------- |
| version     | Meta Algorithm Version        |
| seed        | Entity Name                   |
| key         | Public Key                    |
| fingerprint | Signature to generate address |

### <span id="meta-version">0.0. Version</span>

Now it's value always equal to ```0x01```.

### <span id="meta-seed">0.1. Seed</span>

A string as same as **ID.name** for generate the fingerprint.

### <span id="meta-key">0.2. Key</span>

A **public key** (PK) was binded to an ID by the **Meta Algorithm**.

### <span id="meta-fingerprint">0.3. Fingerprint</span>

THe **fingerprint** field was generated by your **private key** and **seed**:

````javascript
fingerprint = sign(seed, SK);
````

## <span id="id">1. ID</span>
The **ID** is used to identify an **entity**(account/group). It consists of 3 fields and 2 extended properties:

| Field       | Description                   |
| ----------- | ----------------------------- |
| name        | Same with meta.seed           |
| address     | Unique Identification         |
| terminal    | Login point, it's optional.   |
| type        | Network type                  |
| number      | Search Number                 |

The ID format is ```name@address[/terminal]```.

### <span id="id-type">1.0. Type</span>

The **network type** of a person is ```8```, and group is ```16```:

```javascript
// Network ID
enum {
    MKMNetwork_Main  = 0x08, // (Person)
    MKMNetwork_Group = 0x10, // (Multi-Persons)
};
```

### <span id="id-name">1.1. Name</span>
The **Name** field is a username, or just a random string for group:

1. The length of name must more than 1 byte, less than 32 bytes;
2. It should be composed by a-z, A-Z, 0-9, or charactors '_', '-', '.';
3. It cannot contain key charactors('@', '/').

```javascript
// Name examples
userName  = "Albert.Moky";
groupName = "Group-1234567890";
```

### <span id="id-address">1.2. Address</span>

The **Address** field was created with the **Fingerprint** in Meta and a **Network ID**:

```javascript
// Address algorithm
function btcBuildAddress(fingerprint, network) {
    hash       = ripemd160(sha256(fingerprint));
    check_code = sha256(sha256(network + hash)).prefix(4);
    address    = base58(network + hash + code);
    return address;
}
```

When you get a meta for the entity ID from the network,
you must verify it with the consensus algorithm before accept its **key**.

```javascript
// Meta algorithm
function isMatch(ID, meta) {
    // 1. check 'seed', 'key' & 'fingerprint' in meta with ID.name
    if (meta.seed != ID.name) {
        return false;
    }
    if (!verify(meta.seed, meta.fingerprint, meta.key)) {
        return false;
    }
    
    // 2. build address with meta, compare it with ID.address
    address = btcBuildAddress(meta.fingerprint, ID.address.network);
    if (address != ID.address) {
        return false;
    }
    
    // 3. if all of the above matches, get public key from meta
    ID.publicKey = meta.key;
    return true;
}
```

### <span id="id-terminal">1.3. Terminal</span>

A resource identifier as **Login Point**.

### <span id="id-number">1.4. Number</span>

A **Search Number** is defined for easy remember. Its value is converted from the **check code** of the address. It's greater than **0** and smaller than **2<sup>32</sup> (4,294,967,296)**.

## <span id="samples">2. Samples</span>

### ID

```javascript
/* ID examples */
ID1 = "hulk@4YeVEN3aUnvC1DNUufCq1bs9zoBSJTzVEj"; // Immortal Hulk
ID2 = "moki@4WDfe3zZ4T7opFSi3iDAKiuTnUHjxmXekk"; // Monkey King
```

### Meta

```javascript
/* Meta example: hulk@4YeVEN3aUnvC1DNUufCq1bs9zoBSJTzVEj */
{
    version     : 0x01,
    seed        : "hulk",
    key         : {
        algorithm  : "RSA",
        data       : "-----BEGIN PUBLIC KEY-----\nMIGJAoGBALB+vbUK48UU9rjlgnohQowME+3JtTb2hLPqtatVOW364/EKFq0/PSdnZVE9V2Zq+pbX7dj3nCS4pWnYf40ELH8wuDm0Tc4jQ70v4LgAcdy3JGTnWUGiCsY+0Z8kNzRkm3FJid592FL7ryzfvIzB9bjg8U2JqlyCVAyUYEnKv4lDAgMBAAE=\n-----END PUBLIC KEY-----",
        // other parameters
        keySize    : 1024,
        encryption : "PKCS1",
        signature  : "PKCS1v15SHA256"
    },
    fingerprint : "jIPGWpWSbR/DQH6ol3t9DSFkYroVHQDvtbJErmFztMUP2DgRrRSNWuoKY5Y26qL38wfXJQXjYiWqNWKQmQe/gK8M8NkU7lRwm+2nh9wSBYV6Q4WXsCboKbnM0+HVn9Vdfp21hMMGrxTX1pBPRbi0567ZjNQC8ffdW2WvQSoec2I="
}
```

(All data encode with **BASE64** algorithm as default, excepts the **address**)
