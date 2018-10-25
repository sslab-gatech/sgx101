---
layout: page
title: Remote Attestation Example
---

Remote Attestation code example is available [here](https://github.com/sangfansh)

Original unmodified version is available [here](https://github.com/svartkanin/linux-sgx-remoteattestation)

IAS Service Guide is available [here](https://software.intel.com/sites/default/files/managed/7e/3b/ias-api-spec.pdf)

Before running the code, some settings have to be set in the GeneralSettings.h file:
----

* The application port and IP

* A server certificate and private key are required for the SSL communication between the SP and the Application (which can be self-signed)

```sh
openssl req -x509 -nodes -newkey rsa:4096 -keyout server.key -out server.crt -days 365
```

* The SPID provided by Intel when registering for the developer account

* The certificate sent to Intel when registering for the developer account

* IAS Rest API url (should stay the same)


To be able to run the above code some external libraries are needed:
----

* Google Protocol Buffers (should already be installed with the SGX SDK package) otherwise install 

```sh
libprotobuf-dev libprotobuf-c0-dev protobuf-compiler
```

**All other required libraries** can be installed with the following command:

```sh
    sudo apt-get install libboost-thread-dev libboost-system-dev curl libcurl4-openssl-dev libssl-dev liblog4cpp5-dev libjsoncpp-dev
```

After the installation of those dependencies, the code can be compiled with the following commands:

```sh
    cd ServiceProvider
    make
    cd ../Application
    make SGX_MODE=HW SGX_PRERELEASE=1
```

The sample application has two parts: **ServiceProvider** and the **Application**. The Application needs to prove to the ServiceProvider that it is running on a claimed trusted SGX enclave so that the Service Provider can proceed to provision secret data. The messages exchanged during the remote attestation process are serialized using [Google Protobuf](https://github.com/google/protobuf).

First let’s examine the **Application**. The entry point of the Application is inside isv_app.cpp (ISV stands for Individual Software Vendor). It starts by initiating the `MessageHandler` that handles the messages to be exchanged during remote attestation.

<script src="https://gist.github.com/sangfansh/e884ede5fd80082eeedc70f22e8f2e4c.js"></script>

MessageHandler (a wrapper of all of the networking functionalities):
----

The MessageHandler has a protected enclave that handles all the secrets, as well as the generation and processing of cryptographic messages. We can see that MessageHandler has several message generation and handling functions. Specifically they are functions in forms of `generateMsgx()` and `handleMsgx()`. Google Protobuf is used by those functions to serialize the messages to be exchanged via network transportation.

The MessageHandler also has a `NetworkManagerServer` object. The NetworkManagerServer object enables the Application to act as a server to initialize connection and binds itself to a client via SSL (implementation detail in server.h, using object Server* server). It also inherits `NetworkManager` class in order to serialize and send messages.

<script src="https://gist.github.com/sangfansh/9e159a13dc47ed196facd14a9a2e148a.js"></script>

`msg -> init():`
<script src="https://gist.github.com/sangfansh/788651555f6e7e43ceb8dd346764da4f.js"></script>
<script src="https://gist.github.com/sangfansh/13a82a3b4e7b5c51bfe67b7bf97cd542.js"></script>
<script src="https://gist.github.com/sangfansh/60737cb869a669cc45f1b1960347c220.js"></script>

When the `MessageHandler` msg is initialized using `init()`, the `NetworkManagerServer` object inside is also initialized. It causes the initialization of `Server` object, which sets up the SSL io_service socket and the selected port. Then a function `incomingHandler()` is connected to the `NetworkManagerServer` as the `CallbackHandler`. This function is responsible for generating all the message replies according to the type of the message that it receives.

As mentioned above, `incomingHandler(string, int)` handles all of the incoming messages and generates corresponding replies. Let’s briefly examine this handler (line 395 at `MessageHandler.cpp`).

<script src="https://gist.github.com/sangfansh/bda856ff5728ab7fff00172e6b98d456.js"></script>

There are four cases:
----

* RA_VERIFICATION

* RA_MSG0

* RA_MSG2

* RA_ATT_RESULT

Each case is one type of the messages that are exchanged in time order during remote attestation.

Upon here, the ISV application’s MessageHandler has finished initialization.

`msg -> start():`
<script src="https://gist.github.com/sangfansh/ec0b603dc914421b58c4454f3e3ef126.js"></script>
<script src="https://gist.github.com/sangfansh/4004e50e2bc242b5653fd6ea7dfad4a3.js"></script>

Function `start()` calls NetworkManager’s `startService()` function, which calls Server’s `start_accept()` to start SSL service. Upton here, the ISV has started running and is ready for any incoming traffic.

Now let’s examine `ServiceProvider`’s structure and initialization process.
----

`isv_app.cpp` inside `ServiceProvider`:
<script src="https://gist.github.com/sangfansh/2dbd162c0dc46c51d0f046f0fbc8b7ed.js"></script>

It’s the ServiceProvider Application itself. Similar to the client’s application, it has its own message handler `VerificationManager` since it acts as the verifier in the remote attestation process.

Inside the VerificationManager, it has a `NetworkManagerClient` which also inherits `NetworkManager` and is responsible for the SSL connection as well as serializing and sending messages.

<script src="https://gist.github.com/sangfansh/d1c1443a08f7e7128fff3203379a4231.js"></script>

In addition, it has a `WebService` object to performs the verification phase with **Intel Attestation Service (IAS)** using the `QUOTE` sent from the client enclave. In one word, a `ServicePrivider` also acts as a wrapper of all the IAS requests and message processing, as well as any encryption key derivation, using `WebService`.

<script src="https://gist.github.com/sangfansh/dfa4b517e17cbc9030d0f09326b28698.js"></script>

`vm -> init()`:
<script src="https://gist.github.com/sangfansh/f935847f93926df81b4f4b7c1bace686.js"></script>

First, dynamically allocate a new `ServiceProvider` to ensure freshness of secret. Then the function initializes `NetworkManagerClient` that will connect to the Server (the Application) using SSL. Finally, a `CallBackHandler` similar to the one of the Application is set up. The handler itself is the function `incomingHandler(string, int)` inside `VerificationManager` that will handle incoming messages coming from the Application during remote attestation (line 132 at `VerificationManager.cpp`).

<script src="https://gist.github.com/sangfansh/eadaa5c6e6d10fe43e3a562c8efd7f4f.js"></script>

There are also four cases of handling messages:
----

* RA_MSG0

* RA_MSG1

* RA_MSG3

* RA_APP_ATT_OK

Notice that at the end of the handler, it initializes the message stream with a `RA_VERIFICATION` type string and the actual `REQUEST`. By doing so right after SSL handshake, the ServiceProvider can immediately start the RA process by forwarding the remote attestation request to the Application.

`vm -> start()`:
<script src="https://gist.github.com/sangfansh/dbee6d345177c73d1ca2c1914aa5da3c.js"></script>
<script src="https://gist.github.com/sangfansh/6969fb1cf21cdcd19e496a81a909947a.js"></script>
<script src="https://gist.github.com/sangfansh/d1728c9fb3567734b1afd2d4a65ef457.js"></script>

`NetworkManagerClient` starts the service by connecting the SSL client to server. Then the client starts the SSL handshake process with server. We can see that the `Client::handle_handshake()` function triggers the remote attestation by sending out the attestation request mentioned above if handshake is successful.

Upon here, ServiceProvider and Application are connected and remote attestation process has started.

----

Now let’s examine both `incomingHandler(string, int)` in `MessageHandler.cpp` and `VerificationManager.cpp` to reflect the remote attestation process between SP and ISV described in the tutorial.

Summary of function calls:
----
![ra](/sgx101/assets/pics/ra_summary.png)

(Please refer to `Messages.proto` for message structure details.)

(Please refer to the two `incomingHandler`'s attached below for actual implementation.)

![ra](/sgx101/assets/pics/ra_flow.png)
<script src="https://gist.github.com/sangfansh/bda856ff5728ab7fff00172e6b98d456.js"></script>
<script src="https://gist.github.com/sangfansh/eadaa5c6e6d10fe43e3a562c8efd7f4f.js"></script>