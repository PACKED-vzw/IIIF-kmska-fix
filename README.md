IIIF KMSKA FIX
============

Deze repository levert een Catmandu fix om een csv-bestand om te zetten naar een IIIF-manifestbestand.

## Inhoudstafel
- [Hoe gebruiken?](#hoe)
- [Opbouw CSV-bestand](#csv)
- [Opbouw manifest fix](#manifest)
- [Verbeterpunten](#beter)

## Hoe gebruiken?<a id="hoe"></a>
1. Installeer [Catmandu](http://librecat.org/)
2. Installeer de Perl-module *Catmandu-Store-REST*: `cpan install Catmandu::Store::REST`
3. Zorg dat je CSV-bestand opgesteld is volgens de [Opbouw CSV-bestand](#csv) (of [pas de fix aan](#fix) volgens de opbouw van je eigen csv-bestand)
4. [Pas de manifest fix aan](#fix)
5. start Catmandu en voer volgend script uit: `catmandu convert csv to json --fix manifest.fix --array 0 --line_delimeted 1 < path-naar-je-csv-bestand | split -l 1 --additional-suffix=".json"`

Op deze manier krijg je per lijn in je csv-bestand een manifest-json.


## Opbouw CSV-bestand<a id="csv"></a>
Zie [IIIF-testdata.csv](https://github.com/PACKED-vzw/IIIF-kmska-fix/blob/master/testsample/IIIF_testdata.csv).

Het CSV-bestand bestaat uit volgende velden:

- id (id van je manifest)
- label
- rechten
- PID
- afbeeldingen

Het veld *afbeeldingen* bevat een id en een label per afbeelding. Id en label worden van elkaar gescheiden door middel van een *:* (dubbelpunt).

## Opbouw manifest fix<a id="fix"></a>
Zie [manifest.fix](https://github.com/PACKED-vzw/IIIF-kmska-fix/blob/master/manifest.fix).

Een aantal variabelen worden vastgelegd in het begin van het document. Pas de eerste drie variabelen aan (*url*, *attribution*, *logo*).
```
add_field('url', 'url-van-je-IIIF-server')
add_field('attribution', 'naam-van-je-organisatie')
add_field('logo', 'url-naar-je-logo')
add_field('iiif-context', 'http://iiif.io/presentation/2/context.json')
add_field('manifest-type', 'sc:Manifest')
add_field('sequence-type', 'sc:Sequence')
add_field('canvas-type', 'sc:Canvas')
add_field('image-type', 'oa:Annotation')
add_field('image-motivation', 'sc:painting')
add_field('resource-type', 'dctypes:Image')
add_field('format', 'image/jpeg')
add_field('service-context', 'http://iiif.io/api.image/2/context.json')
add_field('service-profile', 'http://iiif.io/api/image/2/profiles/level1.json')
```
De rest van het bestand is verantwoordelijk voor de opbouw van de manifest-structuur, achtereenvolgens: manifest, sequence, canvas, images en resources.

Via een loop wordt per afbeelding een canvas met de juiste images en resources gemaakt. De breedte en de hoogte van de afbeeldingen worden via de [IIIF Image API](http://iiif.io/api/image/2.1/) opgehaald. Hiervoor werd de Perl-module [Catmandu-Store-REST](http://search.cpan.org/~pieterdp/Catmandu-Store-REST-0.01/lib/Catmandu/Store/REST.pm) geschreven.

Pas in de delen *canvas* en *resources* telkens de url aan naar de url van je eigen IIIF-server.
```
prepend('fixed.sequences.$last.canvases.$last.@id', 'url-naar-eigen-IIIF-server')
```
```
prepend('fixed.sequences.$last.canvases.$last.images.$last.resource.@id', 'url-naar-eigen-IIIF-server')
```

Zie [IIIF Prestentation API 2.1](http://iiif.io/api/presentation/2.1/) voor meer info over de structuur van een manifest-bestand.

## Verbeterpunten<a id="beter"></a>
- bestandsnamen van de manifest-bestanden die je nu krijgt via de `split`-commando.
- url naar je IIIF-server maar op één plaats aanpassen.
- In de json-bestanden hebben de velden geen vaste volgorde. Het zou mogelijk moeten zijn om dit wel vast te leggen zodat manifest-bestanden eenvoudiger te vergelijken zijn.
