# Specification on Data Processing (WIP)

This section objective is to describe the functionality of a node, as well to determine how the Coordicide modules work and interact with each other. 

# Terminology 

The main elements defined here are

1. Data Set (**DATA**): Data elements that store information.
2. Communication Port (**PORT**): Communication ports used to exchange data among nodes or with external actors.
3. Logic Process (**PROCESS**): A set of actions the node take interacting with the other elements. 



We also consider the possible functions performed by a PROCESS:

* CALL(PROCESS A): Running **PROCESS** activates  **PROCESS** A.
* WRITE($x$, **DATA** A): Running **PROCESS** writes entry $x$ in the storage **DATA** A.
* DELETE($x$, **DATA** A): Running **PROCESS** deletes entry $x$ in the storage **DATA** A.
* READ(*info* ,**DATA** A): Return from the storage **DATA** A the data related to input *info*.



# PROCESS Anatomy

Now we show how we will present the different processes, showing their different interactions. The main elements of our representation are as follows:
![](https://i.imgur.com/WuHlwWl.png)

As an example, the Message Tangle Solidification **PROCESS** it is represented by
![](https://i.imgur.com/YSjdBOP.png)

For each process we will give a resumed description of its functionality and link to the appropriate section where it will be described in details.

# List of Ports and Data Sets

The following ports will be used in the processes:

* **Autopeering Request Port:** The port used by nodes to send and receivresponsible for the packets relatneeded to nodes to use the Autopeering mechanism and decide who to connect to in the Nodes Network. 
* **Gossip Port:** The port by nodesresponsible for the exchange of messages and notifications between neighbor nodes.
* **FPC Voting Port:** The pPort nodes will use to send and receive FPC votes.
* **Message Creation Port:** The port where Users will be able to send trarnsactions to nodes to create their messages. 
* **Finality Inquiry Port:** TheA port where both Users and Nodes will be able to check the status of their transaction. 

The following message data sets will be used in the processes:
* **Message Iinbox:** A data set where non-duplicated messages will stay until processeds by the node and added to the Tangle. 
* **Solidification Buffer:** A data set where messages that were not able to quick-solidify will be kept until the missing messages arrive.
* **Message Tangle:** The mains data set where all the non-snapshotted messages are kept. 
* **Rate Manager Outbox:** A data set where messages that were processed are kept until they are gossiped further. 

The following other data sets will be used in the processes:
* **Node List:** The list of nodes known by the current Node. 
* **Nodes Informations Database:** A database of quantities related to the nodes that are used for the protocol, such as "Mana Value", "Effective Mana", "Recent transactions arrivals", etc. 
* **Ledger State:** A list with the current balance on each address and node. 


# Main Message Processing Diagram 

The Main Message processor constitutes of 87 processes that constitutes of all actions the node do with a message from the moment it is received until it is included on its Tangle and further gossiped. The 8 processes are: "Duplicate Detector", "Node Signature Validator", "Timestamp Opinion and Filter", "Proof of Work Check", "Solidifier", "Rate Manager" and "Gossip Scheduler".

Ignoring the possible data sets and ports used, the processes are ordered as:

![](https://i.imgur.com/xFjNlFd.png)


1. **Duplicate Detector**
The Duplicate Ddetector is triggered each time a message arrives in the gossip port. It will look if there if the exact same message has already arrived before, being either on the Message Tangle or in the Message Inbox. The message will only be added to the Message Inbox in  case it is not found in both data sets.
![](https://i.imgur.com/5Ke3jId.png)

2. **Node Signature Validator**
The Node Signature Validator will timely get messages from the Message Inbox and check ifn its signaturrute is valid. Messages that fail this verification are deleted from the Message Inbox.
![](https://i.imgur.com/mOrfhqz.png)

3. **Timestamp Opinion and Filter**
This process will compare the arriival time of the message with its declared timestamp in order to define the proper Level of Knowledge of the timestamp of the message. Messages that receive a rating of "Level 3 Bad Timestamp" will be deleted from the Message Inbox. 
![](https://i.imgur.com/lVa7H5L.png)

4. **Proof of Work Check**
This Process will check if the required proof of work was done by the node, if it fails then the message is deleted from the Message Inbox.
![](https://i.imgur.com/sn7UJFD.png)

5. **Solidifier**
This process will check if all messages in the past cone of the transaction are included in either the Message Inbox or in the Message Tangle. In case it is not, then the process will include the message in the Solidification Buffer until the missing messages arrive during a waiting time. If the missing messages do not arrive, the last action will be to demand from neighbor nodes the missing messages. If the last step ends in a failure, the message is dropped from the Message Inbox and proper action against the nodes is taken. 
![](https://i.imgur.com/YSjdBOP.png)\

6. **Rate Manager**
This process will check if the message is within the issuer node allowed rate, and will give a feedback message in case it is violating such limits. It may also trigger a "Sever Connection with Node" action in case of excessive number of violations.

8. **Gossip Scheduler**
The Gossip Scheduler will take messages in the Processor Outbox following priority parameters, further gossip it to the processing node neighbours and write it on the Message Tangle. 



# Value Message Processing Diagram 

The processes of the Value Tangle are 

1. **Duplicate Detector**
Theis Duplicator Detector will check if the transaction is already included in the Value Tangle. This will also define the Epoch of the transaction. If two duplicate transactions are sent in the same Epoch, they are considered the same, while under different Eepochs one is considered an reattachment. 

2. **Validator**
The Validator will check the UTXO inputs to see if the funds being transferred exists and will also remove value messages with no funds bveing moved. 


3. **Conflict Check**
Here we will set the status of the transaction according to the existence of conflicts with the data you already have on the Tangle. If no conflict is found then the transaction is flagged "non-eligible", what means it is still not ready for tip selection but has no other problems. In case a conflict is found and the conflicting transaction is "non-eligible", then both the transactions will be flagged "Vote Pending", and after FPC they will be categorized as either "Eligible" or "Disliked". Finally if a conflict is found but the conflicting transaction is already "Eligible", then the most recent one will be flagged as "Disliked".


# Independent Processes

The Processes that are not part of the main message processing are

* **Mana Updater**
The Mana Updater is called when a messages enters Level 3 Finality, updating the Mana accordingly in the Nodes Information Database. 
* **Tip Selector**
Following the Weighted Tip Selection described in XXX, the Tip Selector when called will return two messages chosen independently from the Eligible list (potentially the same), where the selection follows the mentioned algorithm. 
* **Snapshotter**
The Snappshotter will erase all appropriatte information from the Message Tangle of old enough transactions that are considered confirmed and Level 3 Final. This process may be automated. 
* **Payload Parser**
The Payload Parser is the process that identifiesy the payload the message carries, see if the node has the capability to process it and call the approppriate processes for it. 
* **Value Tip Selector**
The Value Tip Selector works similarly to the Tip Selection, but it changes the eligibility for a value message to be a tip since it needs to be solid.
* **Finalization Tracker**
The Finalization Tracker update the Finality level of transaction according to the criteria defined for each level. 



<!--stackedit_data:
eyJkaXNjdXNzaW9ucyI6eyJ1WnFsUlZyU3ViSGYxYzA4Ijp7In
RleHQiOiJUaGUgcG9ydCByZXNwb25zaWJsZSBmb3IgdGhlIHBh
Y2tldHMgbmVlZGVkIHRvIG5vZGVzIHRvIHVzZSB0aGUgQXV0b3
BlZXJpbmcgbWVj4oCmIiwic3RhcnQiOjE1MzUsImVuZCI6MTY1
MX0sInVIcEs0akRGZG1nUVVYUXgiOnsidGV4dCI6Im5vbi1zbm
Fwc2hvdHRlZCIsInN0YXJ0IjoyNTMyLCJlbmQiOjI1NDd9LCJz
N1BuQmQ0aVpvOFByaEhTIjp7InRleHQiOiJOb2RlcyBJbmZvcm
1hdGlvbiBEYXRhYmFzZSIsInN0YXJ0IjoyODEzLCJlbmQiOjI4
NDB9LCJIMExjTTlYVVdXbVMwd1QwIjp7InRleHQiOiJUaGUgTW
FpbiBNZXNzYWdlIHByb2Nlc3NvciBjb25zdGl0dXRlcyBvZiA3
IHByb2Nlc3NlcyB0aGF0IGNvbnN0aXR1dGVzIG9mIGFsbCBh4o
CmIiwic3RhcnQiOjMxMTcsImVuZCI6MzMxN30sImc2WkRTQjF4
dGY1UVRDMWciOnsidGV4dCI6IlRpbWVzdGFtcCBPcGluaW9uIG
FuZCBGaWx0ZXIiLCJzdGFydCI6MzM5MSwiZW5kIjozNDE5fSwi
ZjR0SDh6MUcxMmFmdE8weCI6eyJ0ZXh0IjoiU29saWRpZmllci
IsInN0YXJ0IjozNDQ2LCJlbmQiOjM0NTZ9LCJKNXJWbTI0VW9J
ZmRBY0plIjp7InRleHQiOiIhW10oaHR0cHM6Ly9pLmltZ3VyLm
NvbS81S2UzaklkLnBuZykiLCJzdGFydCI6Mzk2MCwiZW5kIjoz
OTk2fSwiQ1czVTQ4S1daRVpkM04zVSI6eyJ0ZXh0IjoiZnJvbS
B0aGUgTWVzc2FnZSBJbmJvIiwic3RhcnQiOjQxOTcsImVuZCI6
NDIxOH0sIk9idnNOak8waWFuY3psaWUiOnsidGV4dCI6Ikxldm
VsIG9mIEtub3dsZWRnZSIsInN0YXJ0Ijo0NDEyLCJlbmQiOjQ0
MzB9LCJtOUptUHlwalRHRGtKN05nIjp7InRleHQiOiIhW10oaH
R0cHM6Ly9pLmltZ3VyLmNvbS9sVmE3SDVMLnBuZyIsInN0YXJ0
Ijo0NTYzLCJlbmQiOjQ1OTh9LCJqd0ExT3lKN3VnVm1lejR5Ij
p7InRleHQiOiIhW10oaHR0cHM6Ly9pLmltZ3VyLmNvbS9zbjdV
SkZELnBuZykiLCJzdGFydCI6NDc2OCwiZW5kIjo0ODA0fSwiYj
RhM2tJeVpsWlJPZFZLRiI6eyJ0ZXh0IjoiZnJvbSB0aGUgTWVz
c2FnZSBJbmJveCBhbmQgcHJvcGVyIGFjdGlvbiBhZ2FpbnN0IH
RoZSBub2RlcyBpcyB0YWtlbiIsInN0YXJ0Ijo1Mjg4LCJlbmQi
OjUzNTV9LCJmQWtLa1Z2U3VTUWxwclNnIjp7InRleHQiOiJTZX
ZlciBDb25uZWN0aW9uIHdpdGggTm9kZVwiIGFjdGlvbiBpbiBj
YXNlIG9mIGV4Y2Vzc2l2ZSBudW1iZXIgb2YgdmlvbGF0aW9ucy
4iLCJzdGFydCI6NTU4OSwiZW5kIjo1NjY2fSwiNlNFQk1qZlpy
ZUNWSmFxZCI6eyJ0ZXh0IjoiKlJhdGUgTWFuYWdlcioiLCJzdG
FydCI6NTQwMSwiZW5kIjo1NDE1fSwiRzFPd1p1QjNidHg1cmZx
ViI6eyJ0ZXh0IjoiR29zc2lwIFNjaGVkdWxlciIsInN0YXJ0Ij
o1NjczLCJlbmQiOjU2ODl9LCJ0RklLdWk5eTBmNXJFcnZYIjp7
InRleHQiOiJUaGUgcHJvIiwic3RhcnQiOjU5MTYsImVuZCI6NT
kyM30sImM2RDlWNVU0VGJFeDU2TXYiOnsidGV4dCI6IkVwb2No
cyIsInN0YXJ0Ijo2MjQxLCJlbmQiOjYyNDh9LCJBV2o0VENiWX
c0STVITloxIjp7InRleHQiOiJhbmQgdGhlIGNvbmZsaWN0aW5n
IHRyYW5zYWN0aW9uIGlzIFwibm9uLWVsaWdpYmxlXCIiLCJzdG
FydCI6Njc4OSwiZW5kIjo2ODM4fSwiRVpsRHBQUk40ZTZIZ1dr
OSI6eyJ0ZXh0IjoiaGUgY29uZmxpY3RpbmciLCJzdGFydCI6Nz
AxMCwiZW5kIjo3MDI0fSwiVW84NENsdE0weVMxdmNMcSI6eyJ0
ZXh0IjoiaGUgUHJvY2Vzc2VzIHRoYXQgYXJlIG5vdCBwYXJ0IG
9mIHRoZSBtYWluIG1lc3NhZ2UgcHJvY2Vzc2luZyBhciIsInN0
YXJ0Ijo3MTQ0LCJlbmQiOjcyMDh9LCJzZGRRSTZIbjFYbkIxT0
RXIjp7InRleHQiOiIqKlBheWxvYWQgUGFyc2VyKioiLCJzdGFy
dCI6NzgyMywiZW5kIjo3ODQxfX0sImNvbW1lbnRzIjp7IjBPYz
JZZmF1aXh2OFNDclAiOnsiZGlzY3Vzc2lvbklkIjoidVpxbFJW
clN1YkhmMWMwOCIsInN1YiI6ImdoOjUxMTEyNjE4IiwidGV4dC
I6InRoZSBwb3J0IGZvciBhdXRvcGVlcmluZyBwcm90b2NvbC9y
ZXF1ZXN0cz8gRG9uIHQgdW5kZXJzdGFuZCB3aGF0IHRoZSBkZW
Npc2lvbiBvbiAgd2hvIHRvIGNvbm5lY3QgaGFzIHRvIHRvIHdp
dGggdGhlIHBvcnQuIiwiY3JlYXRlZCI6MTU5NTQyMDk3NzU1NH
0sIjNjQUVXMmZwdHRVRnFvME4iOnsiZGlzY3Vzc2lvbklkIjoi
dVpxbFJWclN1YkhmMWMwOCIsInN1YiI6ImdoOjY4MjUwMzUwIi
widGV4dCI6IkkgdHJpZWQgdG8gZ2l2ZSBhIGJldHRlciB3b3Jk
aW5nLCB0aGUgb2xkIG9uZSB3YXMgYXdmdWwuIiwiY3JlYXRlZC
I6MTU5NTQ0NDExMTE0Mn0sInUyV1B2ME5aT1Q0alJPY2giOnsi
ZGlzY3Vzc2lvbklkIjoidVpxbFJWclN1YkhmMWMwOCIsInN1Yi
I6ImdoOjUxMTEyNjE4IiwidGV4dCI6InNvIGl0cyB0aGUgc2Ft
ZSBwb3J0IGZvciBzZW5kaW5nIGFuZCByZXF1ZXN0aW5nLiBJZi
B5ZXMgcGVyaGFwcyBqdXN0IGNhbGwgaXN0IEF1dG9wZWVyaW5n
IFBvcnQiLCJjcmVhdGVkIjoxNTk1NTAwNTg2MTIyfSwiaDZOdU
JESTRwTm42SmpEQyI6eyJkaXNjdXNzaW9uSWQiOiJ1SHBLNGpE
RmRtZ1FVWFF4Iiwic3ViIjoiZ2g6NTExMTI2MTgiLCJ0ZXh0Ij
oiZGVmaW5lIHNuYXBzaG90IHNvbWV3aGVyZSIsImNyZWF0ZWQi
OjE1OTU1MDA3MDI5NTV9LCJpVDZVTzRFVXRwNHVwYWdXIjp7Im
Rpc2N1c3Npb25JZCI6InM3UG5CZDRpWm84UHJoSFMiLCJzdWIi
OiJnaDo1MTExMjYxOCIsInRleHQiOiJqdXN0IFwiTm9kZSBEYX
RhYmFzZVwiPyIsImNyZWF0ZWQiOjE1OTU1MDA4MTA1MzV9LCJU
UUhsejBXUDdWWk9lMnBLIjp7ImRpc2N1c3Npb25JZCI6IkgwTG
NNOVhVV1dtUzB3VDAiLCJzdWIiOiJnaDo1MTExMjYxOCIsInRl
eHQiOiJUaGUgTWFpbiBNZXNzYWdlIHByb2Nlc3NvciBpcyBkaX
ZpZGVkIGludG8gNyBzdWItcHJvY2Vzc2VzOyBjb3ZlcmluZyBh
bGwgYWN0aW9ucyBhIG5vZGUgaGFzIHRvIHBlcmZvcm0gaW4gb3
JkZXIgdG8gc2VuZCBhIG1lc3NhZ2UgPyIsImNyZWF0ZWQiOjE1
OTU1MDA5NzkxNDJ9LCJkRkpMbmd0cmlTNmp3MWo4Ijp7ImRpc2
N1c3Npb25JZCI6IkgwTGNNOVhVV1dtUzB3VDAiLCJzdWIiOiJn
aDo1MTExMjYxOCIsInRleHQiOiI3LT44IiwiY3JlYXRlZCI6MT
U5NTUwMTAwNzM4M30sIkRKVUhRYmZnMFZMMzh6ZjkiOnsiZGlz
Y3Vzc2lvbklkIjoiZzZaRFNCMXh0ZjVRVEMxZyIsInN1YiI6Im
doOjUxMTEyNjE4IiwidGV4dCI6IlRpbWVzdGFtcCBjaGVjaz8i
LCJjcmVhdGVkIjoxNTk1NTAxMDMxMjg3fSwiWDdVZFVHaHR1QT
hUZk96ciI6eyJkaXNjdXNzaW9uSWQiOiJmNHRIOHoxRzEyYWZ0
TzB4Iiwic3ViIjoiZ2g6NTExMTI2MTgiLCJ0ZXh0Ijoib3IgTV
QgU29saWRpZmllcj8iLCJjcmVhdGVkIjoxNTk1NTAxMDU2NDI0
fSwiWmZJQms3dVYwN3lYRmh5ZCI6eyJkaXNjdXNzaW9uSWQiOi
JKNXJWbTI0VW9JZmRBY0plIiwic3ViIjoiZ2g6NTExMTI2MTgi
LCJ0ZXh0IjoiV2h5IERlbGV0ZSBpbiBNZXNzYWdlIEluYm94Py
IsImNyZWF0ZWQiOjE1OTU1MDEyMDY0NjB9LCI4a2Nhenh5OWpM
NzFXVURhIjp7ImRpc2N1c3Npb25JZCI6IkNXM1U0OEtXWkVaZD
NOM1UiLCJzdWIiOiJnaDo1MTExMjYxOCIsInRleHQiOiJhZGQg
XCJERUxFVEUgTWVzc2FnZSBJbmJveFwiIiwiY3JlYXRlZCI6MT
U5NTUwMTM4NDM2Nn0sIlFBODhKV1dVRklsbVRxMUgiOnsiZGlz
Y3Vzc2lvbklkIjoiQ1czVTQ4S1daRVpkM04zVSIsInN1YiI6Im
doOjUxMTEyNjE4IiwidGV4dCI6Ik9uY2UgYSBtZXNzYWdlIGlz
IGRlbGV0ZWQgZnJvbSBJbmJveCwgZ29zc2lwIG1pZ2h0IHB1dC
BpdCB0aGVyZSB5ZXQgYW5vdGhlciB0aW1lID8iLCJjcmVhdGVk
IjoxNTk1NTAxNDM2MDM2fSwiVlNXSlF6WXd6NzV4VmRoQSI6ey
JkaXNjdXNzaW9uSWQiOiJPYnZzTmpPMGlhbmN6bGllIiwic3Vi
IjoiZ2g6NTExMTI2MTgiLCJ0ZXh0IjoiaW50cm9kdWNlIGxpbm
sgdG8gRGVmaW5pdGlvbiIsImNyZWF0ZWQiOjE1OTU1MDE0NzQ4
NDl9LCJXNUc3WGNmTkJ2a0hQYVZ5Ijp7ImRpc2N1c3Npb25JZC
I6Im05Sm1QeXBqVEdEa0o3TmciLCJzdWIiOiJnaDo1MTExMjYx
OCIsInRleHQiOiJNaXNzaW5nIERFTEVURSBNZXNzYWdlIElOQk
9YLCIsImNyZWF0ZWQiOjE1OTU1MDE1NDYwMzd9LCI0eWVBbjhU
WVVSQnNwekR4Ijp7ImRpc2N1c3Npb25JZCI6Im05Sm1QeXBqVE
dEa0o3TmciLCJzdWIiOiJnaDo1MTExMjYxOCIsInRleHQiOiJB
bmQgUkVBRCIsImNyZWF0ZWQiOjE1OTU1MDE2NjE2MjN9LCJ1M1
dzVjJ0aURkQThuWU5FIjp7ImRpc2N1c3Npb25JZCI6Imp3QTFP
eUo3dWdWbWV6NHkiLCJzdWIiOiJnaDo1MTExMjYxOCIsInRleH
QiOiJXcm9uZyBEQiBvcGVyYXRpb247IFJFQUQvREVMRVRFIE1l
c3NhZ2UgSW5ib3giLCJjcmVhdGVkIjoxNTk1NTAxNzI5NjcwfS
wiMkFiT2FtU0c3OTdXQ0NybSI6eyJkaXNjdXNzaW9uSWQiOiJi
NGEza0l5WmxaUk9kVktGIiwic3ViIjoiZ2g6NTExMTI2MTgiLC
J0ZXh0IjoiYWRkIHRoZXNlIGFjdGlvbiB0byBmaWd1cmUiLCJj
cmVhdGVkIjoxNTk1NTAxODIxNTYwfSwiTEtuR0FZbjJMNkFBUU
ZZRiI6eyJkaXNjdXNzaW9uSWQiOiJmQWtLa1Z2U3VTUWxwclNn
Iiwic3ViIjoiZ2g6NTExMTI2MTgiLCJ0ZXh0IjoieW91IG1lYW
4gZHJvcHBpbmcgdGhlIG5vZGU/IiwiY3JlYXRlZCI6MTU5NTUw
MjAwMjk2MH0sImFyT1RPNHhMVUNNSmdJUlciOnsiZGlzY3Vzc2
lvbklkIjoiNlNFQk1qZlpyZUNWSmFxZCIsInN1YiI6ImdoOjUx
MTEyNjE4IiwidGV4dCI6IkFkZCBGaWd1cmUiLCJjcmVhdGVkIj
oxNTk1NTAyMDIyNTUwfSwiMWFWaUNCck1DRFlDRW91ZiI6eyJk
aXNjdXNzaW9uSWQiOiJHMU93WnVCM2J0eDVyZnFWIiwic3ViIj
oiZ2g6NTExMTI2MTgiLCJ0ZXh0IjoiQWRkIEZpZ3VyZSIsImNy
ZWF0ZWQiOjE1OTU1MDIwNTAwNDl9LCIxeVJ4OGVXOWZ5SHpOd2
xjIjp7ImRpc2N1c3Npb25JZCI6InRGSUt1aTl5MGY1ckVydlgi
LCJzdWIiOiJnaDo1MTExMjYxOCIsInRleHQiOiJBbGwgdGhpcy
Bjb21lIGFmdGVyIHRoZSBNYWluIE1lc3NhZ2UgUHJvYyBEaWFn
LiBUaGlzIHNob3VsZCBiZSBzYWlkIGV4cGxpY3RseSIsImNyZW
F0ZWQiOjE1OTU1MDIyODYxMjh9LCJaNU1lYTUzaktYUDBHOHlo
Ijp7ImRpc2N1c3Npb25JZCI6ImM2RDlWNVU0VGJFeDU2TXYiLC
JzdWIiOiJnaDo1MTExMjYxOCIsInRleHQiOiJOb3QgY2xlYXIg
d2hhdCBhbiBFcG9jaCBpcyIsImNyZWF0ZWQiOjE1OTU1MDIzOT
Q1MjR9LCJEbXJab2JNcnlROHpvdHp2Ijp7ImRpc2N1c3Npb25J
ZCI6IkFXajRUQ2JZdzRJNUhOWjEiLCJzdWIiOiJnaDo1MTExMj
YxOCIsInRleHQiOiJNaXNzaW5nIHRoZSBpbmZvcm1hdGlvbiB3
aGVuIGEgdHggdHVybnMgaW50byBlbGlnaWJsZSIsImNyZWF0ZW
QiOjE1OTU1MDI1ODU5MTR9LCJMbDNVamswd25heVpOT0MwIjp7
ImRpc2N1c3Npb25JZCI6IkVabERwUFJONGU2SGdXazkiLCJzdW
IiOiJnaDo1MTExMjYxOCIsInRleHQiOiJpbiBnZW5lcmFsIHRo
ZXJlIG1pZ2h0IGJlIG1vcmUgdGhhbiAxIGNvbmZsaWN0aW5nIH
R4IiwiY3JlYXRlZCI6MTU5NTUwMjYzMzQ1M30sIlh2RFozVTZH
bTloNnRaWDEiOnsiZGlzY3Vzc2lvbklkIjoiVW84NENsdE0weV
MxdmNMcSIsInN1YiI6ImdoOjUxMTEyNjE4IiwidGV4dCI6ImFk
ZCBsaW5rcyB0byBwbGFjZXMgd2hlcmUgdGhlc2UgcHJvY2Vzc2
VzIGFyZSBzcGVjaWZpZWQiLCJjcmVhdGVkIjoxNTk1NTAyOTc1
MjA5fSwiMG8yYzZBcW12a2RjSTdUNSI6eyJkaXNjdXNzaW9uSW
QiOiJzZGRRSTZIbjFYbkIxT0RXIiwic3ViIjoiZ2g6NTExMTI2
MTgiLCJ0ZXh0Ijoid2h5IGlzIHRoaXMgUGFyc2VycyBub3QgdX
NlZCBhYm92ZSB0byBjaGVjayB0aGUgdmFsdWUgcGF5bG9hZD8i
LCJjcmVhdGVkIjoxNTk1NTAzMDUwNzgzfSwiSWxRNXVsTHNwcz
BVWVhXaCI6eyJkaXNjdXNzaW9uSWQiOiJzZGRRSTZIbjFYbkIx
T0RXIiwic3ViIjoiZ2g6NTExMTI2MTgiLCJ0ZXh0IjoiaS5lLi
BpZGVudGlmeWluZyB2YWx1ZSBtZXNzYWdlcy90eCIsImNyZWF0
ZWQiOjE1OTU1MDMwODIwMDh9fSwiaGlzdG9yeSI6Wy0xMTMyOD
QyNzRdfQ==
-->