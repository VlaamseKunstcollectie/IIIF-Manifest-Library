# Architectuur t.b.v. de IIIF-manifestenbibliotheek

## Inleiding

In dit document wordt de architectuur van de IIIF-manifestenbibliotheek, in
proefopstelling, nader toegelicht. De focus van deze architectuur ligt op het
hergebruik van bestaande open-source componenten en courante IIIF-technologie.

## Gebruikte componenten

Hier volgt een oplijsting van de gebruikte componenten, met een korte
toelichting:

* [elody](https://github.com/search?q=org%3Ainuits+elody&type=repositories)

    *Aaneenschakeling van microservices die in zijn geheel een DAMS (Digital
    Assets Management System) vormen.*

* [IIIF Presentation Validator](https://github.com/IIIF/presentation-validator)

    *Component verantwoordelijk voor de verrijking van de manifestenbibliotheek
    met uitsluitend geldige IIIF-manifesten (zowel volgens de v2.x als de v3.0
    standaard)*

* [pyoai](https://github.com/infrae/pyoai)

    *Deze component ligt aan de basis van een, nog uit te werken,
    microservice die verantwoordelijk is voor het ophalen van
    IIIF-manifesten van de verschillende projectpartners.*

* [Universal Viewer](https://github.com/UniversalViewer/universalviewer)

    *Dit is één van de viewers die ingebouwd wordt in de bibliotheek.*

* [TIFY](https://github.com/tify-iiif-viewer/tify) 

    *Dit is één van de viewers die ingebouwd wordt in de bibliotheek.*

* [OpenSeaDragon](https://github.com/openseadragon/openseadragon)

    *Dit is één van de viewers die ingebouwd wordt in de bibliotheek. Deze
    wordt nu reeds ondersteund door Inuits DAMS.*

* [Mirador](https://github.com/ProjectMirador/mirador)

    *Dit is één van de viewers die ingebouwd wordt in de bibliotheek.*

![Grafische voorstelling van de architectuur](img/final-architecture.drawio.svg)

## Aanvullingen bij software componenten

### [elody](https://github.com/search?q=org%3Ainuits+elody&type=repositories)

Deze component biedt al veel van de gewenste functionaliteiten aan. Enkele
functionaliteiten worden echter nog verder ontwikkeld voor dit project. Dit
gaat dan vooral over het diepgaander ondersteunen van IIIF-manifesten i.p.v.
alleenstaande mediabestanden in het systeem. Ook worden meerdere IIIF-viewers
toegevoegd.

Inuits DAMS bestaat uit een aaneenschakeling van microservices, namelijk;
de collection-api (verantwoordelijk voor het opslaan van metadata, en  tijdelijk
bijhouden van manifesten), de search-api (verantwoordelijk voor het  doorzoeken
van de DAMS), de storage-api (verantwoordelijk voor het opslaan van digitaal
materiaal) en de frontend (grafische schil bovenop deze microservices waarmee de
gebruiker interacteert). Al deze bouwstenen worden gebruikt in de
proefopstelling, behalve de storage-api. Dit omdat er geen digitaal materiaal
opgeslaan moet worden t.b.v. de manifestenbibliotheek.

### [pyoai](https://github.com/infrae/pyoai)

pyoai is een Python-bibliotheek die het toelaat om programma's te schrijven die
een OAI-PMH interface consumeren, en de data die hieruit komt verder te
verwerken, filteren, uitbreiden etc. Voor de proefopstelling ligt deze
bibliotheek aan de basis van de microservice verantwoordelijk voor het
ophalen van manifesten van de verschillende projectpartners.

De microservice waarvan pyoai aan de basis ligt, wordt modulair uitgewerkt
met uitbreidbaarheid in het achterhoofd. In eerste instantie worden alle
projectpartners ondersteund, maar indien nieuwe partners zich bij het project
toevoegen, kunnen hun collecties, in het beste geval, toegevoegd worden  zonder
de implementatie van de microservice aan te passen. Als de collecties van de
nieuwe partner via een niet-standaard manier worden blootgesteld (niet via
OAI-PMH), kunnen deze toch gemakkelijk toegevoegd worden door de modulaire
architectuur van de microservice.

### [TIFY](https://github.com/tify-iiif-viewer/tify) 

TIFY ondersteunt alleen nog maar (momenteel) manifesten van de v2.x standaard.
Bijgevolg is deze viewer alleen zichtbaar/bruikbaar bij v2.x manifesten.

### [Mirador](https://github.com/ProjectMirador/mirador)

Mirador wordt ook gebruikt om de exploratie van manifesten door gebruikers
van de bibliotheek te faciliteren. Dit door bijvoorbeeld een blank Mirador
canvas weer te geven waar de gebruikers manifesten kunnen selecteren uit een
keuzelijst om deze naast elkaar te openen.

## Ophalen van manifesten bij partners

De projectpartners stellen hun manifesten op verschillende manieren ter
beschikking; via OAI-PMH, via een REST-API, via een IIIF-collectie,
datadumps... Er wordt een component gebouwd, met aan de basis
[pyoai](https://github.com/infrae/pyoai) voor het ophalen van manifesten.
Deze manifesten worden bijgehouden in
[Inuits DAMS](https://github.com/search?q=org%3Ainuits+elody&type=repositories), meer specifiek in de
[collection-api](https://github.com/inuits/elody-collection)
microservice. Dit om de geavanceerde zoekfaciliteiten van Inuits DAMS toe te
laten, en om een snelle bevraging te garanderen. Als er bijwerkingen worden
uitgevoerd van de manifesten bij de partners, worden deze ook gereflecteerd
in de manifestenbibliotheek omdat periodiek de OAI-PMH-interface opgevraagd
wordt. Bijgevolg kan het dus voorkomen dat bijgewerkte data niet direct
beschikbaar zijn in de manifestenbibliotheek.

## Persistentie van manifesten in de bibliotheek

De manifesten van de partners worden in zijn geheel bijgehouden in de
bibliotheek (en bijgewerkt indien nodig, zie vorige paragraaf). Bepaalde
metadata worden op een uniforme manier bijgehouden in de bibliotheek. Dit
om performant manifesten te kunnen opzoeken, en om deze metadata op een
uniforme wijze te kunnen weergeven in de bibliotheek. De metadata komen niet
alleen uit de manifesten. Ook wordt gekeken of de manifesten
een [seeAlso](https://iiif.io/api/presentation/3.0/#seealso)-veld bevatten.
Indien dit het geval is, en deze data beschikbaar zijn in een ondersteund
"Linked Data"-formaat, zoals RDF/XML, JSON-LD, turtle... worden deze data ook
gebruikt om deze beschikbaar te stellen in de bibliotheek. Op die manier kunnen
manifesten dus ook opgezocht worden aan de hand van data gelinkt aan het
manifest, maar niet gespecificeerd direct in het manifest. Ook zijn deze data
zichtbaar in de bibliotheek. Niet alle data uit een conform seeAlso-document
zijn echter doorzoekbaar en zichtbaar. Alleen metadata waar Inuits DAMS kennis
van heeft zijn beschikbaar. Het gaat hier dan over data conform bepaalde
ontologieën zoals onder andere
[DublinCore](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/).
Andere data worden ook opgeslaan in het systeem, en deze originele data zijn ook
opvraagbaar.

Tot dusver is er dieper ingegaan op de JSON-representaties van de manifesten.
De verwijzingen naar de effectieve afbeeldingen opvraagbaar via de IIIF
image-api, of thumbnails beschikbaar over HTTP, worden ook direct gebruikt. Dit
wil dus zeggen dat afbeeldingen niet opgeslaan worden in de bibliotheek, maar
direct bevraagd worden aan de bron zoals die gespecificeerd is in de manifesten.

