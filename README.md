pitraft
=====

## Overview

pitraft is an implementation of the game
[Pit](http://en.wikipedia.org/wiki/Pit_(game)) using the distributed consensus
protocol [Raft](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf).

## Code Architecture

This repository contains code for the following:

1. Running a server for pitraft that implements the Raft distributed consensus
   protocol for playing Pit.
2. Running a client to play Pit by communicating to this server.
3. An implementation of Paxos separate from pitraft.

#### Pit Server

Note: Much of the code (especially in `main.go` and `server.go` is borrowed from
[raftd](https://github.com/goraft/raftd)), which implements a key-value store in
distributed fashion. Our main change here is to adapt this to playing Pit.

The main file here is `main.go` in the top-level of the directory. Upon running,
it starts a RaftServer (see [Raft's source
code](https://github.com/goraft/raft)) and starts an HTTP Server that accepts
certain requests, such as proposing trades. The code to start this server is
mainly in `server/server.go`.

Requests are mainly read or write requests. If the server receives a read
request (e.g. to see the outstanding trades), it simply returns an HTTP Response
with that information. If the server receives a write request (e.g. to propose a
trade), it runs a Command that each server machine applies to itself through
Raft's distributed consensus protocol. See `consensus/` for examples. The server
data pertaining to each player's cards and to existing trades; the models for
those objects can be found in `db/` and `tradedb/`.

#### Pit Client

The main file here is `server/server.go`. Upon running, it starts a command line
client through which the player can issue commands to read (e.g. to see the
state of outstanding trades) and to write (e.g. to propose a new trade). Clients
may join and leave the game at any time; they each start with a fixed number of
resources.

#### Paxos Implementation

## Running

To start a game and play, do the following:

1. Install [Go](http://golang.org/).
2. Get the source code via ```go get https://github.com/abliu/pitraft```.
3. To start running a leader machine for the distributed consensus algorithm,
   first make a directory in which you want to store log files. Then, from
   command line, execute:
   ```cd <pitraft_repo_folder>;
      go run main.go -p 4001 <your_log_file_folder>```
4. To start running a follower machine for the distributed consensus algorithm,
   first make a directory in which you want to store log files. Then, from
   command line, execute:
   ```cd <pitraft_repo_folder>;
      go run main.go -p <desired_port> -join <host_address:port>
      <your_log_file_folder>```
   where `host_address:port` denotes the leader's address and port number,
   specified in -p in #3.
5. To start running a client machine that plays Pit, execute:
   ```cd <pitraft_repo_folder>/client;
      go run client.go -u <host_address:port>```
   where `host_address:port` again denotes the leader's address and port number,
   specified in -p in #3.
6. When you are playing on the client machine, you will be playing through a
   command line shell. You can enter 

   ```propose [resource] [amount]```: propose a trade of [amount] of [resource]; returns tradeID and stats on trade proposed;

   ```cancel [tradeID]```: cancel a trade with ID [tradeID];

   ```getState```: get (my own) state;

   ```view```: view all live trades
