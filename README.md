# rps-tailspin
A rock-paper-scissors server written in [tailspin](https://github.com/tobega/tailspin-v0) (with a little help from the built-in server in the jdk or an [Undertow](https://undertow.io) server)

The purpose is to explore what might be good Tailspin style, how the Tailspin module system works and how Tailspin http APIs could be structured, beyond the simpler programming puzzles in Rosettacode or Adventofcode.

The game logic is written in two ways, in rps.tt as data and pattern-matching functions,
and in rpsts.tt as a processor (object) with typestates (state pattern).
The choice of game logic is done in the main program in server.tt. The json parsing is in json.tt.
The Undertow usage is in a tailspin module in modules/undertow/http.tt.

There is also another version which replaces Undertow with the jdk built-in http server.
The main program there is jdkserver.tt which includes the Tailspin code from server.tt. The coice of game logic module still applies.
The http module is in modules/jdk/http.tt and exposes the same API as the undertow module.

## Starting the server using gradle
Type the command `gradle undertow` to run the undertow version, or `gradle jdkserver` to run the plain jdk version.

## Starting the server manually
You need the following dependencies to run Tailspin code:
        "lib/tailspin.jar",
        "lib/antlr-runtime-4.8.jar"

### Undertow version
For the undertow version of the code you additionally need these dependencies (or equivalent)
        "lib/undertow-core-2.2.1.Final.jar",
        "lib/xnio-api-3.8.2.Final.jar",
        "lib/xnio-nio-3.8.2.Final.jar",
        "lib/jboss-logging-3.4.1.Final.jar",
        "lib/wildfly-common-1.5.4.Final.jar",
        "lib/jboss-threads-3.2.0.Final.jar"

You also need to allow reflection access to java.base/java.util

Then run
```
TAILSPIN_MODULES=modules java -cp "lib/tailspin.jar:lib/antlr-runtime-4.8.jar:lib/undertow-core-2.2.1.Final.jar:lib/xnio-api-3.8.2.Final.jar:lib/xnio-nio-3.8.2.Final.jar:lib/jboss-logging-3.4.1.Final.jar:lib/wildfly-common-1.5.4.Final.jar:lib/jboss-threads-3.2.0.Final.jar" --add-opens java.base/java.util=ALL-UNNAMED tailspin.Tailspin server.tt
```

### JDK version
The version that uses the built-in http-server from the jdk needs no additional dependencies, but you do need to allow reflection access to jdk.httpserver/sun.net.httpserver.
Just run
```
TAILSPIN_MODULES=modules java -cp "lib/tailspin.jar:lib/antlr-runtime-4.8.jar" --add-opens jdk.httpserver/sun.net.httpserver=ALL-UNNAMED tailspin.Tailspin jdkserver.tt
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
Tests for game logic are in rps.tt and can be run by `gradle rpsTest` or
```
java -cp "lib/tailspin.jar:lib/antlr-runtime-4.8.jar" --test tailspin.Tailspin rps.tt
```

## Implementation notes
- Tailspin currently has no thread safety, but the code as written is intended to be thread safe through the atomicity/transactionality of method calls to a stateful processor instance.
- There is an awkward impedance mismatch between the Tailspin data transformation flow and the Java fluent interface flow of function sets (interfaces) of the supporting Undertow server.