# Tasks 


## Breif overview of  code 

The program provides a cli for generating cryptographic keys and running a consensus agent to simulate a Byzantine Fault Tolerant (BFT) consensus protocol.

## Commands

### Generate Keys

```bash
go run main.go genkeys --count 4 --config quorum.json
```

Generates a quorum with 4 participants.

### Run a Node

```bash
go run main.go run --id 0 --listen :4680 --config quorum.json --peers peers.json
```

Starts a consensus agent for node `0`, listening on port `4680`, using keys from `quorum.json` and peer info from `peers.json`.

---

# Core Concepts of the BDLS Consensus Protocol Emulator

This document breaks down the core concepts of the `main.go` file from the BDLS (Byzantine Distributed Ledger System) consensus protocol emulator, implemented in Go. Each section explains a key component of the code chunk by chunk, followed by a summary of what the code does overall.

The program provides a command-line interface for generating cryptographic keys and running a consensus agent to simulate a Byzantine Fault Tolerant (BFT) consensus protocol.

---

## Concept 1: Quorum Structure

### Code Snippet

```go
type Quorum struct {
	Keys []*big.Int `json:"keys"` // pem formatted keys
}
```

### What It Means

This defines a `Quorum` – basically a group of nodes (participants) in the consensus process. Each node is represented by a cryptographic private key (specifically, the D component of an ECDSA key).

The `json:"keys"` part ensures these keys are neatly stored in a JSON file, which helps share this configuration across nodes using `quorum.json`.

---

## Concept 2: CLI Setup

### Code Snippet

```go
app := &cli.App{
	Name:                 "BDLS consensus protocol emulator",
	Usage:                "Generate quorum then emulate participants",
	EnableBashCompletion: true,
	Commands: []*cli.Command{ /* genkeys and run */ },
	Action: func(c *cli.Context) error {
		cli.ShowAppHelp(c)
		return nil
	},
}
err := app.Run(os.Args)
if err != nil {
	log.Fatal(err)
}
```

### What It Means

This is the entry point of the program. It sets up a command-line interface (CLI) using the `urfave/cli` package. It defines two commands:

* `genkeys`: for generating participant keys
* `run`: for starting a node

If no command is given, it displays a helpful message instead of crashing or behaving unexpectedly.

---

## Concept 3: Key Generation (`genkeys` Command)

### Code Snippet

```go
{
	Name:  "genkeys",
	Usage: "generate quorum to participant in consensus",
	Flags: []cli.Flag{ /* count, config, append */ },
	Action: func(c *cli.Context) error {
		count := c.Int("count")
		quorum := &Quorum{}
		for i := 0; i < count; i++ {
			privateKey, _ := ecdsa.GenerateKey(bdls.S256Curve, rand.Reader)
			quorum.Keys = append(quorum.Keys, privateKey.D)
		}
		// append logic omitted here
		file, _ := os.Create(c.String("config"))
		enc := json.NewEncoder(file)
		enc.SetIndent("", "\t")
		enc.Encode(quorum)
		file.Close()
		log.Println("generate", c.Int("count"), "keys")
		return nil
	},
}
```

### What It Means

This command generates cryptographic identities (private keys) and stores them in a file. You can also add new keys later by passing the `--append` flag.

---

## Concept 4: Creating the Consensus Config

Before running a node, we configure the consensus using a struct.

### Code Snippet

```go
config := new(bdls.Config)
config.Epoch = time.Now()
config.CurrentHeight = 0
config.StateCompare = func(a bdls.State, b bdls.State) int { return bytes.Compare(a, b) }
config.StateValidate = func(bdls.State) bool { return true }
```

This creates and initializes a `bdls.Config` struct which holds the initial state and validation logic of the consensus system. It also records when the node started and what logic should be used to compare or validate states.

---

## Concept 5: Running a Consensus Agent (`run` Command)

### Code Snippet

```go
{
	Name: "run",
	Usage: "start a consensus agent",
	Flags: []cli.Flag{ 
        ...
     },
	Action: func(c *cli.Context) error {
		file, _ := os.Open(c.String("config"))
		quorum := new(Quorum)
		json.NewDecoder(file).Decode(quorum)
		id := c.Int("id")
		config := &bdls.Config{
            ...
        }
		for k := range quorum.Keys {
			priv := new(ecdsa.PrivateKey)
			priv.PublicKey.Curve = bdls.S256Curve
			priv.D = quorum.Keys[k]
			priv.PublicKey.X, priv.PublicKey.Y = bdls.S256Curve.ScalarBaseMult(priv.D.Bytes())
			if id == k {
				config.PrivateKey = priv
			}
			config.Participants = append(config.Participants, bdls.DefaultPubKeyToIdentity(&priv.PublicKey))
		}
		return startConsensus(c, config)
	},
}
```

### What It Means

This command starts a BDLS consensus agent (i.e., a node). It loads the private key, constructs the participant list, and invokes the main consensus loop.

---

## Concept 6: Consensus Execution (`startConsensus` Function)

### What's Happening

This function runs the consensus loop and handles communication between nodes.

### Key Steps Explained

#### Step 1: Create the Consensus Engine

```go
consensus, _ := bdls.NewConsensus(config)
consensus.SetLatency(200 * time.Millisecond)
```

Initializes a new BDLS consensus engine with artificial latency to simulate real network conditions.

#### Step 2: Load Peer Addresses

```go
file, _ := os.Open(c.String("peers"))
var peers []string
json.NewDecoder(file).Decode(&peers)
```

Loads peer information from `peers.json`.

#### Step 3: Set Up TCP Communication

```go
l, _ := net.ListenTCP("tcp", tcpaddr)
tagent := agent.NewTCPAgent(consensus, config.PrivateKey)
tagent.Update()
```

Initializes TCP listener and a TCP agent to manage communication.

#### Step 4: Handle Connections

**Passive (incoming):** Accepts connections and authenticates them.

**Active (outgoing):** Tries to connect to other peers continuously until successful.

#### Step 5: Consensus Loop

```go
lastHeight := uint64(0)
NEXTHEIGHT:
for {
	data := make([]byte, 1024)
	io.ReadFull(rand.Reader, data)
	tagent.Propose(data)

	for {
		newHeight, newRound, newState := tagent.GetLatestState()
		if newHeight > lastHeight {
			h := blake2b.Sum256(newState)
			log.Printf("<decide> at height:%v round:%v hash:%v", newHeight, newRound, hex.EncodeToString(h[:]))
			lastHeight = newHeight
			continue NEXTHEIGHT
		}
		<-time.After(20 * time.Millisecond)
	}
}
```

Each node keeps proposing random data and checks if consensus is achieved. Once a new height is decided, it logs the result and repeats the process.

---

## What the Code Does

The `main.go` file is a complete CLI tool to simulate BDLS consensus among multiple nodes. Here's the big picture:

* **Key Generation**: Use `genkeys` to generate cryptographic keys and save them to `quorum.json`.
* **Node Execution**: Use `run` to spin up a consensus node.

  * It reads the private key from the config
  * Connects with peer nodes
  * Proposes dummy data
  * Logs consensus outcomes (new heights)
* **Networking**: Uses TCP connections and secure key-based authentication to simulate a real distributed network.
* **Consensus Logic**: Nodes agree on data round-by-round, log results, and move on.

---