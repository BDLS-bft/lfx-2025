<h1>Manually Start Test Network Using Nano Bash</h1>
The Steps Provided I got it from the readme file of the test-network-nano-bash

```
cd to the test-network-nano-bash directory in each terminal window

./generate_artifacts.sh to generate crypto material running BFT consensus then run ./generate_artifacts.sh BFT.

In the three orderer terminals, run ./orderer1.sh, ./orderer2.sh, ./orderer3.sh respectively. If you are running BFT consensus then run ./orderer4.sh in the fourth orderer terminal also.

In the four peer terminals, run ./peer1.sh, ./peer2.sh, ./peer3.sh, ./peer4.sh respectively.

Open a different terminal and run ./join_orderers.sh. If you are running BFT Consensus then run ./join_orderers.sh BFT instead.
In the four peer admin terminals, run source peer1admin.sh && ./join_channel.sh, source peer2admin.sh && ./join_channel.sh, source peer3admin.sh && ./join_channel.sh, source peer4admin.sh && ./join_channel.sh respectively.

```
<img width="1913" height="1038" alt="Manually Start Test Network Using Nano Bash" src="https://github.com/user-attachments/assets/dfbff7bc-c906-4d12-b6b5-341663c686ce" />

<h1>Manually Start Test Network Using Nano Bash (BFT)</h1>
The Steps Provided I got it from the readme file of the test-network-nano-bash, <i><b>But</b></i> Using The BFT Mode

```
cd to the test-network-nano-bash directory in each terminal window

./generate_artifacts.sh to generate crypto material running BFT consensus then run ./generate_artifacts.sh BFT.

In the three orderer terminals, run ./orderer1.sh, ./orderer2.sh, ./orderer3.sh respectively. If you are running BFT consensus then run ./orderer4.sh in the fourth orderer terminal also.

In the four peer terminals, run ./peer1.sh, ./peer2.sh, ./peer3.sh, ./peer4.sh respectively.

Open a different terminal and run ./join_orderers.sh. If you are running BFT Consensus then run ./join_orderers.sh BFT instead.
In the four peer admin terminals, run source peer1admin.sh && ./join_channel.sh, source peer2admin.sh && ./join_channel.sh, source peer3admin.sh && ./join_channel.sh, source peer4admin.sh && ./join_channel.sh respectively.

```
<img width="1910" height="1032" alt="Manually Start Test Network Using Nano Bash (BFT)" src="https://github.com/user-attachments/assets/df1b4978-858c-489e-8abe-5071d1f0988b" />



<h1>Start Test Network Using Nano Bash(./network.sh start)</h1> 

```
./network.sh start
```
<img width="1506" height="1010" alt="Start Test Network Using Nano Bash (netwrok sh start)" src="https://github.com/user-attachments/assets/5d351338-fd54-457e-8e08-e02eefaf4d70" />

<h1>Start Test Network Using Nano Bash(./network.sh start -o bft) (Failed)</h1> 

```
./network.sh start -o bft
```
<img width="1680" height="1011" alt="Failed To Start Test Network Using Nano Bash (netwrok sh start -o bft)" src="https://github.com/user-attachments/assets/0c2eebbe-c26d-4d51-b781-0b6420c7b871" />
