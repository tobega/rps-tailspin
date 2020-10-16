# rps-tailspin
A rock-paper-scissors server written in [tailspin](https://github.com/tobega/tailspin-v0) (with a little help from an [Undertow](https://undertow.io) server)

The purpose is to explore what might be good Tailspin style, how the Tailspin module system works and how Tailspin http APIs could be structured, beyond the simpler programming puzzles in Rosettacode or Adventofcode.

The game logic is in rps.tt, the http server (and main program) is in server.tt with json parsing in json.tt. The Undertow usage is in a tailspin module in modules/undertow/http.tt.

## Starting the server
You need the following dependencies (or equivalent):
        "lib/tailspin.jar",
        "lib/antlr-runtime-4.8.jar",
        "lib/undertow-core-2.2.1.Final.jar",
        "lib/xnio-api-3.8.2.Final.jar",
        "lib/xnio-nio-3.8.2.Final.jar",
        "lib/jboss-logging-3.4.1.Final.jar",
        "lib/wildfly-common-1.5.4.Final.jar",
        "lib/jboss-threads-3.2.0.Final.jar"

Then run
```
TAILSPIN_MODULES=modules java -cp "lib/tailspin.jar:lib/antlr-runtime-4.8.jar:lib/undertow-core-2.2.1.Final.jar:lib/xnio-api-3.8.2.Final.jar:lib/xnio-nio-3.8.2.Final.jar:lib/jboss-logging-3.4.1.Final.jar:lib/wildfly-common-1.5.4.Final.jar:lib/jboss-threads-3.2.0.Final.jar" tailspin.Tailspin server.tt
```

## Playing the game
The json object used for posting to the server contains a mandatory "name" attribute and an optional "move" attribute. Moves must be one of "rock", "paper" or "scissors" and can be updated until the game is finished.

The json object returned will contain an "id" attribute with the id of the game, a list of the "legalMoves" and a "status" in human readable language. If the game is finished, it will also contain the list of players with their moves.

### Create a game
Post to the games endpoint, e.g. `curl -i -d '{"name":"tom", "move":"scissors"}' http://localhost:8080/games`
Send the id and url to your opponent

### Query status
Get games/{id}, e.g. `curl -i http://localhost:8080/games/1`

### Update game
To add the second player and/or make or change a move, post to games/{id}, e.g. `curl -i -d '{"name":"janet", "move":"paper"}' http://localhost:8080/games/1`

## Running tests
Tests for game logic are in rps.tt and can be run by
```
java -cp "lib/tailspin.jar:lib/antlr-runtime-4.8.jar" --test tailspin.Tailspin rps.tt
```

## Implementation notes
- Tailspin currently has no thread safety, but the code as written is intended to be thread safe through the atomicity/transactionality of method calls to a stateful processor instance.
- There is an awkward impedance mismatch between the Tailspin data transformation flow and the Java fluent interface flow of function sets (interfaces) of the supporting Undertow server.