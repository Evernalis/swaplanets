# swaplanets
 A game of strategic planet hoarding.

 current version: 1.0

## About

This is a Top Trumps style game that uses planets from Star Wars as the objects that the players compete to claim. One player is determined the 'caller', and they decide what statistic from the five available is to be used to compare the planets. The opponent's planet is revealed, and the planet with the highest statistic wins the round. The winner of the round becomes the caller for the next round. If there is a draw, the winner of the next round also claims the planets contested in the drawn round.
The game ends when one player has all of the planets.

This project has been built using HTML, CSS, and JavaScript using the Vue.js framework, with data supplied by SWAPI, the Star Wars API [(swapi.info)](swapi.info).

The game can be played in offline singleplayer against an algorithm, against another player in local multiplayer, or in online multiplayer.

There is a backend server that acts as a relay between the two clients when in online mode.

When in offline singleplayer, the program decides the statistic to choose by determining the outcome of each option, then uses that data to select using a weighted random desicion algorithm.
## How to Setup

The project is accessible at [swaplanets.pages.dev](swaplanets.pages.dev). For installing locally, see `INSTALL.md`.

##