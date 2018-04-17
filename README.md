# Authentication

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Document historiek](#document-historiek)
- [OAuth2](#oauth2)
  - [Introductie](#introductie)
  - [Registratie van de applicatie](#registratie-van-de-applicatie)
  - [Authenticeren als een applicatie](#authenticeren-als-een-applicatie)
    - [OAuth2 profiel: client_credentials](#oauth2-profiel-client_credentials)
      - [1) access_token ophalen](#1-access_token-ophalen)
      - [2) API oproepen](#2-api-oproepen)
  - [Authenticeren als een gebruiker](#authenticeren-als-een-gebruiker)
    - [OAuth2 authorizatie server](#oauth2-authorizatie-server)
    - [OAuth2 profiel: authorization_code](#oauth2-profiel-authorization_code)
      - [1) de gebruiker naar de OAuth2 authorize applicatie redirecten](#1-de-gebruiker-naar-de-oauth2-authorize-applicatie-redirecten)
      - [2) access_token bekomen](#2-access_token-bekomen)
      - [3) API oproepen](#3-api-oproepen)
    - [OAuth2 profiel: implicit](#oauth2-profiel-implicit)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Document historiek

Versie   | Auteur                 | Datum      | Opmerkingen
------   | -------                | -----      | ------------
1        | Dries Droesbeke | 13/06/2017 | Initial draft.

# OAuth2

## Introductie

Applications kunnen OAuth2 gebruiken om a/mprofielen te laten authenticeren.

## Registratie van de applicatie

Op de API store kan je uw eigen applicatie registreren. Deze dien je aan te maken in de organisatie waartoe de applicatie behoort.

Indien er nog geen organisatie is dien je deze ook aan te maken.

Voor iedere omgeving is er een aparte omgeving opgezet:
* https://api-store-o.antwerpen.be
* https://api-store-a.antwerpen.be
* https://api-store.antwerpen.be

Nadat je uw applicatie aangemaakt heb kan je een contract aanmaken met de A-Profiel en/of M-Profiel API.

De OAuth2 client_id en client_secret kan je bij de detail gegevens van uw applicatie vinden. Afhankelijk van de API is er nog een goedkeuring nodig van de beheerder van de API. Deze goedkeuringsflow verloopt gehaal automatisch.

De OAuth2 tokens zijn geldig voor alle OAuth2 services waarvoor uw applicatie een contract heeft.

## Authenticeren als een applicatie

### OAuth2 profiel: client_credentials

![](img/client_credentials.png)

#### 1) access_token ophalen

*Request:*

curl -X POST -d "client_id={{CLIENT_ID}}&client_secret={{CLIENT_SECRET}}&grant_type=client_credentials" https://api-gw-p.antwerpen.be/org/service/v1/oauth2/token

*Response:*

{"token_type":"bearer","access_token":"8c5b38ffee744ac6bd1e18eae5cbd406","expires_in":7200}

#### 2) API oproepen
*Request:*

curl -v -H "Authorization: Bearer {{ACCESS_TOKEN_FROM_PREVIOUS_STEP}}" 'https://api-gw-p.antwerpen.be/astad/aprofiel/v1/me'

## Authenticeren als een gebruiker

### OAuth2 authorizatie server

De OAuth2 authorize server kan je op volgende locatie vinden:
* https://api-oauth2-o.antwerpen.be
* https://api-oauth2-a.antwerpen.be
* https://api-oauth2.antwerpen.be

Een voorbeeld redirect naar de authorizatie ziet er zo uit:
```
https://api-oauth2.antwerpen.be/v1/authorize?
response_type=code
&service=astad.aprofiel.v1
&client_id=YOUR_CLIENT_ID
&scope=astad.aprofiel.v1.username%20astad.aprofiel.v1.name%20astad.aprofiel.v1.avatar%20astad.aprofiel.v1.email%20astad.aprofiel.v1.phone
&redirect_uri=YOUR_REDIRECT_URI
&state=thisParameterWillBeAddedToTheRedirectUri
&save_consent=true
```

| Parameter | Verplicht | Omschrijving |
| :---         |     :---:      |  :---   |
| response_type   | true     | Voor de authorization_code flow dien je hier 'code' te gebruiken .    |
| client_id     | true       | De client_id die je kan terugvinden bij uw applicatie in de API store.
| scope     | true       | Een door comma's gescheiden lijst van scopes die de gebruiker dient goed te keuren.      |
| redirect_uri     | true       | De redirect_uri die overeenkomt met diegene die is ingegeven in de API store voor uw applicatie.
| redirect_uri_lng     | false       | Als deze waarde op 'true' staat zal de redirect_uri een extra parameter 'lng' hebben wanneer de gebruiker de OAuth2 authorize applicatie verlaat. De waarde voor de 'lng' querystring parameter zal overeenkomen met de taal op de Authorize applicatie. |
| service     | true       | De service die je wil gebruiker om een gebruiker te authenticeren. Dit kan 'astad.aprofiel.v1' of 'astad.mprofiel.v1' zijn.    |
| state     | false      | We raden aan om deze parameter te voorzien in de redirect naar de authorize applicatie met een referentie naar de sessie van de gebruiker. Aangezien de parameter opnieuw zal worden meegeven in de redirect_uri, kan uw applicatie valideren of de Authorize flow gestart is vanuit uw applicatie.     |
| lng     | false       | De taal voor de authorize applicatie. Standaard staat deze op 'nl'. Beschikbare talen zijn nl, fr, en, de.     |
| force_auth     | false       | Als een gebruiker al een SSO sessie heeft op de achterliggende IDP kan je met deze parameter aangeven dat je de gebruiker verplicht opnieuw wil laten aanmelden.     |
| save_consent | false | Als je deze op 'true' zet, zal de authorize applicatie de gegeven consent onthouden voor uw applicatie/gebruiker/scopes. |





### OAuth2 profiel: authorization_code

![](img/authorization_code.png)

Als uw applicatie met aprofielen of medewerkers wil laten inloggen en gegevens over deze gebruiker wenst op te halen moet de applicatie een geldig contract hebben met de "aprofiel" of "mprofiel" API.

Technische implementatie details en een werkende NodeJS demo applicatie kan je hier vinden:
https://github.com/digipolisantwerp/demo-oauth2-consumer_app_nodejs

#### 1) de gebruiker naar de OAuth2 authorize applicatie redirecten

Een authorization_code kan je bekomen door de gebruiker naar de OAuth2 Authorize applicatie te redirecten.


#### 2) access_token bekomen

Nadat de gebruiker is ingelogd en zijn goedkeuring(consent) heeft gegeven zal de gebruiker naar de redirect_uri geredirect worden die je hebt ingegeven in de details van uw applicatie op de API store;

Deze redirect_uri bevat een *code* querystring parameter als de authenticatie succesvol is. Bijvoorbeeld:
https://domain.com/callback?code=123

*Request:*

curl -X POST -d "client_id={{CLIENT_ID}}&client_secret={{CLIENT_SECRET}}&code={{CODE_FROM_PREV_STEP}}&grant_type=authorization_code" https://api-gw-p.antwerpen.be/org/service/v1/oauth2/token

*Response:*
{"refresh_token":"66f0c43c27a94ad4aa2d7574cf7b4465","token_type":"bearer","access_token":"a2824fb10b2a44b2b6f1a4aba382630a","expires_in":7200}

#### 3) API oproepen
curl -v -H "Authorization: Bearer {{ACCESS_TOKEN_FROM_PREVIOUS_STEP}}" 'https://api-gw-p.antwerpen.be/org/service/v1/resource'


### OAuth2 profiel: implicit

Hoewel het "implicit" OAuth2 profiel beschikbaar is op de API gateway raden we aan om deze niet te gebruiken.

Voornamelijk omwille van onderstaande redenen:
+ Access_tokens die zijn aangemaakt met de implicit/token flow zijn slechts 2 uur gelding en kunnen niet verlengd worden.
+ Aangezien de client_secret niet gebruikt wordt om een access_token aan te vragen is dit OAuth2 profiel vatbaarder voor security issues.



