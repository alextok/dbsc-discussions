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
B->>I: Load sign-in (follow the 302)
 
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
W->>B: StartSession (challenge, tokens)
B->>P: Request Sign JWT (uri, challenge, tokens?)
P->>B: Return JWT Signature
B->>W: POST /securesession/startsession (JWT, tokens)
W->>W: Validate JWT, <br/>(w/ match to tokens)
W->>B: AuthCookie

Note over W, A: Refresh DBSC...
B->>W: GET /securesession/refresh (sessionID)
W->>B: Challenge
B->>P: Request Sign JWT (sessionID)
P->>B: Return JWT Signature
B->>W: GET /securesession/refresh (JWT)
W->>W: Validate JWT (w/public key on file)
W->>B: AuthCookie
````````

## Opened topics
1.  Should introduce a new entity "Device registration client" and Local key helper should be considered as a part of the device registration client or not?
1. Any IDP can call any local Local Key helper?
1. How local key helper is deployed?
1. Special/trusted by default Local Key helpers (Part of OS or Browser).
1. Protocol between IdP and LocalKey helper, if they belong to different vendors (Note: we need to solve clock-skew problem between IdP and Attestation server, probably embed nonce in the request)
1. Format of the public key cert/binding statement, and claims it contains.

    1. We can have multiple public key cert/binding statements for one key, when IdP and LocalKey helper is owned by one vendor, how we include it?
    
    1. For the sign-in ceremony, after key generation happens, we should discuss how exactly we will deliver pubblic key cert/binding statements and whether it should be a header format. For example, step "Sec-Session-GenerateKey ..., RP , HelperId" is included in a header in a 302 response from IDP, does browser attach Pubkey/Attestation information as a header before executing on a 302?

1. Shold we remove RP from this leg
    ```mermaid
    sequenceDiagram
    %%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
    autonumber 7
    participant B as B Browser
    
    B->>B: Evaluate policy for (RP, IdP, HelperId)<br/> - RP can be '*' in policies
    ````````
1. We need to clarify params for this call:
    ```mermaid
    sequenceDiagram
    %%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
    autonumber 8
    participant B as B Browser
    participant P as P Local Key Helper
    
    B->>P: Pre-gen key and attest (params?)
    
    ````````
1. We planned to use KeyContainerId here:
    ```mermaid
    sequenceDiagram
    %%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
    autonumber 13
    participant W as W Relying Party
    participant I as I IdP
    participant B as B Browser
    
    B->>I: PubKey Cert/Binding statement, _KeyContainerId_
 
    I->>B: Sign in ceremony
    B->>I: Sign done
 

    I->>B: Auth tokens, _KeyContainerId_
    B->>W: Auth tokens, _KeyContainerId_
    
    Note over W, B: Initiate DBSC ...
    W->>B: StartSession (challenge, tokens, _KeyContainerId_)
    ````````
1. Step 18 above, should it go LocalKey helper for signature? If yes, how does step that initiates DBSC session know that it needs to go to the local key helper? Should it be by IDP URL or helperID? 

1. Key conteainer Id can be exposed to RP, we need to have always random, to satisfy privacy concerns, prevent tracking.

1. Do we need this step, if we planned to use KeyContainerId?
    ```mermaid
    sequenceDiagram
    %%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
    autonumber 12
    participant B as B Browser
    
    B->>B: Remember this key is for RP (and maybe path)
    ````````
1. Not sure if step 21 needs to be a POST, if we imagine StartSession from RP to be a 302 redirect, then it probably shouldn't be a POST
    ```mermaid
    sequenceDiagram
    %%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
    autonumber 21
    participant W as W Relying Party
    participant B as B Browser
    
    B->>W: POST /securesession/startsession (JWT, tokens)

    ````````

1. We should discuss provisioning flows too in more details

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

```mermaid
sequenceDiagram
%%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
autonumber 13
participant I as I IdP
participant B as B Browser
 
B->>I: PubKey Cert/Binding statement
 
````````

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
