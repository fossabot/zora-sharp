# ZoraSharp [![Build Status](https://travis-ci.org/kabili207/zora-sharp.svg?branch=master)](https://travis-ci.org/kabili207/zora-sharp) [![Build status](https://ci.appveyor.com/api/projects/status/6ok374kxhysq7adc/branch/master?svg=true)](https://ci.appveyor.com/project/kabili207/zora-sharp/branch/master) [![codecov.io](https://codecov.io/github/kabili207/zora-sharp/coverage.svg?branch=master)](https://codecov.io/github/kabili207/zora-sharp?branch=master) [![documentation](http://kabili207.github.io/zora-sharp/api-doc/badge.svg)](http://kabili207.github.io/zora-sharp/api-doc/)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkabili207%2Fzora-sharp.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkabili207%2Fzora-sharp?ref=badge_shield)

A library for working with the password system used in the Legend of Zelda Oracle of Ages and Oracle of Seasons games.
Inspired by the [original password generator](http://home.earthlink.net/~paul3/zeldagbc.html) written by
Paul D. Shoener III a.k.a. Paulygon back in 2001.

The name comes from a contraction of **Z**elda **Ora**cle and from the Zora, one of the games' races and enemies.

### User Interfaces
ZoraSharp is only a library for manipulating the codes from the Oracle series games, meant for use by developers.
General users should instead use of the the following user interfaces:
 * Windows WPF - https://github.com/kabili207/zoragen-wpf
 * Linux GTK - https://github.com/kabili207/zoragen-gtk
 * [Blazor Web App](https://friendlyanon.github.io/zoragen/) - https://github.com/friendlyanon/zoragen-blazor

### Features
 * Decodes game and ring secrets
 * Generates game, ring, and memory secrets
 * Allows import and export of game data using a [json-based `.zora` file](#the-zora-save-file)

## Using the library
Full documentation for the current stable release of ZoraSharp can be found at http://kabili207.github.io/zora-sharp/api-doc/. Below are functions to handle the most common scenarios.

### Regions
There are two supported regions, `US` and `JP`. (The PAL region is
nearly identical to the US region for the purpose of secrets.)

### Getting the raw secret
ZoraSharp uses byte arrays for most operations with the secrets. Most people don't go passing byte values
around, however, opting for a more readable text representation. These secret strings can be parsed like so:
```c#
string gameSecret = "H~2:@ left 2 diamond yq GB3 circle ( 6 heart ? up 6";
byte[] rawGameSecret = SecretParser.ParseSecret(gameSecret, GameRegion.US);
```
The `ParseSecret` method is fairly flexible in what it will accept. Possible formats include 
`6●sW↑`, `6 circle s W up`, and `6{circle}sW{up}`. You can even combine the formats like so: 
`6cIrClesW↑`. Brackets (`{` and `}`) also ignored, so long as they are next to a symbol word such as
`circle` or `left`.

Whitespace is also ignored. This does not cause any issues with the symbol words because the list of valid
characters does not include any vowels.

### Getting a secret string
It's also possible to take the raw bytes and convert them back into a readable string value.
```c#
byte[] rawSecret = new byte[]
{
     4, 37, 51, 36, 63,
    61, 51, 10, 44, 39,
     3,  0, 52, 21, 48,
    55,  9, 45, 59, 55
};
string secret = SecretParser.CreateString(rawSecret, GameRegion.US);
// H~2:@ ←2♦yq GB3●( 6♥?↑6
```

The `CreateString` method is far less flexible than it's counter-part, and will only return
a UTF-8 string, as shown above.

### Loading a secret
Secrets can be loaded from a string...
```c#
string gameSecret = "H~2:@ left 2 diamond yq GB3 circle ( 6 heart ? up 6";
Secret secret = new GameSecret(GameRegion.US);
secret.Load(gameSecret);
```
...or from a byte array
```c#
// H~2:@ ←2♦yq GB3●( 6♥?↑6
byte[] rawSecret = new byte[]
{
     4, 37, 51, 36, 63,
    61, 51, 10, 44, 39,
     3,  0, 52, 21, 48,
    55,  9, 45, 59, 55
};
Secret secret = new GameSecret(GameRegion.US);
secret.Load(rawSecret);
```
### Loading a ring secret
```c#
// L~2:N @bB↑& hmRh=
byte[] rawSecret = new byte[]
{
     6, 37, 51, 36, 13,
    63, 26,  0, 59, 47,
    30, 32, 15, 30, 49
};
Secret secret = new RingSecret(GameRegion.US);
secret.Load(rawSecret);
```

### Creating a game secret
```c#
GameSecret secret = new GameSecret(GameRegion.US)
{
    GameID = 14129,
    TargetGame = Game.Ages,
    Hero = "Link",
    Child = "Pip",
    Animal = Animal.Dimitri,
    Behavior = 4,
    IsLinkedGame = true,
    IsHeroQuest = false,
    WasGivenFreeRing = true
};
string secretString = secret.ToString();
// H~2:@ ←2♦yq GB3●( 6♥?↑6
byte[] data = secret.ToBytes();
```

### Creating a ring secret
```c#
RingSecret secret = new RingSecret(GameRegion.US)
{
    GameID = 14129,
    Rings = Rings.PowerRingL1 | Rings.DoubleEdgeRing | Rings.ProtectionRing
};
string ringSecret = secret.ToString();
// L~2:N @bB↑& hmRh=
byte[] data = secret.ToBytes();
```

### Creating a memory secret
```c#
MemorySecret secret = new MemorySecret(GameRegion.US)
{
    GameID = 14129,
    TargetGame = Game.Ages,
    Memory = Memory.ClockShopKingZora,
    IsReturnSecret = true
};
string secret = secret.ToString();
// 6●sW↑
byte[] data = secret.ToBytes();
```
 
## The .zora save file
The `.zora` file contains all relevent information to recreate a player's game and ring secrets. Data is saved
as a JSON object, with the intention that it can be used with other implementations of the password system.


```json
{
    "Region": "US",
    "Hero": "Link",
    "GameID": 14129,
    "Game": "Ages",
    "Child": "Pip",
    "Animal": "Dimitri",
    "Behavior": 4,
    "IsLinkedGame": true,
    "IsHeroQuest": false,
    "WasGivenFreeRing": true,
    "Rings": -9222246136947933182
}
```

Valid values for `Game` are `Ages` or `Seasons`. This value refers to the _target_ game.

Valid values for `Animal` are `Ricky`, `Dimitri`, or `Moosh`.

The rings are saved as a 64 bit signed integer. A signed integer was chosen to maintain compatibility with
languages that don't support unsigned integers.

None of the fields are required; the ZoraSharp library will load whatever is present, however the same
cannot be guaranteed for other libraries that implement the `.zora` save file.

## Special Thanks
 * Paulygon - Created the [original secret generator](http://home.earthlink.net/~paul3/zeldagbc.html) way back in 2001
 * 39ster - Rediscovered [how to decode game secrets](http://www.gamefaqs.com/boards/472313-the-legend-of-zelda-oracle-of-ages/66934363) using paulygon's program
 * [LunarCookies](https://github.com/LunarCookies) - Discovered the correct cipher and checksum logic used to generate secrets
 * [Drenn1](https://github.com/Drenn1) - Helped determine what the remaining unknown values represented and added support for Japanese secrets.


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkabili207%2Fzora-sharp.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkabili207%2Fzora-sharp?ref=badge_large)