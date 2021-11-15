# General Notes From Chapter One

<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2000-37-04.png" width="500" height="300">

### Attacks Threatening Confidentiality
* **Snooping** refers to unauthorized access to or interception of data.
* **Traffic analysis** refers to obtaining some other type of information by monitoring online traffic.

### Attacks Threatening Integrity
* **Modification** means that the attacker intercepts the message and changes it.
* **Masquerading** or **spoofing** happens when the attacker impersonates somebody else.
* **Replaying** means the attacker obtains a copy of a message sent by a user and later tries to replay it. 
* **Repudiation** means that sender of the message might later deny that she has sent the message; the receiver of the message might later deny that he has received the message.

### Attacks Threatening Availability
* **Denial of service (DoS)** is a very common attack. It may slow down or totally interrupt the service of a system.
    * A sever class of this attack is the Distributed Denial of service (DDoS) attack. In this class very large number (thousands or even millions) of attacking machines are coordinated and synchronized to attack a victim system simultaneously. 

### Active Vs. Passive Attacks
In Active attack, an attacker tries to modify the content of the messages. Whereas in Passive attack, an attacker observes the messages, copy them and may use them for malicious purposes.

* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2000-53-32.png" width="550" height="300">
* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2000-54-44.png" width="550" height="300">

### Security Services and Mechanisms
for every type of attacks there a service to prevent this attack (service is the concept), and for every service we may have one or more mechanisms to implement this service.
Mechanisms is group of processes or operations.
* **ITU-T** provides some security services and some mechanisms to implement those services. 
* Security services and mechanisms are closely related because a mechanism or combination of mechanisms are used to provide a service.

### Security Services
(As recommended in the **X.800** Protocol of the **ITU-T*)
1. **Authentication** - assurance that communicating entity is the one claimed
	1. **peer-entity** authentication 
	2. **data origin** authentication
2. **Access Control** - prevention of the unauthorized use of a resource
3. **Access Control** - prevention of the unauthorized use of a resource
4. **Data Confidentiality** –protection of data from unauthorized disclosure
5. **Data Integrity** - assurance that data received is as sent by an authorized entity
6. **Non-Repudiation** - protection against denial by one of the parties in a communication
7. **Availability** – resource accessible and usable
8. alternative definition for Security Service
https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2001-14-36.png

### Security Mechanisms
1. **Encryption** The cryptographic transformation of data to produce ciphertext.
2. **Digital signatures** Data appended to, or a cryptographic transformation of a data unit that allows a recipient of the data unit to prove the source and integrity of the data unit and protect against forgery e.g. by the recipient.
3. **Access controls** Access control mechanisms may be applied at either end of a communications association and/or at any intermediate point. Access controls involved at the origin or any intermediate point are used to determine whether the sender is authorized to communicate with the recipient and/or to use the required communications resources.
4. **Data integrity** Two aspects of data integrity are:
	* the integrity of a single data unit or field.
	* and the integrity of a stream of data units or fields.
5. **Authentication exchange** A mechanism intended to ensure the identity of an entity by means of information exchange.
6. **Traffic padding** The generation of spurious instances of communication, spurious data units and/or spurious data within data units.
7. **Routing control** The application of rules during the process of routing to chose or avoid specific networks, links or relays.
8. **Notarization** The registration of data with a trusted third party that allows the later assurance of the accuracy of its characteristics such as content, origin, time and delivery.

#### Example for relation between Services and Mechanisms
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2001-25-25.png" width="550" height="300">
