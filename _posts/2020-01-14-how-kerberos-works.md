---
title: 'The Three-Headed Hound of the Underworld (Kerberos)'
date: 2020-01-14
permalink: /posts/2020/01/how-kerberos-works/
tags:
  - kerberos
  - security
  - authentication
---

Kerberos is simply a "ticket-based" authentication protocol. It enhances the security approach used by password-based authentication protocol. Since there might be a possibility for tappers to take over the password, Kerberos mitigates this by leveraging a ticket (how it is generated is explained below) that ideally should only be known by the client and the service.

When a client requests access to a service, it has to communicate with three parties. Each communication involves the use of "ticket". Those parties are:
<ul>
<li>the Authentication server</li>
<li>the Ticket Granting Server</li>
<li>the service it want to connect to</li>
</ul>

Both the <I>Authentication Server</I> and the <I>Ticket Granting Server</I> live within the Key Distribution Centre (KDC). KDC is simply a third-party used to authenticate the access request from the client to the service.

<h2>The Client & the Authentication Server</h2>

<b>Step 1</b>. This is the first authentication step. When the user does the login or executes `kinit USERNAME` command, the client sends out an information to retrieve the <b>Ticket Granting Ticket</b> (TGT). Such an information consists of the followings:

<ul>
<li>The Client ID</li>
<li>The ID of the Ticket Granting Server</li>
<li>The Client IP address</li>
<li>Validity period of the TGT</li>
</ul>

<b>Step 2</b>. The <I>Authentication Server</I> performs a check to see whether the <I>Client ID</I> exists in the KDC database.

<b>Step 3</b>. If the client exists in the KDC database, then the <I>Authentication Server</I> will <b>randomly</b> generate a key called `TGS SESSION KEY`. This key will be used for communication between the client and the <I>Ticket Granting Server</I>.

<b>Step 4</b>. The <I>Authentication Server</I> sends back two messages to the client.

The first message is the TGT which consists of the followings:

<ul>
<li>The Client ID</li>
<li>The TGS ID</li>
<li>Current Timestamp</li>
<li>The Client IP address</li>
<li>Validity period of the TGT</li>
<li><b>TGS SESSION KEY</b></li>
</ul>

The above message (TGT) is encrypted using the `TGS SECRET KEY`.

Meanwhile, the second message (let’s call it <I>Ticket Granting Server Session Key</i>) consists of the followings:

<ul>
<li>TGS ID</li>
<li>Current timestamp</li>
<li>Validity period of the TGT</li>
<li><b>TGS SESSION KEY</b></li>
</ul>

The above message is encrypted using the `CLIENT SECRET KEY`. This client secret key is password plus a salt that are hashed using a certain algorithm that was selected during the Kerberos setup.

<b>Step 5</b>. Upon receiving back the two messages, the client can now decrypt the <I>Ticket Granting Server Session Key</I> using its `CLIENT SECRET KEY`. Doing so, the client should have retrieve the `TGS SESSION KEY`.

The other message, namely TGT, can't be decrypted since the client does not know the `TGS SECRET KEY`.

<h2>The Client & the Ticket Granting Server</h2>

At this point, the client already possess the `TGS SESSION KEY` and TGT message (can not be decrypted).

<b>Step 1</b>. The client sends out two messages along with the TGT to the <I>Ticket Granting Server</I>.

The first message is called the <I>Authenticator</I> and is encrypted with the `TGS SESSION KEY`. It consists of the followings:

<ul>
<li>The Client ID</li>
<li>Current timestamp</li>
</ul>

Meanwhile, the other message is unencrypted and consists of the followings:

<ul>
<li>The ID of the requested service</li>
<li>Validity period of the ticket for the requested service</li>
</ul>

<b>Step 2</b>. The <I>Ticket Granting Server</I> will then check whether the requested service exists in the KDC database.

If it exists, then the TGT is decrypted using the `TGS SECRET KEY`. The `TGS SESSION KEY` stored within the TGT is used to decrypt the <I>Authenticator</I>.

<b>Step 3</b>. The TGS does the followings upon decrypting the TGT and the <I>Authenticator</I>.

<ul>
<li>Compare the Client ID in the <I>Authenticator</I> to the one in the TGT</li>
<li>Compare the timestamp in the <I>Authenticator</I> to the one in the TGT (the tolerance of difference can be configured)</li>
<li>Check whether the TGT is already expired (based on the Validity period of the TGT)</li>
<li>Check whether the <I>Authenticator</I> does not exist in the TGS's cache</li>
<li>Compare the client’s IP address to the one in the TGT</li>
</ul>

<b>Step 4</b>. If the previous checks returns no error, the TGS sends back two messages to the client.

The first one is the <b>service ticket</b> which consists of the followings:

<ul>
<li>the Client ID</li>
<li>the ID of the requested service</li>
<li>the Client’s IP address</li>
<li>Current timestamp</li>
<li>Validity period of the service ticket</li>
<li><b>REQUESTED SERVICE SESSION KEY</b></li>
</ul>

The above <b>service ticket</b> is encrypted with the `REQUESTED SERVICE SECRET KEY`.

Meanwhile, the other one (call it <I>SERVICE SESSION KEY</I>) consists of the followings:

<ul>
<li>the ID of the requested service</li>
<li>Current timestamp</li>
<li>Validity period of the service ticket</li>
<li><b>REQUESTED SERVICE SESSION KEY</b></li>
</ul>

The above message is encrypted with the `TGS SESSION KEY`.

<b>Step 5</b>. Upon receiving both messages, the client can decrypt the <I>SERVICE SESSION KEY</I> with its `TGS SESSION KEY`. Doing so, the client should have retrieved the `REQUESTED SERVICE SESSION KEY`.

<h2>The Client & the Requested Service</h2>

<b>Step 1</b>. Similar to the previous interaction, the client sends out two messages to the requested service.

The first one is the <I>Authenticator</I> that consists of:

<ul>
<li>The Client ID</li>
<li>Current timestamp</li>
</ul>

The above message is encrypted with the `REQUESTED SERVICE SESSION KEY`.

Meanwhile, the other one is the <b>service ticket</b> retrieved previously.

<b>Step 2</b>. Since the requested service has the `REQUESTED SERVICE SECRET KEY`, it can fetch the `REQUESTED SERVICE SESSION KEY` from the encrypted <b>secret key</b>.

<b>Step 3</b>. The fetched `REQUESTED SERVICE SESSION KEY` is then used to open the <I>Authenticator</I>.

<b>Step 4</b>. The requested service performs the followings:

<ul>
<li>Compare the client ID in the <I>Authenticator</I> to the one in the <b>service ticket</b></li>
<li>Compare the timestamp in the <I>Authenticator</I> to the one in the <b>service ticket</b> (the tolerance of difference can be configured)</li>
<li>Check whether the <b>service ticket</b> is already expired (based on the validity period)</li>
<li>Check whether the <I>Authenticator</I> does not exist in the service's cache</li>
<li>Compare the client’s IP address to the one in the <b>service ticket</b></li>
</ul>

<b>Step 5</b>. If the previous checks returns no error, the requested service sends back a message to the client. The message consists of the followings:

<ul>
<li>the Client ID</li>
<li>Current timestamp</li>
</ul>

The above message is encrypted with the `REQUESTED SERVICE SESSION KEY`.

<b>Step 6</b>. The client receives the message and decrypts it with the `REQUESTED SERVICE SESSION KEY`.

<h2>Finally</h2>

The client has now been authenticated to use the service. Next time it requests access to the service, it can just use the cached encrypted <b>service ticket</b> (as long as the ticket is still valid).
