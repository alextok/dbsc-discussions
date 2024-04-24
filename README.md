# DBSC discussions

## DBSC with attested Security Module

```mermaid
sequenceDiagram
%%{ init: { 'sequence': { 'noteAlign': 'left'} } }%%
autonumber
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
B->>I: PubKey Cert
 
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

## Open questions

Opened question here

## Meeting notes

Meeting notes here
