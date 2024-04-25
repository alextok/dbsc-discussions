```mermaid
sequenceDiagram
%%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
autonumber 1
participant W as W Relying Party
participant I as I IdP
participant B as B Browser
participant P as P Local Key Helper
participant A as A AttestationService
 
Note over W, A: Provisioning ...
A->>B: Policy enabling Local Key Helper
P->>A: Initial Enrollment (AIK?)
W->>A: Get Root CA
Note over W, A: Sign in...
W->>B: Start sign in (302)
B->>I: Load sign-in (follow the 302
 
I->>B: Sec-Session-GenerateKey ..., RP, HelperId
B->>B: Evaluate policy for (RP, IdP, HelperId)<br/> - RP can be '*' in policies
B->>P: Pre-gen key and attest
 
P->>P: Generate Key
 
P->>A: Get Key Attestation (publicKey, AIK)
A->>B: Return PubKey Certificate
B->>B: Remember this key is for RP (and maybe path)
B->>I: PubKey Cert/Binding statement
 
I->>B: Sign in ceremony
B->>I: Sign done
 
I->>B: Auth tokens
B->>W: Auth tokens
 
Note over W, A: Initiate DBSC ...
W->B: StartSession (challenge, tokens)
B->P: Request Sign JWT (uri, challenge, tokens?)
P->B: Return JWT Signature
B->W: POST /securesession/startsession (JWT, tokens)
W->W: Validate JWT, <br/>(w/ match to tokens)
W->B: AuthCookie
Note over W, A: Refresh DBSC...
B->W: GET /securesession/refresh (sessionID)
W->B: Challenge
B->P: Request Sign JWT (sessionID)
P->B: Return JWT Signature
B->W: GET /securesession/refresh (JWT)
W->W: Validate JWT (w/public key on file)
W->B: AuthCookie
````````

## Opened topics
-  Should introduce a new entity "Device registration client" and Local key helper and device registration client should be developped by the same vendor?
- Can we consider Local Key helper to be part of device registration client or not?
- Any IDP can call any local Local Key helper?
- How local key helper is deployed?
- Special/trusted by default Local Key helpers (Part of OS or Browser).
- Protocol between IdP and LocalKey helper, if they belong to different vendors (Note: we need to solve clock-skew problem between IdP and Attestation server, probably embed nonce in the request)
- Format of the public key cert/binding statement, and claims it contains.

## Closed topics
- Existance of the local key helper.
- Local key helper can be a 3P software.
- PublicKey cert/binding statement can be either short-lived (IdP and Local Key helper belong to different vendors) or long-lived(IdP and Local Key helper belong to the same vendor).
- The protocol between LocalKey helper and Attestation service doesn't need to be documented as part of public spec, it can stay internal.
- Attestation service may not exist.

## Meeting notes

### 4/30/2024

_\<adding notes here\>_

### 4/23/2024

We discussed properties of the PublicKey Cert/Binding statement from the step 13 of the flow diagram above.

It was demonstrated that in the scenario where a Contoso's IdP calls Fabrikam's Local Key Helper, this artifact should be a short lived, otherwise it is possible to take the PublicKey cert and the binding key from a malicious device and be able to bind the auth cookie to the malicious binding key.

It was also demonstrated that if both IdP and Local Key helper belong to the same vendor, then public key cert/binding statement can be long-lived. As proof of possession of the device can be done during every authentication. After the device auth has happened the IdP can use long lived binding statement/public key cert to establish the fact that binding key belong to the same device.

We concluded that in some scenarios 
 - PublicKey cert/binding statement can be short-lived
 - PublicKey cert/binding statement can be long-lived.

We also agreed that format of that the public key cert/binding statement should be public, but may be private, if IdP and Local key helper belongs to the same vendor.

We need to define format of the public key cert/binding statement, and claims it contains.

For scenarios where PublicKey cert/binding statement is short-lived, we must solve a problem of clock-skew of between 2 different servers IdP and Attestation servers. For that purpose IdP can pass nonce that will be reflected inside PublicKey cert/binding statement. IdP will be able to validate nonce to ensure public key cert/binding statement is freshly issued.

We agreed that the protocol between LocalKey helper and Attestation service doesn't need to be documented as part of public spec. Attestation service may not exists.

On the meating we discussed Local Key helper. We agreed that Local Key helper can be a 3P software (not developed by browser or OS). During the meeting we discussed that Local Key helper and Attestation service can be developped by the same vendor. While it is true, after the meeting I came to conclusion that Attestation service may not exists. The source of truth for this scheme is device, and I believe we should introduce a new entity "Device registration client" and Local key helper and device registration client should be developped by the same vendor.

On the meeeting we agreed that Local Key Helper:
- can be owned by MDM client (3P MDM providers)
- can be owned by Device Registration client (MS on iOS, Android)
- can be owned by OS (MS on Windows, MAC)
- can be owned by Browser (Google Chrome)
 
Altentively we can think as Local Key helper is part of the device registation client. The device registartion client:
- can be owned by MDM client (3P MDM providers)
- can be owned by 3P vendor (MS on iOS, Android)
- can be owned by OS (MS on Windows, MAC)
- can be owned by Browser (Google Chrome)
