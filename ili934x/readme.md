# Pilote ILI9341/ILI9340 générique pour différent TFT - No FrameBuffer

__<<<=== CE PILOTE EST EN COURS DE REALISATION ===>>>__

![Adafruit 2.4" TFT FeatherWing](docs/_static/tft-wing-00.jpg)

Ce pilote:
* permet de contrôler des écrans à base de contrôleur ILI9341 / ILI9340
* effectue les opérations directement dans la mémoire du contrôleur ILIxxxx.
* __Ne dérive pas de FrameBuffer!__

Le bibliothèque [ili934x](lib/ili934x.py) expose une interface __imitant__ l'[API de FrameBuffer](http://docs.micropython.org/en/latest/library/framebuf.html?highlight=framebufer) de MicroPython. Le but est de pouvoir utiliser ce pilote comme si c'était un FrameBuffer.

L'__API du pilote ILI934x__ [est détaillée dans API.md](api.md)

## Crédit:
Ce pilote est basé sur les travaux suivants:
* [Micropython Driver for ILI9341 display de Jeffmer](https://github.com/jeffmer/micropython-ili9341) (GitHub)
* [Micropython Driver for ILI9341 de Ropod](https://github.com/mchobby/pyboard_drive/tree/master/ILI9341) (GitHub)
* [FreeType generator (binary font file for MicroControler)](https://github.com/mchobby/freetype-generator) (GitHub)

# Brancher

## PYBStick + 2.4" FeatherWing

La [PYBStick est une carte de Garatronic](https://shop.mchobby.be/fr/micropython/1844-pybstick-standard-26-micropython-et-arduino-3232100018440-garatronic.html) qui fonctionne aussi bien sous MicroPython que sous Arduino.

Voici les raccordement à effectuer sur le [TFT 2.4" FeatherWing d'Adafruit](https://shop.mchobby.be/fr/feather-adafruit/1050-tft-featherwing-24-touch-320x240-3232100010505-adafruit.html).

![PYBStick 26 vers TFT 2.4" FeatherWing](docs/_static/pybstick-to-tft-2.4-featherwing.jpg)


``` python
from machine import SPI, PIN
spi = SPI( 1, baudrate=40000000 )
cs_pin = Pin("S15") # PYBStick
dc_pin = Pin("S13") # PYBStick
rst_pin = None      # PYBStick

# r in 0..3 is rotation, r in 4..7 = rotation+miroring (Use 3 for landscape mode)
lcd = ILI9341( spi, cs=cs_pin, dc=dc_pin, rst=rst_pin, w=320, h=240, r=0)
```

## PYBStick + Olimex 2.8" TFT UEXT

La [PYBStick est une carte de Garatronic](https://shop.mchobby.be/fr/micropython/1844-pybstick-standard-26-micropython-et-arduino-3232100018440-garatronic.html) qui fonctionne aussi bien sous MicroPython que sous Arduino.

Voici les raccordement à effectuer sur le [TFT 2.8" d'Olimex](https://shop.mchobby.be/fr/afficheur-lcd-tft-oled/1866-afficheur-28-tactile-couleur-320x240px-uext-3232100018662-olimex.html).

![PYBStick 26 vers TFT 2.8" Olimex](docs/_static/pybstick-to-tft-2.8-olimex.jpg)

L'attribution des broches sur l'UEXT est la suivante:

```
     Descr    : UEXT Pin  (PYBStick)  (PYBStick)  UEXT Pin : Descr
    ================================================================
                                  UEXT
                                +-------+
                           3.3V | 1   2 | GND
LCD BackLight : uext-tx   (S24) | 3   4 | (S22) uext-rx   : Touch IRQ
        Touch : uext-scl  ( S5) | 5   6 | ( S3) uext-sda  : Touch
      TFT D/C : uext-miso (S21) | 7   8 | (S19) uext-mosi : TFT MOSI
      TFT SCK : uext-sck  (S23) | 9  10 | (S26) uext-cs   : TFT CS
                                +-------+
```

__A propos de la configuration__:

* Sur l'afficheur TFT 2.8" et son connecteur UEXT, Olimex n'utilise que la ligne  MOSI pour envoyer les données vers l'afficheur (MISO n'étant pas utilisé).
* La ligne S21 normalement réservée à MISO est exploitée pour contrôler le signal Data/Command du TFT vers l'afficheur. Il n'est donc pas possible d'utiliser le bus SPI matériel puisque S21 doit être piloté indépendamment en sortie.
* Les données sont donc envoyées vers l'écran en utilisant un bus SPI logiciel (_BitBanging_) sur les broches S19, S23, S26. Comme il faut impérativement une broche MISO pour le bus SPI logiciel, celle-ci est fixée arbitrairement sur la broche S16 non utilisée.

__A propos des performances__:
Etant donné que l'afficheur utilise un bus SPI Logiciel (_BitBanging_), le débit est donc limité sur le bus, ce qui impacte le taux de rafraîchissement.

``` python
from machine import SPI
from machine import Pin
from ili934x import *

# PYBStick config requires Bit-Banging one-way SPI for Olimex.
# MISO (S26) of UEXT is used for D/C.
# SPI must declare a MISO! So we used a unused in the project (S16 in this case) as fake pin
spi = SPI( -1, mosi=Pin("S19", Pin.OUT), miso=Pin("S16", Pin.IN), sck=Pin("S23", Pin.OUT) )
cs_pin = Pin("S26")
dc_pin = Pin("S21")
rst_pin = None

# r in 0..3 is rotation, r in 4..7 = rotation+miroring # Use 3 for landscape mode
lcd = ILI9341( spi, cs=cs_pin, dc=dc_pin, rst=rst_pin, w=320, h=240, r=0)
```
L'afficheur peut également être connecté sur un PYBStick à l'aide de la [carte d'interface PYBStick-Feather-Face](https://shop.mchobby.be/product.php?id_product=1996) comme indiqué sur l'image ci-dessous.

![PYBStick + PYBStick-Feather-Face + TFT Olimex](docs/_static/pybstick-feather-face-tft-olimex.jpg)

# Tester

Tous les tests sont conduits avec l'afficheur [TFT 2.4" FeatherWing d'Adafruit](https://shop.mchobby.be/fr/feather-adafruit/1050-tft-featherwing-24-touch-320x240-3232100010505-adafruit.html) mais fonctionne aussi avec les autres TFTs présentés sur cette page. Il faudra cependant adapter les instructions de création des bus.

Tous les exemples sont stockés dans le sous-répertoire [examples/](examples)

__ TODO __ : A continuer

# Resources
* [TFT 2.4" FeatherWing d'Adafruit](https://shop.mchobby.be/product.php?id_product=1050) @ MCHobby
* [TFT 2.8" d'Olimex](https://shop.mchobby.be/product.php?id_product=1866) @ MCHobby
* [carte d'interface PYBStick-Feather-Face](https://shop.mchobby.be/product.php?id_product=1996) @ MCHobby
* [ILI9341 Datasheet](https://cdn-shop.adafruit.com/datasheets/ILI9341.pdf) _stored at Adafruit Industries_
